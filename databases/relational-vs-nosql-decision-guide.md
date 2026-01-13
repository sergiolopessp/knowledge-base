# Guia de Decisão: Bancos de Dados Relacionais (SQL) vs. Não-Relacionais (NoSQL)

Este documento sintetiza os critérios para escolha de persistência de dados, equilibrando trade-offs entre consistência, escalabilidade e flexibilidade.

## 1. Contextualização Técnica

Em mais de três décadas de engenharia, vimos o pêndulo oscilar do domínio absoluto do modelo relacional para a explosão do NoSQL e, agora, para o equilíbrio do "Polyglot Persistence". A escolha não é sobre "qual é melhor", mas sobre "qual se ajusta ao modelo de dados e requisitos não-funcionais".

## 2. Visão Comparativa

| Atributo | Relacional (SQL) | Não-Relacional (NoSQL) |
| :--- | :--- | :--- |
| **Esquema** | Rígido (Fixed Schema). Requer normalização. | Flexível (Schema-less/Dynamic). |
| **Escalabilidade** | Vertical (Scale-up). Sharding é complexo. | Horizontal (Scale-out). Nativo para clusters. |
| **Garantias** | ACID (Atomicidade, Consistência, Isolamento, Durabilidade). | BASE (Basically Available, Soft state, Eventual consistency). |
| **Transações** | Complexas e robustas entre múltiplas tabelas. | Geralmente limitadas ao nível do documento/chave. |

## 3. Quando Escolher Relacional (MySQL, PostgreSQL, Oracle)
* **Consistência é Inegociável:** Sistemas financeiros, core banking e ERPs onde a integridade referencial e o suporte a transações ACID são críticos.
* **Dados Estruturados:** Quando o modelo de dados é previsível e pouco volátil.
* **Consultas Complexas:** Necessidade de Joins pesados e relatórios ad-hoc complexos.
* **Ecossistema Java:** Excelente suporte via JPA/Hibernate e JDBC.

## 4. Quando Escolher Não-Relacional (MongoDB, Redis, Cassandra)
* **Velocidade de Desenvolvimento:** Projetos em estágio inicial com requisitos que mudam rapidamente.
* **Big Data & Real-time:** Aplicações de chat, redes sociais, IoT ou catálogos de produtos com milhões de itens e atributos variados.
* **Escalabilidade Global:** Aplicações Cloud-Native que precisam distribuir dados geograficamente com baixa latência (Horizontal Scaling).
* **Caching & Session Management:** Uso de Key-Value stores (Redis) para performance extrema.

## 5. Perspectiva "Docker Captain" & Cloud
No cenário moderno de containers e nuvem, considere:
* **Persistência no Docker:** Bancos SQL costumam ser mais sensíveis a latência de IO em volumes. Certifique-se de usar drivers de storage performáticos e gerenciar o estado via StatefulSets em Kubernetes.
* **Arquitetura Cloud-Native:** Considere bancos gerenciados (RDS, DynamoDB, CosmosDB) para reduzir o overhead operacional.
* **Observabilidade:** Bancos NoSQL costumam oferecer métricas de throughput mais granulares, facilitando o auto-scaling do cluster.

## 6. Checklist de Decisão (O "Dream Project")

1.  **O dado é tabular?** Sim → SQL.
2.  **Preciso de consistência imediata em todo o sistema?** Sim → SQL.
3.  **O volume de escrita é massivo e precisa de escala horizontal simples?** Sim → NoSQL.
4.  **A estrutura do dado muda toda semana?** Sim → NoSQL.

---
**Referências:**
* [Relational vs. Non-Relational Databases: Choosing the Right One](https://dev.to/hamidrazadev/relational-vs-non-relational-databases-choosing-the-right-one-for-your-dream-project-3a94)
* Experiência acumulada em sistemas distribuídos e certificação Java.