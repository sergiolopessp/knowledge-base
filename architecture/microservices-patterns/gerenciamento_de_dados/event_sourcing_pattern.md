# Padrão Event Sourcing

O Padrão Event Sourcing armazena o estado de uma aplicação como uma sequência de eventos, permitindo reconstrução do estado e auditoria completa em sistemas distribuídos.

## Sumário

- [Padrão Event Sourcing](#padrão-event-sourcing)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)
    - [Modelo de Evento](#modelo-de-evento)
    - [Agregado](#agregado)
    - [Repositório de Eventos](#repositório-de-eventos)
    - [Uso](#uso)

## Componentes

- **Evento**: Representa uma mudança de estado.
- **Agregado**: Entidade que aplica eventos para construir seu estado.
- **Repositório de Eventos**: Armazena e recupera eventos.
- **Consumidor**: Reconstrói estado ou reage a eventos.

## Benefícios

- Auditoria completa do sistema.
- Facilita reconstrução de estados passados.
- Suporta consistência eventual.
- Flexível para evoluções no modelo.

## Contras

- Complexidade na implementação.
- Curva de aprendizado elevada.
- Requer gerenciamento de eventos (ex.: snapshots).
- Pode aumentar consumo de armazenamento.

## Quando Usar

Use Event Sourcing em sistemas que exigem auditoria ou reconstrução de estados, como em um e-commerce para rastrear mudanças em pedidos (criação, pagamento, entrega) ou em sistemas financeiros para histórico de transações.

## Exemplo com Spring Boot

### Modelo de Evento

Define eventos para um pedido.

```java
package com.example.eventsourcing;

public sealed interface PedidoEvent permits PedidoCriadoEvent, PedidoPagoEvent {
    String pedidoId();
}

package com.example.eventsourcing;

public record PedidoCriadoEvent(String pedidoId, String produto, double valor) implements PedidoEvent {}

package com.example.eventsourcing;

public record PedidoPagoEvent(String pedidoId, String status) implements PedidoEvent {}
```

### Agregado

Reconstrói o estado do pedido aplicando eventos.

```java
package com.example.eventsourcing;

import jakarta.inject.Named;
import lombok.Getter;

import java.util.ArrayList;
import java.util.List;

@Named
@Getter
public class PedidoAggregate {
    private String pedidoId;
    private String produto;
    private double valor;
    private String status;
    private final List<PedidoEvent> eventos = new ArrayList<>();

    public void criar(String pedidoId, String produto, double valor) {
        PedidoCriadoEvent evento = new PedidoCriadoEvent(pedidoId, produto, valor);
        aplicar(evento);
        eventos.add(evento);
    }

    public void pagar(String status) {
        PedidoPagoEvent evento = new PedidoPagoEvent(pedidoId, status);
        aplicar(evento);
        eventos.add(evento);
    }

    private void aplicar(PedidoEvent evento) {
        switch (evento) {
            case PedidoCriadoEvent e -> {
                this.pedidoId = e.pedidoId();
                this.produto = e.produto();
                this.valor = e.valor();
                this.status = "CRIADO";
            }
            case PedidoPagoEvent e -> this.status = e.status();
        }
    }

    public void reconstruir(List<PedidoEvent> eventos) {
        this.eventos.clear();
        eventos.forEach(evento -> {
            aplicar(evento);
            this.eventos.add(evento);
        });
    }
}
```

### Repositório de Eventos

Armazena e recupera eventos.

```java
package com.example.eventsourcing;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import lombok.Data;
import lombok.NoArgsConstructor;

@Entity
@Data
@NoArgsConstructor
public class EventoEntity {
    @Id
    private String id;
    private String pedidoId;
    private String tipoEvento;
    private String dados;

    public EventoEntity(String id, String pedidoId, String tipoEvento, String dados) {
        this.id = id;
        this.pedidoId = pedidoId;
        this.tipoEvento = tipoEvento;
        this.dados = dados;
    }
}

package com.example.eventsourcing;

import jakarta.inject.Named;
import org.springframework.data.jpa.repository.JpaRepository;

@Named
public interface EventoRepository extends JpaRepository<EventoEntity, String> {
    List<EventoEntity> findByPedidoId(String pedidoId);
}

package com.example.eventsourcing;

import com.fasterxml.jackson.databind.ObjectMapper;
import jakarta.inject.Named;
import lombok.RequiredArgsConstructor;

import java.util.List;
import java.util.UUID;

@Named
@RequiredArgsConstructor
public class PedidoEventStore {
    private final EventoRepository repository;
    private final ObjectMapper objectMapper;

    public void salvarEvento(PedidoEvent evento) throws Exception {
        String tipoEvento = evento.getClass().getSimpleName();
        String dados = objectMapper.writeValueAsString(evento);
        EventoEntity entidade = new EventoEntity(UUID.randomUUID().toString(), evento.pedidoId(), tipoEvento, dados);
        repository.save(entidade);
    }

    public List<PedidoEvent> buscarEventos(String pedidoId) throws Exception {
        List<EventoEntity> entidades = repository.findByPedidoId(pedidoId);
        return entidades.stream().map(entidade -> {
            try {
                Class<?> tipo = Class.forName("com.example.eventsourcing." + entidade.getTipoEvento());
                return (PedidoEvent) objectMapper.readValue(entidade.getDados(), tipo);
            } catch (Exception e) {
                throw new RuntimeException("Erro ao desserializar evento", e);
            }
        }).toList();
    }
}
```

### Uso

Demonstra o uso em um controlador REST.

```java
package com.example.eventsourcing;

import jakarta.inject.Named;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/pedidos")
@RequiredArgsConstructor
public class PedidoController {
    private final PedidoAggregate aggregate;
    private final PedidoEventStore eventStore;

    @PostMapping
    public void criar(@RequestBody PedidoCriadoEvent evento) throws Exception {
        aggregate.criar(evento.pedidoId(), evento.produto(), evento.valor());
        aggregate.getEventos().forEach(eventStore::salvarEvento);
    }

    @PostMapping("/{pedidoId}/pagar")
    public void pagar(@PathVariable String pedidoId, @RequestBody PedidoPagoEvent evento) throws Exception {
        aggregate.reconstruir(eventStore.buscarEventos(pedidoId));
        aggregate.pagar(evento.status());
        aggregate.getEventos().forEach(eventStore::salvarEvento);
    }

    @GetMapping("/{pedidoId}")
    public PedidoAggregate buscar(@PathVariable String pedidoId) throws Exception {
        aggregate.reconstruir(eventStore.buscarEventos(pedidoId));
        return aggregate;
    }
}
```