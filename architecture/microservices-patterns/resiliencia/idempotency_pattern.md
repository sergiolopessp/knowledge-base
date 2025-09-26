# Padrão Idempotency

O Padrão Idempotency garante que uma operação, quando repetida, produza o mesmo resultado sem efeitos colaterais adicionais, aumentando a confiabilidade em sistemas distribuídos.

## Sumário

- [Padrão Idempotency](#padrão-idempotency)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)
    - [Configuração do Repositório](#configuração-do-repositório)
    - [Serviço Idempotente](#serviço-idempotente)

## Componentes

- **Chave Idempotente**: Identificador único para cada requisição (ex.: UUID).
- **Armazenamento de Estado**: Banco de dados ou cache para rastrear requisições processadas.
- **Lógica de Verificação**: Valida se a requisição já foi processada.
- **Cliente**: Envia requisições com chave idempotente.

## Benefícios

- Evita processamento duplicado.
- Aumenta confiabilidade em sistemas distribuídos.
- Facilita recuperação de falhas.
- Simplifica retry em chamadas de rede.

## Contras

- Requer armazenamento adicional.
- Pode aumentar latência devido à verificação.
- Complexidade na gestão de chaves idempotentes.
- Expiração de chaves precisa de gerenciamento.

## Quando Usar

Use o Padrão Idempotency em sistemas distribuídos, como APIs REST ou filas de mensagens, onde requisições podem ser repetidas devido a falhas de rede ou retries.

## Exemplo com Spring Boot

### Configuração do Repositório

Define um repositório para armazenar chaves idempotentes.

```java
package com.example.demo;

import org.springframework.data.jpa.repository.JpaRepository;

public interface IdempotencyRepository extends JpaRepository<IdempotencyRecord, String> {
}

@Entity
public class IdempotencyRecord {
    @Id
    private String idempotencyKey;
    private String response;
}
```

### Serviço Idempotente

Implementa lógica idempotente para processar requisições.

```java
package com.example.demo;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class IdempotentService {
    private final IdempotencyRepository repository;
    private final RestTemplate restTemplate;

    public IdempotentService(IdempotencyRepository repository, RestTemplate restTemplate) {
        this.repository = repository;
        this.restTemplate = restTemplate;
    }

    @Transactional
    public String processRequest(String idempotencyKey, String url) {
        Optional<IdempotencyRecord> record = repository.findById(idempotencyKey);
        if (record.isPresent()) {
            return record.get().getResponse();
        }

        String response = restTemplate.getForObject(url, String.class);
        IdempotencyRecord newRecord = new IdempotencyRecord();
        newRecord.setIdempotencyKey(idempotencyKey);
        newRecord.setResponse(response);
        repository.save(newRecord);
        return response;
    }
}
```