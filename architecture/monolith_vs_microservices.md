# Arquitetura: Monolito vs. Microsserviços

**Contexto:** A escolha entre uma arquitetura **Monolítica** e **Microsserviços** é a decisão de design mais fundamental em um projeto. Esta escolha afeta diretamente a velocidade de desenvolvimento, escalabilidade, resiliência em Cloud e a estrutura da equipe.

---

## 1. O Monolito (Monolith)

Uma aplicação Monolítica é construída como uma única unidade coesa. Todos os componentes (UI, lógica de negócio, acesso a dados) são empacotados e implementados juntos.

| Vantagens (Melhor para) | Desafios (Pior para) |
| :--- | :--- |
| **Simplicidade de Início:** Ideal para MVPs e startups. | **Escalabilidade Restrita:** Para escalar uma função, toda a aplicação deve ser escalada. |
| **Desenvolvimento Rápido:** Ambiente de desenvolvimento unificado e testes End-to-End mais fáceis. | **Manutenção e Refatoração:** A complexidade aumenta exponencialmente (*Big Ball of Mud*). |
| **Transações ACID:** Fácil de implementar consistência transacional forte (Rollbacks). | **Barreira de Tecnologia:** Linguagem única (ex: **Java** monolítico); difícil adoção de **Go** ou **C++** para partes específicas. |
| **Deploy Simples:** Uma única unidade para deploy (fácil gerenciamento de **Docker** / CI/CD inicial). | **Tempo de Deploy Lento:** Pequenas mudanças exigem o deploy de toda a aplicação. |

---

## 2. Microsserviços (Microservices)

A arquitetura de Microsserviços consiste em uma coleção de pequenos serviços autônomos que se comunicam através de APIs (geralmente HTTP ou *messaging*).

| Vantagens (Melhor para) | Desafios (Pior para) |
| :--- | :--- |
| **Escalabilidade Fina:** Escala serviços individuais (*Just-in-Time* em Cloud/Kubernetes) para otimizar custos. | **Complexidade Operacional:** Requer gerenciamento robusto de **Kubernetes**, *Service Mesh* e observabilidade. |
| **Independência Tecnológica:** Permite usar a melhor linguagem para o trabalho (ex: **Go** para serviços de alta performance, **Java** para serviços empresariais). | **Consistência de Dados (SAGA):** Transações distribuídas são complexas; exige **Eventual Consistency** e padrões como **Saga**. |
| **Resiliência:** A falha de um serviço não derruba todo o sistema (*Circuit Breakers* são essenciais). | **Comunicação:** Latência e falhas de rede se tornam um problema de design fundamental. |
| **Deploy Rápido:** Times independentes podem fazer deploy de seus serviços a qualquer momento (entrega contínua de verdade). | **Testes E2E:** Difíceis de coordenar, exigindo ambientes de *staging* complexos. |

---

## 3. Quando Usar Cada Arquitetura (A Decisão Estratégica)

Como um engenheiro sênior, a decisão nunca é binária; ela é pragmática e baseada no contexto.

| Cenário | Arquitetura Recomendada | Racional Engenharia Sênior |
| :--- | :--- | :--- |
| **Produto Novo/MVP** | **Monolito** | Priorize o *Time-to-Market*. Evite a sobrecarga operacional dos microsserviços no início. Refatore mais tarde. |
| **Escala Extrema / Alto Tráfego** | **Microsserviços** | Permite escalabilidade granular e otimiza o uso de recursos de **Cloud**. Essencial para sistemas de alto volume. |
| **Equipe Grande / Domínios Claros** | **Microsserviços** | Alinha-se ao princípio de Conway. Permite que equipes independentes (ex: Time A em **Java**, Time B em **Go**) possuam e deployem seus serviços. |
| **Regras de Negócio Estáveis e Simples** | **Monolito** | Se a lógica for estável e a taxa de alteração for baixa, a complexidade dos microsserviços não se justifica. |

## 4. Estratégia de Migração (O Caminho de Transição)

A transição de Monolito para Microsserviços deve ser feita através do padrão **Strangler Fig** (Figueira Estranguladora):

1.  **Isolar o Domínio:** Identifique um domínio de negócio claro (ex: Autenticação).
2.  **Externalizar:** Construa o novo serviço (ex: em **Go**, empacotado em **Docker**) fora do monolito.
3.  **Redirecionar:** Altere o monolito para chamar o novo serviço via API (HTTP) em vez de código interno.
4.  **Repetir:** Continue "estrangulando" o monolito, componente por componente.

Esta abordagem minimiza o risco de refatoração massiva (*Big Bang*) e permite que você aproveite as vantagens da **Cloud** e das novas tecnologias gradualmente.

---

## Implicações em Cloud e DevOps 

* **Docker:** É o habilitador fundamental dos microsserviços. Cada serviço é um container **Docker** independente.
* **Kubernetes (K8s):** É o orquestrador necessário. Ele gerencia o deploy, a auto-cura (resiliência), a escala e a rede (*Service Discovery*) entre centenas de containers.
* **Observabilidade:** Microsserviços exigem *Tracing Distribuído* (Jaeger, Zipkin) para entender o fluxo de transações entre os serviços escritos em diferentes linguagens (**Java**, **Go**, **C++**).