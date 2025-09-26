# Padrão Rate Limiting

Estratégia para controlar o número de requisições a um serviço em um período, protegendo contra abuso e sobrecarga.

## Sumário

- [Padrão Rate Limiting](#padrão-rate-limiting)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)

## Componentes

- **Limite**: Número máximo de requisições por período (ex.: 100/hora).
- **Contador**: Rastreia requisições por usuário ou endpoint.
- **Algoritmo**: Define lógica (ex.: Token Bucket, Leaky Bucket).
- **Middleware**: Aplica limite (ex.: API Gateway, biblioteca).

## Benefícios

- Proteção: Evita sobrecarga e ataques (ex.: DDoS).
- Estabilidade: Mantém desempenho sob alta demanda.
- Justiça: Garante acesso equitativo aos recursos.
- Escalabilidade: Controla uso em sistemas distribuídos.

## Contras

- Complexidade: Configuração e ajuste do limite.
- Experiência do usuário: Requisições legítimas podem ser bloqueadas.
- Manutenção: Requer monitoramento e ajustes frequentes.
- Overhead: Adiciona latência para verificar limites.

## Quando Usar

Use em APIs públicas, microservices ou sistemas expostos a alto tráfego para evitar abuso e garantir disponibilidade.

## Exemplo com Spring Boot

### Configuração