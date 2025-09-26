# Padrão Síncrono (REST/gRPC)

Define comunicação síncrona entre sistemas, onde o cliente aguarda a resposta do servidor antes de prosseguir.

## Sumário

- [Padrão Síncrono](#padrão-síncrono)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot (REST)](#exemplo-com-spring-boot-rest)
  - [Exemplo com gRPC](#exemplo-com-grpc)

## Componentes

- **Cliente**: Inicia a chamada e espera resposta.
- **Servidor**: Processa requisição e retorna resposta.
- **Protocolo**: REST (HTTP) ou gRPC (HTTP/2).
- **Payload**: Dados enviados/recebidos (JSON, Protobuf).

## Benefícios

- Simplicidade: Fluxo direto, fácil depuração.
- Feedback imediato: Cliente recebe resposta instantânea.
- gRPC: Alta performance, menor latência, tipagem forte.
- REST: Ampla adoção, interoperabilidade.

## Contras

- Bloqueio: Cliente aguarda, podendo causar atrasos.
- Acoplamento temporal: Cliente e servidor devem estar disponíveis.
- gRPC: Curva de aprendizado, menos suporte em ferramentas.
- REST: Overhead de HTTP, maior latência em grandes payloads.

## Quando Usar

Use em sistemas que requerem respostas imediatas (ex.: APIs REST para frontends, gRPC para microservices com alta performance).

## Exemplo com Spring Boot (REST)

### Configuração do Controlador REST

```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class SyncController {
    @GetMapping("/api/sync")
    public String getData(@RequestParam String input) {
        return "Resposta síncrona para: " + input;
    }
}
```

### Cliente REST

```java
package com.example.demo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class SyncService {
    private final RestTemplate restTemplate;

    @Autowired
    public SyncService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;