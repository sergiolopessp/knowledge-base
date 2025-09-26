# Padrão Assíncrono (Mensageria/Eventos)

Define comunicação assíncrona, onde o cliente envia mensagens ou eventos sem aguardar resposta imediata, permitindo processamento desacoplado.

## Sumário

- [Padrão Assíncrono](#padrão-assíncrono)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot e Kafka](#exemplo-com-spring-boot-e-kafka)

## Componentes

- **Produtor**: Envia mensagens/eventos.
- **Consumidor**: Processa mensagens/eventos recebidos.
- **Broker**: Intermediário (ex.: Kafka, RabbitMQ).
- **Mensagem/Evento**: Dados transmitidos (JSON, Avro).

## Benefícios

- Desacoplamento: Produtor e consumidor operam independentemente.
- Escalabilidade: Suporta alta carga com processamento paralelo.
- Resiliência: Falhas não bloqueiam o sistema.
- Flexibilidade: Permite reprocessamento ou roteamento.

## Contras

- Complexidade: Configuração e monitoramento do broker.
- Latência: Atraso no processamento de mensagens.
- Consistência: Possíveis problemas de ordenação ou duplicação.
- Depuração: Mais difícil rastrear fluxo de eventos.

## Quando Usar

Use em sistemas distribuídos para processamento em segundo plano, notificações, ou integração entre microservices com baixa dependência temporal.

## Exemplo com Spring Boot e Kafka

### Configuração do Produtor

```java
package com.example.demo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

@Service
public class AsyncProducer {
    private final KafkaTemplate<String, String> kafkaTemplate;

    @Autowired
    public AsyncProducer(KafkaTemplate<String, String> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void sendMessage(String topic, String message) {
        kafkaTemplate.send(topic, message);
    }
}
```

### Configuração do Consumidor

```java
package com.example.demo;

import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

@Service
public class AsyncConsumer {
    @KafkaListener(topics = "my-topic", groupId = "my-group")
    public void consume(String message) {
        System.out.println("Mensagem recebida: " + message);
    }
}
```

### Configuração do Kafka

```properties
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=my-group
spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
```