# Padrão By Business Capability

O Padrão By Business Capability organiza microservices com base nas capacidades de negócio da organização, alinhando serviços às funções específicas que a empresa desempenha.

## Sumário

- [Padrão By Business Capability](#padrão-by-business-capability)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)
    - [Serviço de Gestão de Pedidos](#serviço-de-gestão-de-pedidos)
    - [Serviço de Gestão de Clientes](#serviço-de-gestão-de-clientes)
    - [Configuração](#configuração)

## Componentes

- **Capacidade de Negócio**: Função ou processo específico do negócio (ex.: pedidos, clientes).
- **Microservice**: Serviço autônomo alinhado a uma capacidade de negócio.
- **API**: Interface para interação entre serviços e clientes.
- **Integração**: Comunicação entre serviços via APIs ou eventos.

## Benefícios

- Alinha serviços com o negócio.
- Promove autonomia por domínio.
- Facilita escalabilidade por capacidade.
- Reduz acoplamento entre equipes.

## Contras

- Requer mapeamento claro de capacidades.
- Pode gerar duplicação de dados.
- Comunicação entre serviços pode ser complexa.
- Exige governança para consistência.

## Quando Usar

Use o By Business Capability em arquiteturas de microservices para organizar serviços conforme funções de negócio, como em um e-commerce onde capacidades como pedidos, clientes e pagamentos são separadas em serviços distintos.

## Exemplo com Spring Boot

### Serviço de Gestão de Pedidos

Microservice para gerenciar pedidos.

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

### Serviço de Gestão de Clientes

Microservice para gerenciar clientes.

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

Configuração dos serviços.

```yaml
# application.yml do Serviço de Pedidos
server:
  port: 8081
spring:
  application:
    name: pedido-service
```

```yaml
# application.yml do Serviço de Clientes
server:
  port: 8082
spring:
  application:
    name: cliente-service
```