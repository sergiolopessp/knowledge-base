---
title: ACID vs BASE – Definições, Trade‑offs e Quando Usar
date: 2025‑09‑23
tags: database, reliability, consistency, distributed‑systems, CAP theorem
---

# ACID vs BASE: O duelo definitivo pela confiabilidade de banco de dados

Este documento resume os conceitos de **ACID** e **BASE** conforme apresentado no artigo *“ACID vs BASE: The Ultimate Showdown for Database Reliability”* de Leonard Kachi, com comentários enxertos de engenheiro de software experiente.

---

## Definições

| Acrônimo | Significado |
|---|---|
| **ACID** | *Atomicity*, *Consistency*, *Isolation*, *Durability* ([dev.to](https://dev.to/leonardkachi/acid-vs-base-the-ultimate-showdown-for-database-reliability-500c?utm_source=chatgpt.com)) |
| **BASE** | *Basically Available*, *Soft State*, *Eventually Consistent* ([dev.to](https://dev.to/leonardkachi/acid-vs-base-the-ultimate-showdown-for-database-reliability-500c?utm_source=chatgpt.com)) |

---

## Propriedades de ACID

1. **Atomicidade (Atomicity)**  
   Toda transação é "tudo ou nada": ou todas as operações dentro dela são aplicadas, ou nenhuma. Se algo falhar, desfaz‑se tudo. ([dev.to](https://dev.to/leonardkachi/acid-vs-base-the-ultimate-showdown-for-database-reliability-500c?utm_source=chatgpt.com))

2. **Consistência (Consistency)**  
   A transação deve levar o banco de um estado válido a outro, obedecendo restrições, regras, integridade. ([dev.to](https://dev.to/leonardkachi/acid-vs-base-the-ultimate-showdown-for-database-reliability-500c?utm_source=chatgpt.com))

3. **Isolamento (Isolation)**  
   Transações concorrentes não devem interferir umas nas outras. Cada transação deve parecer como se fosse a única sendo executada no sistema. ([dev.to](https://dev.to/leonardkachi/acid-vs-base-the-ultimate-showdown-for-database-reliability-500c?utm_source=chatgpt.com))

4. **Durabilidade (Durability)**  
   Uma vez que a transação é confirmada, suas alterações persistem mesmo diante de falhas de sistema, quedas de energia, etc. ([dev.to](https://dev.to/leonardkachi/acid-vs-base-the-ultimate-showdown-for-database-reliability-500c?utm_source=chatgpt.com))

---

## Propriedades de BASE

1. **Basically Available**  
   O sistema garante que toda requisição receba uma resposta (sucesso ou falha), mesmo se partes do sistema estiverem indisponíveis. ([dev.to](https://dev.to/leonardkachi/acid-vs-base-the-ultimate-showdown-for-database-reliability-500c?utm_source=chatgpt.com))

2. **Soft State**  
   O estado do sistema pode mudar ao longo do tempo, mesmo sem novas requisições explícitas, pois a consistência pode ainda estar sendo propagada ou resolvida. ([dev.to](https://dev.to/leonardkachi/acid-vs-base-the-ultimate-showdown-for-database-reliability-500c?utm_source=chatgpt.com))

3. **Eventual Consistency**  
   Se não houverem novas atualizações, eventualmente todos os nós convergirão para o mesmo valor. Ou seja: a consistência imediata não é garantida, mas espera‑se que ocorra mais tarde. ([dev.to](https://dev.to/leonardkachi/acid-vs-base-the-ultimate-showdown-for-database-reliability-500c?utm_source=chatgpt.com))

---

## Trade‑offs e CAP Theorem

- **CAP Theorem**: Em sistemas distribuídos, não é possível garantir ao mesmo tempo *Consistency*, *Availability*, e *Partition Tolerance*. Deve‑se escolher entre consistência e disponibilidade quando há particionamentos de rede inevitáveis. ([dev.to](https://dev.to/leonardkachi/acid-vs-base-the-ultimate-showdown-for-database-reliability-500c?utm_source=chatgpt.com))  
- ACID tende a priorizar *Consistency + Partition Tolerance* (sacrificando, quando necessário, *Availability*) em caso de falha de partição. ([dev.to](https://dev.to/leonardkachi/acid-vs-base-the-ultimate-showdown-for-database-reliability-500c?utm_source=chatgpt.com))  
- BASE tende a priorizar *Availability + Partition Tolerance*, permitindo temporária inconsistência. ([dev.to](https://dev.to/leonardkachi/acid-vs-base-the-ultimate-showdown-for-database-reliability-500c?utm_source=chatgpt.com))

---

## Quando usar cada modelo

| Situação / Critério | Prefira **ACID** | Prefira **BASE** |
|---|---|---|
| Integridade de dados crítica (financeiro, contabilidade, estoque) | ✅ | ❌ |
| Alta escalabilidade, distribuição geográfica, latência tolerável de leitura | ❌ | ✅ |
| Requisitos regulatórios ou compliance que exigem consistência forte sempre | ✅ | ❌ |
| Sistemas onde algum atraso ou eventual inconsistência é aceitável (feeds, logs, métricas, caches) | ❌ | ✅ |
| Resiliência frente a falhas de partição / rede | depende do tradeoff Consistency vs Availability | BASE geralmente melhor |

---

## Integração prática

Como engenheiro com experiência:

- Avaliar as garantias oferecidas pelo banco (RDBMS ou NoSQL) no contexto de contêineres ou clusters orquestrados (ex: Kubernetes) — latência de replicação, falhas de rede, etc.
- Verificar suporte transacional (ACID) nos bancos NoSQL ou híbridos — por exemplo, algumas operações/módulos do MongoDB ou Spanner fornecem transações ACID.
- Projetar partes do sistema de forma híbrida: usar ACID onde integridade é essencial, BASE para componentes que precisam escalar massivamente ou tolerar degradações temporárias.
- Monitoramento e observabilidade: rastrear divergências de consistência, latência de propagação, erros de partições, falhas de sincronização.

---

## Pontos de atenção / boas práticas

- A escolha entre ACID e BASE não é absoluta — muitas arquiteturas modernas usam ambos em diferentes domínios ou camadas.
- Entender os *SLAs* de consistência, disponibilidade e latência exigidos pelo negócio.
- Testar cenários de falha (partições de rede, queda de nós) para ver como o sistema se comporta.
- Evitar surpresas de consistência “eventual” em partes críticas do sistema; documentar claramente onde estórias ou APIs podem retornar dados não totalmente atualizados.

---

## Referências

- Leonard Kachi, *ACID vs BASE: The Ultimate Showdown for Database Reliability* ([dev.to](https://dev.to/leonardkachi/acid-vs-base-the-ultimate-showdown-for-database-reliability-500c?utm_source=chatgpt.com))  
- Artigos sobre CAP Theorem, ACID em bancos NoSQL, padrões distribuídos.
