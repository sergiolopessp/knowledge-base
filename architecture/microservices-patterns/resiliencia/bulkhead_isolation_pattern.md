# Padrão Bulkhead Isolation

O Padrão Bulkhead Isolation separa componentes ou recursos de um sistema em partições independentes, limitando o impacto de falhas e garantindo estabilidade.

## Sumário

- [Padrão Bulkhead Isolation](#padrão-bulkhead-isolation)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)
    - [Configuração de Thread Pools](#configuração-de-thread-pools)
    - [Serviço Isolado](#serviço-isolado)

## Componentes

- **Partições (Bulkheads)**: Recursos isolados (ex.: thread pools, conexões).
- **Recurso Crítico**: Componente protegido (ex.: banco de dados, API externa).
- **Gerenciador de Recursos**: Controla alocação e limites (ex.: ExecutorService).
- **Cliente**: Interage com o sistema, sem perceber a separação.

## Benefícios

- Limita propagação de falhas.
- Melhora resiliência.
- Permite escalabilidade seletiva.
- Facilita monitoramento de recursos.

## Contras

- Configuração complexa.
- Overhead de gerenciamento.
- Possível subutilização de recursos.
- Requer planejamento cuidadoso.

## Quando Usar

Use Bulkhead Isolation em sistemas distribuídos, como microservices, para proteger contra falhas em cascata ou sobrecarga em chamadas a serviços externos.

## Exemplo com Spring Boot

### Configuração de Thread Pools

Define thread pools separados para isolar chamadas a serviços.

```java
package com.example.demo;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

@Configuration
public class BulkheadConfig {
    @Bean(name = "serviceAPool")
    public ExecutorService serviceAPool() {
        return Executors.newFixedThreadPool(10); // Limite de 10 threads
    }

    @Bean(name = "serviceBPool")
    public ExecutorService serviceBPool() {
        return Executors.newFixedThreadPool(5); // Limite de 5 threads
    }
}
```

### Serviço Isolado

Usa thread pools para isolar chamadas a serviços externos.

```java
package com.example.demo;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

import java.util.concurrent.ExecutorService;

@Service
public class IsolatedService {
    private final RestTemplate restTemplate;
    private final ExecutorService serviceAPool;
    private final ExecutorService serviceBPool;

    public IsolatedService(RestTemplate restTemplate,
                          @Qualifier("serviceAPool") ExecutorService serviceAPool,
                          @Qualifier("serviceBPool") ExecutorService serviceBPool) {
        this.restTemplate = restTemplate;
        this.serviceAPool = serviceAPool;
        this.serviceBPool = serviceBPool;
    }

    public String callServiceA() {
        return serviceAPool.submit(() -> restTemplate.getForObject(
                "http://service-a/api", String.class)).get();
    }

    public String callServiceB() {
        return serviceBPool.submit(() -> restTemplate.getForObject(
                "http://service-b/api", String.class)).get();
    }
}
```
