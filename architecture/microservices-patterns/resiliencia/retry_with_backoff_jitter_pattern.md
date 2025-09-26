# Padrão Retry with Backoff/Jitter

O Padrão Retry with Backoff/Jitter implementa retentativas automáticas para chamadas a serviços, com atrasos crescentes (backoff) e variação aleatória (jitter) para evitar sobrecarga.

## Sumário

- [Padrão Retry with Backoff/Jitter](#padrão-retry-with-backoffjitter)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)
    - [Configuração de Retry](#configuração-de-retry)
    - [Serviço com Retry](#serviço-com-retry)

## Componentes

- **Cliente**: Inicia chamadas a serviços.
- **Mecanismo de Retry**: Gerencia retentativas com backoff e jitter.
- **Backoff**: Atraso crescente entre tentativas (ex.: exponencial).
- **Jitter**: Variação aleatória no atraso para evitar picos de carga.

## Benefícios

- Aumenta resiliência a falhas temporárias.
- Reduz sobrecarga em sistemas sobrecarregados.
- Evita picos de requisições simultâneas.
- Configuração flexível.

## Contras

- Complexidade na implementação.
- Atrasos podem impactar latência.
- Configuração inadequada pode prolongar falhas.
- Requer monitoramento de tentativas.

## Quando Usar

Use Retry with Backoff/Jitter em sistemas distribuídos, como microservices, para lidar com falhas transitórias em APIs externas ou bancos de dados.

## Exemplo com Spring Boot

### Configuração de Retry

Usa Spring Retry para configurar retentativas com backoff e jitter.

```java
package com.example.demo;

import org.springframework.context.annotation.Configuration;
import org.springframework.retry.annotation.EnableRetry;

@Configuration
@EnableRetry
public class RetryConfig {
}
```

### Serviço com Retry

Implementa retentativas com backoff exponencial e jitter.

```java
package com.example.demo;

import org.springframework.retry.annotation.Backoff;
import org.springframework.retry.annotation.Retryable;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class RetryService {
    private final RestTemplate restTemplate;

    public RetryService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    @Retryable(
        value = { Exception.class },
        maxAttempts = 3,
        backoff = @Backoff(delay = 1000, multiplier = 2, random = true)
    )
    public String callExternalService() {
        return restTemplate.getForObject("http://external-service/api", String.class);
    }
}
```