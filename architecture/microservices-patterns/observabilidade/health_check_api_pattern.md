# Padrão Health Check API

O Padrão Health Check API fornece um endpoint para verificar a saúde e disponibilidade de um sistema, facilitando monitoramento e orquestração em arquiteturas distribuídas.

## Sumário

- [Padrão Health Check API](#padrão-health-check-api)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)
    - [Endpoint de Health Check](#endpoint-de-health-check)
    - [Configuração do Actuator](#configuração-do-actuator)

## Componentes

- **Endpoint de Health**: URL que retorna o status do sistema (ex.: `/health`).
- **Indicadores de Saúde**: Verificações de componentes críticos (ex.: banco de dados, memória).
- **Ferramenta de Monitoramento**: Consome o endpoint para alertas (ex.: Prometheus, Kubernetes).
- **Resposta Padronizada**: JSON com status (ex.: `UP`, `DOWN`) e detalhes.

## Benefícios

- Monitoramento proativo.
- Integração com orquestradores (ex.: Kubernetes).
- Diagnóstico rápido de falhas.
- Simplicidade na implementação.

## Contras

- Exposição de informações sensíveis se não protegido.
- Sobrecarga com verificações frequentes.
- Configuração adicional para segurança.

## Quando Usar

Use Health Check API em sistemas distribuídos, como microservices, para verificar disponibilidade, integrar com orquestradores ou suportar balanceadores de carga.

## Exemplo com Spring Boot

### Endpoint de Health Check

Usa Spring Boot Actuator para expor o endpoint `/actuator/health`.

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class HealthCheckApplication {
    public static void main(String[] args) {
        SpringApplication.run(HealthCheckApplication.class, args);
    }
}
```

### Configuração do Actuator

Habilita o endpoint de saúde no arquivo `application.properties`.

```properties
management.endpoints.web.exposure.include=health
management.endpoint.health.show-details=always
```

**Exemplo de Resposta**

```json
{
    "status": "UP",
    "components": {
        "db": {
            "status": "UP",
            "details": {
                "database": "H2",
                "validationQuery": "isValid()"
            }
        },
        "diskSpace": {
            "status": "UP",
            "details": {
                "total": 1000000000,
                "free": 500000000,
                "threshold": 10000000
            }
        }
    }
}
```