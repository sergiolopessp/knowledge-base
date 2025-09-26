# Padrão Circuit Breaker

O Padrão Circuit Breaker protege microservices contra falhas em cascata, monitorando chamadas a serviços externos e interrompendo requisições quando falhas excedem um limite, permitindo recuperação.

## Sumário

- [Padrão Circuit Breaker](#padrão-circuit-breaker)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)
    - [Interface do Serviço](#interface-do-serviço)
    - [Serviço Real](#serviço-real)
    - [Circuit Breaker](#circuit-breaker)
    - [Uso](#uso)

## Componentes

- **Serviço**: Interface do serviço externo.
- **Serviço Real**: Implementação do serviço.
- **Circuit Breaker**: Gerencia chamadas, alternando entre estados (Fechado, Aberto, Meio-Aberto).
- **Cliente**: Usa o Circuit Breaker como se fosse o serviço.

## Benefícios

- Previne falhas em cascata.
- Suporta recuperação automática.
- Reduz sobrecarga em falhas.
- Aumenta resiliência.

## Contras

- Adiciona complexidade.
- Configuração inadequada pode causar erros.
- Exige monitoramento.

## Quando Usar

Aplique em microservices para proteger contra falhas de serviços externos, como em um e-commerce para chamadas a um serviço de pagamento, usando fallback ou recuperação.

## Exemplo com Spring Boot

### Interface do Serviço

Define o contrato para o serviço de pagamento.

```java
package com.example.circuitbreaker;

public interface PagamentoService {
    String processarPagamento(String pedidoId);
}
```

### Serviço Real

Implementação que simula falhas.

```java
package com.example.circuitbreaker;

import jakarta.inject.Named;
import lombok.RequiredArgsConstructor;

@Named
@RequiredArgsConstructor
public class PagamentoServiceImpl implements PagamentoService {
    @Override
    public String processarPagamento(String pedidoId) {
        if (Math.random() > 0.5) {
            throw new RuntimeException("Falha no pagamento");
        }
        return "Pagamento processado: " + pedidoId;
    }
}
```

### Circuit Breaker

Gerencia chamadas com estados do circuito.

```java
package com.example.circuitbreaker;

import jakarta.inject.Named;
import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Named
@RequiredArgsConstructor
public class CircuitBreaker {
    private enum Estado { FECHADO, ABERTO, MEIO_ABERTO }

    private final PagamentoService servico;
    @Getter private Estado estado = Estado.FECHADO;
    private int falhas = 0;
    private long ultimaFalha = 0;
    private static final int LIMITE_FALHAS = 3;
    private static final long TEMPO_ESPERA = 5000;

    public String executar(String pedidoId) {
        if (estado == Estado.ABERTO && System.currentTimeMillis() - ultimaFalha > TEMPO_ESPERA) {
            estado = Estado.MEIO_ABERTO;
        }
        return switch (estado) {
            case ABERTO -> "Serviço indisponível";
            case MEIO_ABERTO, FECHADO -> tentarExecutar(pedidoId);
        };
    }

    private String tentarExecutar(String pedidoId) {
        try {
            String resultado = servico.processarPagamento(pedidoId);
            if (estado == Estado.MEIO_ABERTO) {
                estado = Estado.FECHADO;
                falhas = 0;
            }
            return resultado;
        } catch (RuntimeException e) {
            falhas++;
            if (falhas >= LIMITE_FALHAS) {
                estado = Estado.ABERTO;
                ultimaFalha = System.currentTimeMillis();
            }
            return "Erro: " + e.getMessage();
        }
    }
}
```

### Uso

Demonstra o uso em um controlador REST.

```java
package com.example.circuitbreaker;

import jakarta.inject.Named;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequiredArgsConstructor
public class PagamentoController {
    private final CircuitBreaker circuitBreaker;

    @GetMapping("/pagamento/{pedidoId}")
    public String processar(@PathVariable String pedidoId) {
        return circuitBreaker.executar(pedidoId);
    }
}
```