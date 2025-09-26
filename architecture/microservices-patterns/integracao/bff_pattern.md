# Padrão Backend for Frontend (BFF)

O Padrão Backend for Frontend (BFF) cria backends específicos para cada interface de usuário, adaptando dados e APIs às necessidades de clientes (web, mobile), simplificando a comunicação com microservices.

## Sumário

- [Padrão Backend for Frontend](#padrão-backend-for-frontend)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)
    - [Serviço de Pedidos](#serviço-de-pedidos)
    - [BFF para Web](#bff-para-web)
    - [BFF para Mobile](#bff-para-mobile)
    - [Configuração](#configuração)

## Componentes

- **BFF**: Backend dedicado a uma interface de usuário específica.
- **Microservices**: Serviços backend que fornecem dados e lógica.
- **Cliente**: Interface de usuário (web, mobile) que consome o BFF.
- **Integração**: BFF agrega e adapta dados dos microservices.

## Benefícios

- APIs otimizadas para cada cliente.
- Reduz complexidade no frontend.
- Facilita evolução independente de interfaces.
- Melhora desempenho com respostas personalizadas.

## Contras

- Aumenta número de backends a manter.
- Pode duplicar lógica entre BFFs.
- Requer coordenação com microservices.
- Complexidade adicional na arquitetura.

## Quando Usar

Use o BFF quando diferentes interfaces (web, mobile) têm necessidades distintas de dados, como em um e-commerce onde a web precisa de dados detalhados de produtos e o mobile requer respostas compactas.

## Exemplo com Spring Boot

### Serviço de Pedidos

Microservice de pedidos.

```java
package com.example.pedido;

import jakarta.inject.Named;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/pedidos")
@Named
public class PedidoController {
    @GetMapping("/{pedidoId}")
    public Pedido buscar(@PathVariable String pedidoId) {
        return new Pedido(pedidoId, "Produto X", 100.0, "CRIADO");
    }
}

package com.example.pedido;

import lombok.Data;

@Data
public class Pedido {
    private String pedidoId;
    private String produto;
    private double valor;
    private String status;

    public Pedido(String pedidoId, String produto, double valor, String status) {
        this.pedidoId = pedidoId;
        this.produto = produto;
        this.valor = valor;
        this.status = status;
    }
}
```

### BFF para Web

BFF otimizado para interface web, com dados detalhados.

```java
package com.example.bffweb;

import jakarta.inject.Named;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.client.RestTemplate;

@RestController
@RequestMapping("/web/pedidos")
@RequiredArgsConstructor
@Named
public class WebBffController {
    private final RestTemplate restTemplate;

    @GetMapping("/{pedidoId}")
    public WebPedidoView buscar(@PathVariable String pedidoId) {
        Pedido pedido = restTemplate.getForObject(
            "http://localhost:8081/pedidos/{pedidoId}", Pedido.class, pedidoId);
        return new WebPedidoView(pedido.getPedidoId(), pedido.getProduto(), 
            pedido.getValor(), pedido.getStatus(), "Detalhes completos para web");
    }
}

package com.example.bffweb;

import lombok.Data;

@Data
public class WebPedidoView {
    private String pedidoId;
    private String produto;
    private double valor;
    private String status;
    private String detalhes;

    public WebPedidoView(String pedidoId, String produto, double valor, String status, String detalhes) {
        this.pedidoId = pedidoId;
        this.produto = produto;
        this.valor = valor;
        this.status = status;
        this.detalhes = detalhes;
    }
}
```

### BFF para Mobile

BFF otimizado para interface mobile, com dados compactos.

```java
package com.example.bffmobile;

import jakarta.inject.Named;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.client.RestTemplate;

@RestController
@RequestMapping("/mobile/pedidos")
@RequiredArgsConstructor
@Named
public class MobileBffController {
    private final RestTemplate restTemplate;

    @GetMapping("/{pedidoId}")
    public MobilePedidoView buscar(@PathVariable String pedidoId) {
        Pedido pedido = restTemplate.getForObject(
            "http://localhost:8081/pedidos/{pedidoId}", Pedido.class, pedidoId);
        return new MobilePedidoView(pedido.getPedidoId(), pedido.getStatus());
    }
}

package com.example.bffmobile;

import lombok.Data;

@Data
public class MobilePedidoView {
    private String pedidoId;
    private String status;

    public MobilePedidoView(String pedidoId, String status) {
        this.pedidoId = pedidoId;
        this.status = status;
    }
}
```

### Configuração

Configuração dos BFFs e do serviço de pedidos.

```yaml
# application.yml do Serviço de Pedidos
server:
  port: 8081
spring:
  application:
    name: pedido-service
```

```yaml
# application.yml do BFF Web
server:
  port: 8082
spring:
  application:
    name: bff-web
```

```yaml
# application.yml do BFF Mobile
server:
  port: 8083
spring:
  application:
    name: bff-mobile
```