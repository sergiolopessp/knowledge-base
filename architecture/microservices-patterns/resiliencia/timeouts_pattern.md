# Padrão Timeouts

O Padrão Timeouts define limites de tempo para operações, prevenindo esperas indefinidas em chamadas de rede ou recursos externos, melhorando a resiliência.

## Sumário

- [Padrão Timeouts](#padrão-timeouts)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)
    - [Configuração de RestTemplate](#configuração-de-resttemplate)
    - [Serviço com Timeout](#serviço-com-timeout)

## Componentes

- **Cliente**: Inicia chamadas com timeout configurado.
- **Serviço Externo**: Pode demorar além do limite.
- **Configuração de Timeout**: Define tempo máximo (ex.: connect, read timeout).
- **Mecanismo de Fallback**: Ação alternativa em caso de timeout.

## Benefícios

- Evita bloqueios indefinidos.
- Melhora latência percebida.
- Libera recursos rapidamente.
- Facilita circuit breakers.

## Contras

- Pode interromper operações válidas lentas.
- Necessita tuning por serviço.
- Aumenta complexidade de configuração.
- Falsos positivos em redes instáveis.

## Quando Usar

Use Timeouts em sistemas distribuídos, como microservices, para chamadas HTTP, consultas de BD ou integrações externas sujeitas a atrasos.

## Exemplo com Spring Boot

### Configuração de RestTemplate

Configura timeouts para chamadas HTTP.

```java
package com.example.demo;

import org.apache.http.client.HttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

@Configuration
public class TimeoutConfig {
    @Bean
    public RestTemplate restTemplate() {
        PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager();
        connectionManager.setMaxTotal(10);

        HttpClient httpClient = HttpClients.custom()
                .setConnectionManager(connectionManager)
                .build();

        HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory(httpClient);
        factory.setConnectTimeout(5000); // 5s para conectar
        factory.setReadTimeout(10000);   // 10s para ler resposta

        return new RestTemplate(factory);
    }
}
```

### Serviço com Timeout

Usa RestTemplate configurado para chamadas com timeout.

```java
package com.example.demo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class TimeoutService {
    private final RestTemplate restTemplate;

    @Autowired
    public TimeoutService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    public String callExternalService() {
        try {
            return restTemplate.getForObject("http://external-service/api", String.class);
        } catch (Exception e) {
            return "Fallback: Serviço indisponível";
        }
    }
}
```