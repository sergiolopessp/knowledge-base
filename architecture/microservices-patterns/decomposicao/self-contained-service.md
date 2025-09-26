# Padrão Self-contained Service

O Padrão Self-contained Service define serviços autônomos que encapsulam toda a lógica, dados e dependências necessárias, eliminando dependências externas e promovendo independência em arquiteturas de microservices.

## Sumário

- [Padrão Self-contained Service](#padrão-self-contained-service)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)
    - [Serviço Autônomo](#serviço-autônomo)
    - [Configuração](#configuração)

## Componentes

- **Serviço Autônomo**: Contém lógica de negócio, dados (banco embarcado ou em memória) e dependências.
- **Banco de Dados Embutido**: Ex.: H2, SQLite para armazenamento local.
- **Empacotamento**: Serviço empacotado como JAR ou container (ex.: Docker).
- **API Exposta**: Interface para interação com outros serviços.

## Benefícios

- Alta independência entre serviços.
- Implantação e testes simplificados.
- Reduz latência de comunicação externa.
- Menor dependência de infraestrutura compartilhada.

## Contras

- Maior consumo de recursos por serviço.
- Complexidade em sincronizar dados entre serviços.
- Manutenção de bancos embarcados pode ser desafiadora.
- Escalabilidade limitada para grandes volumes de dados.

## Quando Usar

Use Self-contained Services em arquiteturas de microservices onde isolamento total é prioritário, como em sistemas distribuídos com baixa interdependência, por exemplo, um serviço de notificação independente em um e-commerce.

## Exemplo com Spring Boot

Simula um serviço autônomo de notificações com banco H2 embarcado.

### Serviço Autônomo

Serviço de notificação com armazenamento local.

```java
package com.example.selfcontained;

import jakarta.inject.Named;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/notificacoes")
@Named
public class NotificacaoController {
    @Autowired
    private NotificacaoRepository repository;

    @PostMapping
    public Notificacao criar(@RequestBody Notificacao notificacao) {
        return repository.save(notificacao);
    }

    @GetMapping("/{id}")
    public Notificacao buscar(@PathVariable Long id) {
        return repository.findById(id).orElse(null);
    }
}

package com.example.selfcontained;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import lombok.Data;

@Data
@Entity
public class Notificacao {
    @Id
    private Long id;
    private String mensagem;
    private String destinatario;
}
```

```java
package com.example.selfcontained;

import org.springframework.data.jpa.repository.JpaRepository;

public interface NotificacaoRepository extends JpaRepository<Notificacao, Long> {
}
```

### Configuração

Configuração do serviço com banco H2 embarcado.

```yaml
# application.yml
server:
  port: 8080
spring:
  application:
    name: notificacao-service
  datasource:
    url: jdbc:h2:mem:notificacoes
    driver-class-name: org.h2.Driver
    username: sa
    password:
  jpa:
    database-platform: org.hibernate.dialect.H2Dialect
    hibernate:
      ddl-auto: update
```