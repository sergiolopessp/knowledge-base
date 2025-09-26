# Padrão Centralized Logging

O Padrão Centralized Logging coleta, armazena e analisa logs de múltiplos sistemas em um único repositório, facilitando monitoramento, depuração e auditoria.

## Sumário

- [Padrão Centralized Logging](#padrão-centralized-logging)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot e ELK Stack](#exemplo-com-spring-boot-e-elk-stack)
    - [Aplicação Spring Boot](#aplicação-spring-boot)
    - [Configuração de Logstash](#configuração-de-logstash)
    - [Visualização com Kibana](#visualização-com-kibana)

## Componentes

- **Aplicações/Serviços**: Geram logs (ex.: microservices, monolitos).
- **Agente de Coleta**: Encaminha logs para o repositório central (ex.: Logstash, Fluentd).
- **Repositório Central**: Armazena logs (ex.: Elasticsearch).
- **Ferramenta de Visualização**: Analisa e exibe logs (ex.: Kibana, Grafana).

## Benefícios

- Monitoramento unificado.
- Depuração simplificada.
- Rastre