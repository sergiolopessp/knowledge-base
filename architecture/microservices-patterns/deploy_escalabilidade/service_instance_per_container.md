# Padrão Service Instance per Container

Define a execução de uma instância de serviço por contêiner, promovendo isolamento e escalabilidade em arquiteturas baseadas em contêineres.

## Sumário

- [Padrão Service Instance per Container](#padrão-service-instance-per-container)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Docker e Spring Boot](#exemplo-com-docker-e-spring-boot)

## Componentes

- **Serviço**: Aplicação ou microserviço com funcionalidade específica.
- **Contêiner**: Unidade isolada (ex.: Docker) que executa uma instância do serviço.
- **Orquestrador**: Gerencia contêineres (ex.: Kubernetes, Docker Compose).
- **Imagem do Contêiner**: Blueprint com dependências e código do serviço.

## Benefícios

- Isolamento: Cada serviço roda em seu próprio ambiente.
- Escalabilidade: Fácil adicionar ou remover instâncias.
- Portabilidade: Consistência entre ambientes (dev, teste, prod).
- Gerenciamento simplificado: Orquestradores automatizam deploy e balanceamento.

## Contras

- Overhead: Consumo de recursos por contêiner.
- Complexidade: Gerenciamento de múltiplos contêineres requer orquestração.
- Rede: Configuração adicional para comunicação entre contêineres.
- Monitoramento: Necessita ferramentas para logs e métricas distribuídas.

## Quando Usar

Use em arquiteturas de microservices ou aplicações distribuídas onde isolamento, escalabilidade e portabilidade são prioritários.

## Exemplo com Docker e Spring Boot

### Código do Serviço (Spring Boot)

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class ServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServiceApplication.class, args);
    }

    @GetMapping("/health")
    public String healthCheck() {
        return "Service is running";
    }
}
```

### Dockerfile

```dockerfile
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY target/service.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Configuração do Docker Compose

```yaml
version: '3.8'
services:
  service-instance:
    image: service:latest
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
    deploy:
      replicas: 2
```