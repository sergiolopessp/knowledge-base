# Padrão Strangler Fig

O Padrão Strangler Fig substitui gradualmente um sistema legado por novos microservices, roteando funcionalidades para o novo sistema enquanto o antigo é desativado, minimizando riscos.

## Sumário

- [Padrão Strangler Fig](#padrão-strangler-fig)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)
    - [Sistema Legado](#sistema-legado)
    - [Novo Microservice](#novo-microservice)
    - [Proxy de Roteamento](#proxy-de-roteamento)
    - [Configuração](#configuração)

## Componentes

- **Sistema Legado**: Aplicação monolítica original.
- **Novo Microservice**: Serviço que substitui parte da funcionalidade.
- **Proxy/Fachada**: Roteia requisições entre legado e novo sistema.
- **Cliente**: Interage via proxy, sem perceber a transição.

## Benefícios

- Reduz riscos em migrações.
- Permite evolução incremental.
- Facilita testes em produção.
- Mantém sistema legado funcional.

## Contras

- Complexidade no roteamento.
- Sincronização entre sistemas.
- Manutenção temporária de dois sistemas.
- Possível duplicação de dados.

## Quando Usar

Use o Strangler Fig para modernizar sistemas legados, como em um e-commerce monolítico, migrando funcionalidades (ex.: pedidos) para microservices sem interromper operações.

## Exemplo com Spring Boot

### Sistema Legado

Simula um monolito para gerenciar pedidos.

```java
package com.example.legacy;

import jakarta.inject.Named;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/legacy")
@Named
public class LegacyController {
    @GetMapping("/pedidos/{pedidoId}")
    public Pedido buscar(@PathVariable String pedidoId) {
        return new Pedido(pedidoId, "Produto Legado", 50.0);
    }
}

package com.example.legacy;

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

### Novo Microservice

Microservice que substitui a funcionalidade de pedidos.

```java
package com.example.novo;

import jakarta.inject.Named;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/pedidos")
@Named
public class PedidoController {
    @GetMapping("/{pedidoId}")
    public Pedido buscar(@PathVariable String pedidoId) {
        return new Pedido(pedidoId, "Produto Novo", 100.0);
    }
}

package com.example.novo;

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

### Proxy de Roteamento

Fachada que roteia requisições entre legado e novo sistema.

```java
package com.example.proxy;

import jakarta.inject.Named;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.client.RestTemplate;

@RestController
@RequestMapping("/api")
@Named
public class StranglerProxy {
    private final RestTemplate restTemplate;

    public StranglerProxy(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    @GetMapping("/pedidos/{pedidoId}")
    public Pedido buscar(@PathVariable String pedidoId) {
        // Roteia para novo sistema se pedidoId começa com "novo"
        if (pedidoId.startsWith("novo")) {
            return restTemplate.getForObject(
                "http://localhost:8082/pedidos/{pedidoId}", Pedido.class, pedidoId);
        }
        // Caso contrário, usa