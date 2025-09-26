# Padrão Service Mesh / Sidecar

O Padrão Service Mesh implementa uma camada de infraestrutura dedicada para gerenciar comunicação entre microservices, usando proxies sidecar para roteamento, segurança, observabilidade e resiliência sem alterar o código dos serviços.

## Sumário

- [Padrão Service Mesh / Sidecar](#padrão-service-mesh--sidecar)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)
    - [Serviço Principal](#serviço-principal)
    - [Proxy Sidecar](#proxy-sidecar)
    - [Configuração](#configuração)

## Componentes

- **Service Mesh**: Infraestrutura que gerencia tráfego de rede.
- **Sidecar**: Proxy (ex.: Envoy) injetado ao lado de cada microservice.
- **Data Plane**: Proxies que processam tráfego.
- **Control Plane**: Gerencia configuração do data plane.

## Benefícios

- Centraliza gerenciamento de rede.
- Melhora segurança e criptografia.
- Facilita observabilidade (logs, métricas).
- Suporta resiliência (circuit breaker, retry).

## Contras

- Aumenta complexidade operacional.
- Overhead de performance no tráfego.
- Curva de aprendizado elevada.
- Dependência de ferramentas específicas.

## Quando Usar

Use Service Mesh em arquiteturas de microservices maduras com tráfego complexo, como em um e-commerce para gerenciar comunicação segura entre serviços de pedidos, pagamentos e estoque.

## Exemplo com Spring Boot

Simula um sidecar proxy simples para roteamento e logging.

### Serviço Principal

Microservice de pedidos.

```java
package com.example.servicemesh;

import jakarta.inject.Named;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/pedidos")
@Named
public class PedidoController {
    @GetMapping("/{pedidoId}")
    public Pedido buscar(@PathVariable String pedidoId) {
        return new Pedido(pedidoId, "Produto X", 100.0);
    }
}

package com.example.servicemesh;

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

### Proxy Sidecar

Proxy simples que intercepta chamadas e adiciona logging.

```java
package com.example.sidecar;

import jakarta.inject.Named;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.client.RestTemplate;

@RestController
@RequestMapping("/api")
@RequiredArgsConstructor
@Named
public class SidecarProxy {
    private final RestTemplate restTemplate;

    @GetMapping("/pedidos/{pedidoId}")
    public Pedido proxyPedido(@PathVariable String pedidoId) {
        System.out.println("Sidecar: Log de requisição para pedido " + pedidoId);
        Pedido pedido = restTemplate.getForObject(
            "http://localhost:8081/pedidos/{pedidoId}", Pedido.class, pedidoId);
        System.out.println("Sidecar: Log de resposta para pedido " + pedidoId);
        return pedido;
    }
}
```

### Configuração

Configuração para simular sidecar (em produção, use Istio/Envoy).

```yaml
# application.yml do Serviço Principal
server:
  port: 8081
spring:
  application:
    name: pedido-service
```

```yaml
# application.yml do Sidecar Proxy
server:
  port: 8080
spring:
  application:
    name: sidecar-proxy
```