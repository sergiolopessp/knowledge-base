# Structured Logging no Spring Boot (Observability)

Logs estruturados são essenciais para sistemas modernos em Cloud. Em vez de strings de texto puro, os logs são gerados em formatos legíveis por máquinas (geralmente JSON), facilitando a agregação e análise em ferramentas como ELK Stack (Elasticsearch, Logstash, Kibana), Splunk ou Datadog.

## Benefícios
- **Queryability:** Permite filtrar por campos específicos (ex: `party_id`, `trace_id`) em vez de usar Regex complexos.
- **Contexto rico:** Uso de metadados automáticos (URI, método HTTP, user-id).
- **Análise macro:** Facilita a criação de dashboards de performance e erros.

## Implementação (SLF4J + Logback)

Para implementar logs em JSON no Spring Boot, utilizamos o **MDC (Mapped Diagnostic Context)** para injetar contexto e configuramos o `logback.xml`.

### 1. Adicionando Contexto com MDC
O MDC permite que você adicione pares de chave-valor que serão incluídos em todos os logs da thread atual.

```java
public void processOrder(String orderId) {
    try {
        MDC.put("order_id", orderId);
        logger.info("Processando pedido");
        // lógica de negócio
    } finally {
        MDC.remove("order_id"); // Crucial para não vazar contexto em pools de threads
    }
}
```

### 2. Configuração do logback.xml

Exemplo de configuração para gerar logs em JSON:

```XML

<appender name="JSON_ROLLING" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>logs/app.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>logs/app-%d{yyyy-MM-dd}.log</fileNamePattern>
    </rollingPolicy>
    <layout class="ch.qos.logback.contrib.json.classic.JsonLayout">
        <jsonFormatter class="ch.qos.logback.contrib.jackson.JacksonJsonFormatter">
            <prettyPrint>false</prettyPrint>
        </jsonFormatter>
        <timestampFormat>yyyy-MM-dd HH:mm:ss.SSS</timestampFormat>
        <appendLineSeparator>true</appendLineSeparator>
    </layout>
</appender>
```
## Lições Aprendidas (Best Practices)

MDC Cleanup: Como Docker Captain e dev experiente, reforço: sempre limpe o MDC no bloco finally. Em servidores como Tomcat/Netty, as threads são reaproveitadas. Se não limpar, o log de uma requisição pode aparecer com o ID da anterior.

Thread Pools: O MDC não é transferido automaticamente para sub-threads. Ao usar Executors, você deve copiar o mapa de contexto manualmente ou usar wrappers.

Standardization: Defina nomes de campos padrão (ex: trace_id em vez de correlationId) para que todos os seus microserviços (Java, Go ou C++) sigam o mesmo esquema.

🔗 Referência
[Unlocking Observability: Structured Logging in Spring Boot (Booking.com)](https://medium.com/booking-com-development/unlocking-observability-structured-logging-in-spring-boot-c81dbabfb9e7)