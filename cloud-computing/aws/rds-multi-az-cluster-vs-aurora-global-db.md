# RDS Multi-AZ DB Clusters vs. Amazon Aurora Global Database: Comparativo de HA e DR

## Visão Geral

Este documento resume a comparação entre as duas principais estratégias de alta disponibilidade (HA) e recuperação de desastres (DR) para bancos de dados relacionais gerenciados na AWS: o recém-lançado **RDS Multi-AZ DB Cluster** (que oferece um cluster de banco de dados com uma arquitetura de armazenamento distribuída e Multi-AZ) e o **Amazon Aurora Global Database** (focado em replicação cross-region para DR rápido).

O objetivo é guiar a decisão sobre qual arquitetura escolher com base nos requisitos de **RTO (Recovery Time Objective)**, **RPO (Recovery Point Objective)**, e **latência de leitura global**.

---

## 1. RDS Multi-AZ DB Cluster (com 2 Writers ou 1 Writer + 1 Reader)

Esta é a evolução da implantação Multi-AZ padrão do RDS, projetada para *Maior Disponibilidade* dentro de uma *Única Região*.

| Característica | Detalhe |
| :--- | :--- |
| **Foco** | Alta Disponibilidade (HA) e Durabilidade na Região. |
| **Arquitetura** | Cluster com Múltiplos Nós em Múltiplas AZs, usando um Volume de Armazenamento Único (em contraste com o Multi-AZ clássico que usava EBS de replicação síncrona). |
| **Nós (Exemplo)** | 1 Writer (AZ-A) + 1 Reader (AZ-B) + 1 Node de Failover (AZ-C). |
| **Failover (RTO)** | Muito rápido, tipicamente **~35 segundos** (comparável a algumas versões do Aurora). |
| **Latência de Leitura** | Baixa, pois as réplicas de leitura estão na mesma região (Replicação Síncrona). |
| **Custo** | Geralmente menor que o Global Database, mas maior que o Multi-AZ clássico. |
| **Casos de Uso** | Aplicações de missão crítica que requerem o **menor RTO possível em uma única região** e o RPO é zero (replicação síncrona). |

---

## 2. Amazon Aurora Global Database

Esta arquitetura é projetada para *Recuperação de Desastres (DR)* e *Leitura Global*, estendendo o cluster Aurora para *Múltiplas Regiões*.

| Característica | Detalhe |
| :--- | :--- |
| **Foco** | Recuperação de Desastres (DR) e Latência de Leitura Cross-Region. |
| **Arquitetura** | Cluster Primário (Região A) + Clusters Secundários (Regiões B, C, etc.) usando a replicação **Aurora Storage Layer** de baixo overhead. |
| **Replicação** | Assíncrona, mas muito rápida (latência tipicamente menor que 1 segundo). |
| **Failover (RTO)** | Rápido para DR Cross-Region: o cluster secundário pode ser promovido para Writer em **menos de 1 minuto** (melhor que o Cross-Region Snapshot). |
| **RPO** | Baixo, mas **não é zero** devido à replicação assíncrona. O RPO é medido em segundos. |
| **Latência de Leitura** | Extremamente baixa para usuários próximos às Regiões Secundárias. |
| **Casos de Uso** | 1. **DR de Região**: Necessidade de sobreviver a uma falha total de Região. 2. **Leitura Global**: Aplicações com usuários distribuídos globalmente, necessitando de leituras de baixa latência. |

---

## 3. Matriz de Decisão Rápida

| Requisito Chave | Escolha Preferencial | Observação de Engenharia |
| :--- | :--- | :--- |
| **HA (Disponibilidade) em uma Única Região** | **RDS Multi-AZ DB Cluster** | RTO é minimizado (tipicamente < 35s), RPO é zero (síncrono). |
| **DR (Disaster Recovery) Cross-Region** | **Aurora Global Database** | Promove um cluster secundário em outra região em < 1 minuto. RPO é baixo (segundos). |
| **Redução de Latência Global de Leitura** | **Aurora Global Database** | Oferece réplicas de leitura em regiões geograficamente próximas aos usuários. |
| **Manter a Arquitetura RDS (Não-Aurora)** | **RDS Multi-AZ DB Cluster** | Se não puder migrar para Aurora (ex: dependência de recursos específicos do RDS nativo). |
| **Custo como Fator Primário** | *(Avaliação Caso a Caso)* | O Multi-AZ DB Cluster é complexo de precificar, mas o Global Database adiciona custo de inter-região e mais instâncias. |

---

## 4. Conclusão do Especialista (Meu Ponto de Vista)

Como um profissional com foco em resiliência e sistemas distribuídos, minha regra de ouro é:

1.  **Se a preocupação principal é *Resiliência Regional* (HA), use o RDS Multi-AZ DB Cluster.** Ele é uma melhoria robusta sobre o Multi-AZ clássico, oferecendo um *failover* extremamente rápido e síncrono.
2.  **Se a preocupação é *Continuidade de Negócio após Falha de Região* (DR) e/ou *Latência Global*, o Aurora Global Database é o padrão-ouro.** Ele é a opção mais madura e rápida da AWS para DR Cross-Region, um requisito crítico em sistemas de larga escala.

Em ambientes *mission-critical* de alta performance (o meu foco), o **Aurora Global Database** geralmente é a escolha para DR, complementando a resiliência já inerente do cluster Aurora dentro da região.