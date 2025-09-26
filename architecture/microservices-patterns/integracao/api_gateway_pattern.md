# Padrão API Gateway

O Padrão API Gateway atua como ponto de entrada único para clientes, roteando requisições para microservices, agregando respostas e lidando com preocupações transversais como autenticação e balanceamento de carga.

## Sumário

- [Padrão API Gateway](#padrão-api-gateway)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)
    - [Serviço de Pedidos](#serviço-de-pedidos)
    - [Serviço de Pagamentos](#serviço-de-pagamentos)
    - [API Gateway](#api-gateway)
    - [Configuração](#configuração)

## Componentes

- **API Gateway**: Ponto de entrada que roteia requisições e gerencia preocupações transversais.
- **Microservices**: Serviços backend que processam requisições específicas.
- **Cliente**: Interage com o sistema através do API Gateway.
- **Roteamento**: Mapeia requisições do cliente para o microservice correto.

## Benefícios

- Centraliza autenticação, autorização e logging.
- Simplifica acesso do cliente a múltiplos serviços.
- Suporta balanceamento de carga e caching.
- Abstrai complexidade da arquitetura.

## Contras

- Ponto único de falha, se não bem projetado.
- Pode introduzir latência.
- Configuração e manutenção complexas.
- Requer escalabilidade própria.

## Quando Usar

Use o API Gateway em arquiteturas de microservices para unificar acesso a serviços, como em um e-commerce onde clientes acessam pedidos, pagamentos e entregas por uma única interface, com autenticação centralizada.

## Exemplo com Spring Boot

### Serviço de Pedidos

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
    @PostMapping
    public String criar(@RequestBody Pedido pedido) {
        return "Pedido criado: " + pedido.getPedidoId();
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

### Serviço de Pagamentos

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
    @PostMapping
    public String processar(@RequestBody Pagamento pagamento) {
        return "Pagamento processado: " + pagamento.getPagamentoId();
    }

    @GetMapping("/{pagamentoId}")
    public Pagamento buscar(@PathVariable String pagamentoId) {
        return new Pagamento(pagamentoId, "pedido123", 100.0, "PROCESSADO");
    }
}

package com.example.pagamento;

import lombok.Data;

@Data
public class Pagamento {
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
```

### API Gateway

Configuração do gateway com roteamento.

```java
package com.example.apigateway;

import jakarta.inject.Named;
import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Named
@Configuration
public class ApiGatewayConfig {
    @Bean
    public RouteLocator routeLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("pedidos", r -> r.path("/api/pedidos/**")
                        .filters(f -> f.stripPrefix(1))
                        .uri("http://localhost:8081"))
                .route("pagamentos", r -> r.path("/api/pagamentos/**")
                        .filters(f -> f.stripPrefix(1))
                        .uri("http://localhost:8082"))
                .build();
    }
}
```

### Configuração

Configuração do API Gateway com Spring Cloud Gateway.

```yaml
# application.yml do API Gateway
server:
  port: 8080
spring:
  cloud:
    gateway:
      routes:
        - id: pedidos
          uri: http://localhost:8081
          predicates:
            - Path=/api/pedidos/**
          filters:
            - StripPrefix=1
        - id: pagamentos
          uri: http://localhost:8082
          predicates:
            - Path=/api/pagamentos/**
          filters:
            - StripPrefix=1
```