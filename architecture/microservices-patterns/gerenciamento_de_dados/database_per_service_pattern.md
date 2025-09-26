# Padrão Database per Service

O Padrão Database per Service atribui a cada microservice seu próprio banco de dados, promovendo isolamento, autonomia e desacoplamento em arquiteturas distribuídas.

## Sumário

- [Padrão Database per Service](#padrão-database-per-service)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)
    - [Serviço de Pedidos](#serviço-de-pedidos)
    - [Serviço de Pagamentos](#serviço-de-pagamentos)
    - [Configuração de Bancos](#configuração-de-bancos)

## Componentes

- **Microservice**: Unidade autônoma com lógica de negócio.
- **Banco de Dados**: Base exclusiva para cada microservice.
- **API**: Interface para comunicação entre serviços.
- **Sincronização**: Mecanismos (ex.: eventos) para consistência.

## Benefícios

- Autonomia para cada serviço.
- Escalabilidade independente.
- Isolamento de falhas.
- Flexibilidade na escolha de tecnologias.

## Contras

- Complexidade na sincronização de dados.
- Consistência eventual pode ser desafiadora.
- Gerenciamento de múltiplos bancos.
- Custo operacional aumentado.

## Quando Usar

Use em arquiteturas de microservices onde serviços têm requisitos de dados distintos, como em um e-commerce com serviços de pedidos e pagamentos, para garantir isolamento e escalabilidade.

## Exemplo com Spring Boot

### Serviço de Pedidos

Modelo e repositório para o serviço de pedidos.

```java
package com.example.pedido;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import lombok.Data;
import lombok.NoArgsConstructor;

@Entity
@Data
@NoArgsConstructor
public class Pedido {
    @Id
    private String pedidoId;
    private String produto;
    private double valor;

    public Pedido(String pedidoId, String produto, double valor) {
        this.pedidoId = pedidoId;
        this.produto = produto;
        this.valor = valor;
    }
}

package com.example.pedido;

import jakarta.inject.Named;
import org.springframework.data.jpa.repository.JpaRepository;

@Named
public interface PedidoRepository extends JpaRepository<Pedido, String> {}
```

Controlador do serviço de pedidos.

```java
package com.example.pedido;

import jakarta.inject.Named;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/pedidos")
@RequiredArgsConstructor
public class PedidoController {
    private final PedidoRepository repository;

    @PostMapping
    public void criar(@RequestBody Pedido pedido) {
        repository.save(pedido);
    }

    @GetMapping("/{pedidoId}")
    public Pedido buscar(@PathVariable String pedidoId) {
        return repository.findById(pedidoId)
                .orElseThrow(() -> new RuntimeException("Pedido não encontrado"));
    }
}
```

### Serviço de Pagamentos

Modelo e repositório para o serviço de pagamentos.

```java
package com.example.pagamento;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import lombok.Data;
import lombok.NoArgsConstructor;

@Entity
@Data
@NoArgsConstructor
public class Pagamento {
    @Id
    private String pagamentoId;
    private String pedidoId;
    private double valor;
    private String status;

    public Pagamento(String pagamentoId, String pedidoId, double valor, String status) {
        this.pagamentoId = pagamentoId;
        this.pedidoId = pedidoId;
        this.valor = valor;
        this.status = status;
    }
}

package com.example.pagamento;

import jakarta.inject.Named;
import org.springframework.data.jpa.repository.JpaRepository;

@Named
public interface PagamentoRepository extends JpaRepository<Pagamento, String> {}
```

Controlador do serviço de pagamentos.

```java
package com.example.pagamento;

import jakarta.inject.Named;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/pagamentos")
@RequiredArgsConstructor
public class PagamentoController {
    private final PagamentoRepository repository;

    @PostMapping
    public void processar(@RequestBody Pagamento pagamento) {
        pagamento.setStatus("PROCESSADO");
        repository.save(pagamento);
    }

    @GetMapping("/{pagamentoId}")
    public Pagamento buscar(@PathVariable String pagamentoId) {
        return repository.findById(pagamentoId)
                .orElseThrow(() -> new RuntimeException("Pagamento não encontrado"));
    }
}
```

### Configuração de Bancos

Configuração do banco de dados para cada serviço (exemplo com H2).

```yaml
# application.yml do Serviço de Pedidos
spring:
  datasource:
    url: jdbc:h2:mem:pedidosdb
    driverClassName: org.h2.Driver
    username: sa
    password:
  jpa:
    database-platform: org.hibernate.dialect.H2Dialect
```

```yaml
# application.yml do Serviço de Pagamentos
spring:
  datasource:
    url: jdbc:h2:mem:pagamentosdb
    driverClassName: org.h2.Driver
    username: sa
    password:
  jpa:
    database-platform: org.hibernate.dialect.H2Dialect
```