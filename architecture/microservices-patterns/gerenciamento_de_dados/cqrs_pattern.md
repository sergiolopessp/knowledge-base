# Padrão CQRS

O Padrão CQRS (Command Query Responsibility Segregation) separa as responsabilidades de leitura (queries) e escrita (commands) em modelos distintos, otimizando desempenho e escalabilidade em sistemas complexos.

## Sumário

- [Padrão CQRS](#padrão-cqrs)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)
    - [Modelo de Comando](#modelo-de-comando)
    - [Modelo de Consulta](#modelo-de-consulta)
    - [Serviço CQRS](#serviço-cqrs)
    - [Uso](#uso)

## Componentes

- **Comando**: Operação que altera o estado do sistema.
- **Consulta**: Operação que recupera dados sem alterar estado.
- **Modelo de Comando**: Gerencia escrita (banco de escrita).
- **Modelo de Consulta**: Gerencia leitura (banco de leitura).
- **Sincronizador**: Garante consistência entre modelos.

## Benefícios

- Otimiza desempenho de leitura e escrita.
- Escalabilidade independente para cada modelo.
- Suporta modelos de dados específicos por operação.
- Facilita arquiteturas orientadas a eventos.

## Contras

- Aumenta complexidade da arquitetura.
- Consistência eventual pode ser desafiadora.
- Requer sincronização entre modelos.

## Quando Usar

Use CQRS em sistemas com alta demanda de leitura e escrita, como em um e-commerce para gerenciar pedidos (escrita) e relatórios de vendas (leitura), onde modelos otimizados melhoram desempenho.

## Exemplo com Spring Boot

### Modelo de Comando

Define o comando e manipula escrita.

```java
package com.example.cqrs;

public record PedidoCommand(String pedidoId, String produto, double valor) {}

package com.example.cqrs;

import jakarta.inject.Named;
import lombok.RequiredArgsConstructor;

@Named
@RequiredArgsConstructor
public class PedidoCommandHandler {
    private final PedidoWriteRepository writeRepository;
    private final PedidoEventPublisher eventPublisher;

    public void handle(PedidoCommand command) {
        writeRepository.salvar(new PedidoEntity(command.pedidoId(), command.produto(), command.valor()));
        eventPublisher.publicar(new PedidoCriadoEvent(command.pedidoId(), command.produto(), command.valor()));
    }
}

package com.example.cqrs;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import lombok.Data;
import lombok.NoArgsConstructor;

@Entity
@Data
@NoArgsConstructor
public class PedidoEntity {
    @Id
    private String pedidoId;
    private String produto;
    private double valor;

    public PedidoEntity(String pedidoId, String produto, double valor) {
        this.pedidoId = pedidoId;
        this.produto = produto;
        this.valor = valor;
    }
}

package com.example.cqrs;

import jakarta.inject.Named;
import org.springframework.data.jpa.repository.JpaRepository;

@Named
public interface PedidoWriteRepository extends JpaRepository<PedidoEntity, String> {}
```

### Modelo de Consulta

Define a consulta e manipula leitura.

```java
package com.example.cqrs;

public record PedidoQuery(String pedidoId) {}

package com.example.cqrs;

import jakarta.inject.Named;
import lombok.RequiredArgsConstructor;

@Named
@RequiredArgsConstructor
public class PedidoQueryHandler {
    private final PedidoReadRepository readRepository;

    public PedidoView handle(PedidoQuery query) {
        return readRepository.findById(query.pedidoId())
                .orElseThrow(() -> new RuntimeException("Pedido não encontrado"));
    }
}

package com.example.cqrs;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import lombok.Data;
import lombok.NoArgsConstructor;

@Entity
@Data
@NoArgsConstructor
public class PedidoView {
    @Id
    private String pedidoId;
    private String produto;
    private double valor;

    public PedidoView(String pedidoId, String produto, double valor) {
        this.pedidoId = pedidoId;
        this.produto = produto;
        this.valor = valor;
    }
}

package com.example.cqrs;

import jakarta.inject.Named;
import org.springframework.data.jpa.repository.JpaRepository;

@Named
public interface PedidoReadRepository extends JpaRepository<PedidoView, String> {}
```

### Serviço CQRS

Sincroniza modelos via eventos.

```java
package com.example.cqrs;

public record PedidoCriadoEvent(String pedidoId, String produto, double valor) {}

package com.example.cqrs;

import jakarta.inject.Named;
import org.springframework.context.ApplicationEventPublisher;

@Named
public class PedidoEventPublisher {
    private final ApplicationEventPublisher publisher;

    public PedidoEventPublisher(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    public void publicar(PedidoCriadoEvent event) {
        publisher.publishEvent(event);
    }
}

package com.example.cqrs;

import jakarta.inject.Named;
import lombok.RequiredArgsConstructor;
import org.springframework.context.event.EventListener;

@Named
@RequiredArgsConstructor
public class PedidoEventHandler {
    private final PedidoReadRepository readRepository;

    @EventListener
    public void handle(PedidoCriadoEvent event) {
        readRepository.save(new PedidoView(event.pedidoId(), event.produto(), event.valor()));
    }
}
```

### Uso

Demonstra o uso em um controlador REST.

```java
package com.example.cqrs;

import jakarta.inject.Named;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RestController
@RequiredArgsConstructor
@RequestMapping("/pedidos")
public class PedidoController {
    private final PedidoCommandHandler commandHandler;
    private final PedidoQueryHandler queryHandler;

    @PostMapping
    public void criarPedido(@RequestBody PedidoCommand command) {
        commandHandler.handle(command);
    }

    @GetMapping("/{pedidoId}")
    public PedidoView buscarPedido(@PathVariable String pedidoId) {
        return queryHandler.handle(new PedidoQuery(pedidoId));
    }
}
```