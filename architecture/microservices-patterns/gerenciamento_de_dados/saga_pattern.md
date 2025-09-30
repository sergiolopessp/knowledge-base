# Padrão Saga

O Padrão Saga gerencia transações distribuídas em microservices, dividindo-as em etapas locais com compensações, garantindo consistência sem bloqueios tradicionais.

## Sumário

- [Padrão Saga](#padrão-saga)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)
    - [Interface do Serviço](#interface-do-serviço)
    - [Serviços Locais](#serviços-locais)
    - [Orquestrador Saga](#orquestrador-saga)
    - [Uso](#uso)

## Componentes

- **Saga**: Sequência de transações locais.
- **Serviço Local**: Executa uma etapa e sua compensação.
- **Orquestrador**: Coordena a execução e compensações.
- **Cliente**: Inicia a Saga.

## Benefícios

- Garante consistência eventual.
- Evita bloqueios de recursos.
- Suporta falhas com compensações.
- Escalável para microservices.

## Contras

- Complexidade na implementação de compensações.
- Consistência eventual pode ser confusa.
- Requer coordenação cuidadosa.

## Quando Usar

Use o Saga para transações distribuídas em microservices, como em um e-commerce para processar pedidos envolvendo estoque, pagamento e entrega, onde falhas exigem reversão.

## Exemplo com Spring Boot

### Interface do Serviço

Define o contrato para serviços locais.

```java
package com.example.saga;

public interface SagaService {
    boolean executar(String pedidoId);
    boolean compensar(String pedidoId);
}
```

### Serviços Locais

Implementam etapas e compensações.

```java
package com.example.saga;

import jakarta.inject.Named;
import lombok.RequiredArgsConstructor;
import java.util.Random;

@Named
@RequiredArgsConstructor
public class EstoqueService implements SagaService {
    @Override
    public boolean executar(String pedidoId) {
        if (new Random().nextBoolean()) {
            System.out.println("Estoque reservado: " + pedidoId);
            return true;
        }
        throw new RuntimeException("Falha ao reservar estoque");
    }

    @Override
    public boolean compensar(String pedidoId) {
        System.out.println("Estoque liberado: " + pedidoId);
        return true;
    }
}

@Named
@RequiredArgsConstructor
public class PagamentoService implements SagaService {
    @Override
    public boolean executar(String pedidoId) {
        if (new Random().nextBoolean()) {
            System.out.println("Pagamento processado: " + pedidoId);
            return true;
        }
        throw new RuntimeException("Falha no pagamento");
    }

    @Override
    public boolean compensar(String pedidoId) {
        System.out.println("Pagamento estornado: " + pedidoId);
        return true;
    }
}
```

### Orquestrador Saga

Coordena a execução e compensações.

```java
package com.example.saga;

import jakarta.inject.Named;
import lombok.RequiredArgsConstructor;
import java.util.ArrayList;
import java.util.List;

@Named
@RequiredArgsConstructor
public class SagaOrchestrator {
    private final List<SagaService> servicos;
    private final List<SagaService> executados = new ArrayList<>();

    public boolean executarSaga(String pedidoId) {
        try {
            for (SagaService servico : servicos) {
                if (servico.executar(pedidoId)) {
                    executados.add(servico);
                }
            }
            return true;
        } catch (Exception e) {
            compensar();
            return false;
        }
    }

    private void compensar() {
        for (int i = executados.size() - 1; i >= 0; i--) {
            executados.get(i).compensar("pedido");
        }
    }
}
```

### Uso

Demonstra o uso em um controlador REST.

```java
package com.example.saga;

import jakarta.inject.Named;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import java.util.List;

@RestController
@RequiredArgsConstructor
public class SagaController {
    private final EstoqueService estoqueService;
    private final PagamentoService pagamentoService;

    @PostMapping("/pedido/{ped