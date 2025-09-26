# Event-Driven Architecture

Define um padrão onde sistemas reagem a eventos, permitindo comunicação assíncrona e desacoplada entre componentes.

## Sumário

- [Event-Driven Architecture](#event-driven-architecture)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot e Kafka](#exemplo-com-spring-boot-e-kafka)

## Componentes

- **Evento**: Registro de uma ocorrência (ex.: pedido criado, pagamento processado).
- **Produtor de Eventos**: Gera e publica eventos.
- **Consumidor de Eventos**: Reage a eventos recebidos.
- **Broker de Eventos**: Gerencia entrega (ex.: Kafka, RabbitMQ).

## Benefícios

- Desacoplamento: Componentes independentes, sem dependência temporal.
- Escalabilidade: Suporta picos de eventos com processamento paralelo.
- Flexibilidade: Facilita adição de novos consumidores.
- Resiliência: Falhas isoladas não param o sistema.

## Contras

- Complexidade: Configuração e monitoramento do broker.
- Consistência eventual: Pode haver atrasos na propagação de eventos.
- Depuração: Dificuldade em rastrear fluxos de eventos.
- Overhead: Gerenciamento de eventos exige infraestrutura robusta.

## Quando Usar

Use em sistemas distribuídos, como microservices, para notificações, auditoria, integração ou processamento em tempo real (ex.: e-commerce, IoT).

## Exemplo com Spring Boot e Kafka

### Definição do Evento

```java
package com.example.demo;

public record OrderEvent(String orderId, String status) {}
```

### Produtor de Eventos

```java
package com.example.demo;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

@Service
public class OrderEventProducer {
    private final KafkaTemplate<String, String> kafkaTemplate;
    private final ObjectMapper objectMapper;

    @Autowired
    public OrderEventProducer(KafkaTemplate<String, String> kafkaTemplate, ObjectMapper objectMapper) {
        this.kafkaTemplate = kafkaTemplate;
        this.objectMapper = objectMapper;
    }

    public void publishEvent(OrderEvent event) throws Exception {
        String eventJson = objectMapper.writeValueAsString(event);
        kafkaTemplate.send("order-events", event.orderId(), eventJson);
    }
}
```

### Consumidor de Eventos

```java
package com.example.demo;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

@Service
public class OrderEventConsumer {
    private final ObjectMapper objectMapper;

    @Autowired
    public OrderEventConsumer(ObjectMapper objectMapper) {
        this.objectMapper = objectMapper;
    }

    @KafkaListener(topics = "order-events", groupId = "order-group")
    public void consume(String eventJson) throws Exception {
        OrderEvent event = objectMapper.readValue(eventJson, OrderEvent.class);
        System.out.println("Evento recebido: " + event.orderId() + " - " + event.status());
    }
}
```

### Configuração do Kafka

```properties
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=order-group
spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
```