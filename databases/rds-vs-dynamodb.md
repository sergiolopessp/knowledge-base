# RDS vs. DynamoDB: Critérios de Escolha e Casos de Uso

Este documento estabelece as diretrizes para a escolha entre Amazon RDS (Relacional) e Amazon DynamoDB (NoSQL) em arquiteturas Cloud Native, baseando-se em padrões de acesso e requisitos de consistência.

## Visão Geral

A escolha do banco de dados deve ser orientada pelo **padrão de acesso (Access Pattern)** e não pela familiaridade da equipe com a tecnologia.

| Característica | Amazon RDS (SQL) | Amazon DynamoDB (NoSQL) |
| :--- | :--- | :--- |
| **Modelo de Dados** | Tabelas com esquemas fixos e relações. | Chave-valor / Documento (Schemaless). |
| **Escalabilidade** | Vertical (Read Replicas para leitura). | Horizontal (virtualmente ilimitada). |
| **Consistência** | Forte (ACID compliant). | Eventual por padrão (Forte opcional). |
| **Consultas** | Flexíveis (Joins, Ad-hoc queries). | Previsíveis (Baseadas em Partition/Sort Key). |
| **Performance** | Latência varia com a complexidade da query. | Latência consistente em milissegundos (single-digit). |

## Quando escolher Amazon RDS?

O RDS é a escolha correta quando a integridade referencial e a flexibilidade de consulta são primordiais.

- **Consultas Complexas:** Quando você precisa realizar `JOINs` entre múltiplas entidades ou agregações dinâmicas (ex: relatórios de BI).
- **Esquema Rígido:** Dados que exigem validação estrita a nível de banco de dados.
- **Sistemas Tradicionais:** ERPs, sistemas financeiros, folha de pagamento e aplicações onde as relações entre os dados são o ponto central.
- **Garantias ACID:** Quando a atomicidade e o isolamento de transações complexas não podem ser comprometidos.

## Quando escolher Amazon DynamoDB?

O DynamoDB brilha em cenários de alta escala e sistemas distribuídos/serverless.

- **Escala Massiva:** Aplicações com milhões de usuários onde o volume de escrita/leitura é imprevisível ou muito alto.
- **Acessos Previsíveis:** Onde você sabe exatamente como os dados serão buscados (ex: `GetItem` por ID).
- **Latência Crítica:** Aplicações de tempo real, carrinhos de compras, sessões de usuário e sistemas de apostas/jogos.
- **Arquitetura Serverless:** Integração nativa com AWS Lambda, SQS e EventBridge, sem preocupação com exaustão de pool de conexões.

## Abordagem Híbrida (Best Practice)

Em arquiteturas modernas de microsserviços, é comum utilizar ambos através da **Persistência Poliglota**:
1. **DynamoDB:** Para a camada operacional de alta performance (fast path).
2. **RDS:** Para a camada de análise, auditoria e relatórios complexos.

> **Dica de Arquiteto:** Evite tentar emular um modelo relacional dentro do DynamoDB usando múltiplos índices globais secundários (GSIs) se a sua aplicação exige consultas altamente dinâmicas. O custo de manutenção e a complexidade do código não compensarão a performance.

## Referências
- [RDS vs DynamoDB: When do I Choose Which? - Dev.to](https://dev.to/tuneshman/rds-vs-dynamodb-when-do-i-choose-which-8cf)
- [Amazon DynamoDB - Melhores Práticas](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)
- [Amazon RDS Documentation](https://docs.aws.amazon.com/rds/index.html)

---
