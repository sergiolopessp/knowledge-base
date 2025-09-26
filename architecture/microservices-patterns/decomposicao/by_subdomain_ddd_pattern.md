# Padrão By Subdomain (DDD)

O Padrão By Subdomain, baseado em Domain-Driven Design (DDD), organiza microservices em torno de subdomínios do negócio, alinhando serviços às áreas específicas do domínio da aplicação.

## Sumário

- [Padrão By Subdomain (DDD)](#padrão-by-subdomain-ddd)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)
    - [Subdomínio de Pedidos](#subdomínio-de-pedidos)
    - [Subdomínio de Clientes](#subdomínio-de-clientes)
    - [Configuração](#configuração)

## Componentes

- **Subdomínio**: Parte específica do domínio do negócio (ex.: pedidos, clientes).
- **Microservice**: Serviço autônomo mapeado a um subdomínio.
- **Contexto Delimitado (Bounded Context)**: Define limites claros para cada subdomínio.
- **Integração**: Comunicação entre contextos via APIs ou eventos.

## Benefícios

- Alinha serviços com o domínio do negócio.
- Promove contextos delimitados claros.
- Reduz acoplamento entre subdomínios.
- Facilita evolução independente de serviços.

## Contras

- Requer análise profunda do domínio.
- Integração entre contextos pode ser complexa.
- Pode levar a duplicação de dados.
- Exige conhecimento em DDD.

## Quando Usar

Use o By Subdomain (DDD) em arquiteturas de microservices com domínios complexos, como em um e-commerce onde subdomínios como pedidos e clientes têm regras e modelos distintos, exigindo contextos delimitados.

## Exemplo com Spring Boot

### Subdomínio de Pedidos

Microservice para o contexto delimitado de pedidos.

```java
package com.example.pedido;

import jakarta.inject.Named;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/pedidos")
@Named
public class PedidoController {
    @PostMapping
    public Pedido criar(@RequestBody Pedido pedido) {
        return new Pedido(pedido.getPedidoId(), pedido.getProduto(), pedido.getValor());
    }

    @GetMapping("/{pedidoId}")
    public Pedido buscar(@PathVariable String pedidoId) {
        return new Pedido(pedidoId, "Produto X", 100.0);
    }
}

package com.example.pedido;

import lombok.Data;

@Data
public class Pedido {
    private String pedidoId;
    private String produto;
    private double valor;

    public Pedido(String pedidoId, String produto, double valor) {
        this.pedidoId = pedidoId;
        this.produto = produto;
        this.valor = valor;
    }
}
```

### Subdomínio de Clientes

Microservice para o contexto delimitado de clientes.

```java
package com.example.cliente;

import jakarta.inject.Named;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/clientes")
@Named
public class ClienteController {
    @PostMapping
    public Cliente criar(@RequestBody Cliente cliente) {
        return new Cliente(cliente.getClienteId(), cliente.getNome(), cliente.getEmail());
    }

    @GetMapping("/{clienteId}")
    public Cliente buscar(@PathVariable String clienteId) {
        return new Cliente(clienteId, "João Silva", "joao@exemplo.com");
    }
}

package com.example.cliente;

import lombok.Data;

@Data
public class Cliente {
    private String clienteId;
    private String nome;
    private String email;

    public Cliente(String clienteId, String nome, String email) {
        this.clienteId = clienteId;
        this.nome = nome;
        this.email = email;
    }
}
```

### Configuração

Configuração dos serviços por subdomínio.

```yaml
# application.yml do Subdomínio de Pedidos
server:
  port: 8081
spring:
  application:
    name: pedido-service
```

```yaml
# application.yml do Subdomínio de Clientes
server:
  port: 8082
spring:
  application:
    name: cliente-service
```