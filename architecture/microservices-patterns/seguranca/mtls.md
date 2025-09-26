# Padrão mTLS (Mutual TLS)

Autenticação bidirecional entre cliente e servidor usando certificados TLS, garantindo segurança e confiança em comunicações.

## Sumário

- [Padrão mTLS](#padrão-mtls)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)

## Componentes

- **Certificados**: Cliente e servidor possuem certificados assinados por uma CA.
- **Autoridade Certificadora (CA)**: Emite e valida certificados.
- **TLS Handshake**: Verifica certificados de ambas as partes.
- **Configuração de Rede**: Define endpoints para mTLS.

## Benefícios

- Segurança reforçada: Autenticação mútua evita acessos não autorizados.
- Confidencialidade: Criptografia protege dados em trânsito.
- Integridade: Garante que dados não foram alterados.
- Escalabilidade: Ideal para APIs e microservices.

## Contras

- Complexidade: Gerenciamento de certificados é trabalhoso.
- Overhead: Handshake TLS adiciona latência.
- Manutenção: Renovação e revogação de certificados.
- Compatibilidade: Requer suporte em cliente e servidor.

## Quando Usar

Use em sistemas distribuídos, como microservices, APIs internas ou comunicações sensíveis entre serviços (ex.: bancos, saúde).

## Exemplo com Spring Boot

### Configuração do Servidor (application.yml)

```yaml
server:
  port: 8443
  ssl:
    enabled: true
    key-store: classpath:server-keystore.p12
    key-store-password: changeit
    key-store-type: PKCS12
    trust-store: classpath:server-truststore.p12
    trust-store-password: changeit
    trust-store-type: PKCS12
    client-auth: need
```

### Configuração do Cliente HTTP

```java
package com.example.demo;

import org.apache.http.client.HttpClient;
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.ssl.SSLContextBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

@Configuration
public class MtlsConfig {
    @Bean
    public RestTemplate restTemplate() throws Exception {
        SSLContextBuilder sslContext = SSLContextBuilder.create()
            .loadKeyMaterial(
                ResourceUtils.getFile("classpath:client-keystore.p12"),
                "changeit".toCharArray(),
                "changeit".toCharArray()
            )
            .loadTrustMaterial(