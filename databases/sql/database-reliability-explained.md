# Confiabilidade de Banco de Dados (Database Reliability)

**Tópicos Chave:** Transações (ACID), Deadlocks, Recuperação Baseada em Log (WAL).

**Contexto:** Este resumo explora os pilares fundamentais da confiabilidade de Bancos de Dados, essenciais para qualquer sistema de missão crítica, especialmente em ambientes de **Cloud** e **Microserviços** onde a consistência e a resiliência são primordiais.

---

## 1. Transações e o Paradigma ACID

Uma **Transação** é uma sequência de operações de banco de dados tratadas como uma única unidade lógica de trabalho. A confiabilidade de um DB é medida pela adesão aos princípios **ACID**:

| Princípio | Descrição | Implicação em Engenharia (Cloud/Distributed) |
| :---: | :--- | :--- |
| **A**tomicity | Uma transação deve ser executada completamente ou não ser executada. | Se o serviço falhar (ex: crash de um microsserviço no Kubernetes), o banco deve garantir que o estado é o *anterior* à transação (Rollback). |
| **C**onsistency | A transação deve levar o DB de um estado válido para outro. | Garante que regras de negócio e restrições (ex: chaves estrangeiras) sejam mantidas. Essencial ao lidar com **integridade de dados** em diferentes serviços. |
| **I**solation | Múltiplas transações simultâneas não devem interferir umas nas outras. | Controlado por Níveis de Isolamento (Read Committed, Serializable, etc.). Em **Go** ou **Java** (Spring/Hibernate), gerenciar o nível de isolamento correto é vital para *performance* vs. *integridade*. |
| **D**urability | Uma vez que a transação é confirmada (Commit), as mudanças são permanentes, mesmo em caso de falha. | Requer gravação em armazenamento *não volátil* (ex: discos persistentes em Cloud) e é onde o **Write-Ahead Log (WAL)** se torna crucial. |

---

## 2. O Desafio dos Deadlocks

Um **Deadlock** ocorre quando duas ou mais transações esperam indefinidamente por um recurso que a outra transação está segurando.

* **Causa:** Geralmente, resultado de um bloqueio circular de recursos (locks).
* **Detecção:** Bancos de dados utilizam algoritmos de detecção baseados em **Wait-For Graphs**.
* **Resolução (Ação do DB):** O banco de dados escolhe uma das transações (a **vítima**) e a **Rollback** para que a outra possa prosseguir.
* **Melhores Práticas (Visão de SW Eng):**
    * Sempre acessar os recursos (tabelas/linhas) **na mesma ordem** em todo o código (uniformidade reduz a chance de ciclo).
    * Manter as transações o mais **curtas** possível, minimizando o tempo de retenção de locks. *Isto é especialmente crítico em microsserviços onde a latência é variável*.
    * Considerar padrões alternativos, como **Retry Logic** ou **Eventual Consistency** (padrão **Saga**), para evitar transações distribuídas de longa duração.

---

## 3. Recuperação Baseada em Log (Write-Ahead Log - WAL)

O **Write-Ahead Log (WAL)** é a espinha dorsal da **Durability** e da **Recuperação** do banco de dados.

* **Princípio WAL:** Nenhuma alteração de dados no disco principal pode ser escrita até que o registro dessa alteração tenha sido escrito no Log.
* **Como Funciona:**
    1.  Uma transação altera dados na **memória** (buffer pool).
    2.  O registro de alteração (*log record*) é escrito no **WAL** em disco (Commit).
    3.  Somente *depois* do log ser persistido, o banco retorna sucesso (*Durability* garantida).
* **Processo de Recuperação (Crash):**
    1.  **Redo Phase:** O sistema refaz todas as transações que foram commitadas (estão no WAL) mas cujas alterações ainda não foram gravadas nos arquivos de dados.
    2.  **Undo Phase:** O sistema desfaz (Rollback) todas as transações que *não* foram commitadas (não têm registro de commit no WAL).

---

## 4. Implicações em Cloud e Sistemas Distribuídos

Pensando em Cloud:

1.  **Resiliência (Cloud-Native):** A recuperação (Redo/Undo) deve ser **rápida**. Bancos de dados Cloud-nativos (como Aurora, CockroachDB, Spanner) otimizam a arquitetura WAL/Log para desacoplar o armazenamento do processamento, facilitando a escalabilidade e a recuperação em caso de falha de **AZ** (Availability Zone) ou **Node**.
2.  **Linguagens (Java/Go/C++):**
    * Em **Java** (ex: usando JDBC), a correta gestão do ``Connection.setAutoCommit(false)`` e o uso de blocos ``try-catch-finally`` para garantir ``commit()`` ou ``rollback()`` é fundamental para a Atomicidade.
    * Em **Go**, o manuseio de `tx.Commit()` e `tx.Rollback()` em defer statements é uma prática comum e robusta.
    * Em **C++**, a gestão de recursos (locks e transações) exige um controle de baixo nível mais rigoroso, sendo essencial para sistemas de alta performance/baixa latência.
3.  **Containers (Docker):** A persistência dos dados do WAL e dos dados principais deve ser garantida via **Volumes** (ou **Persistent Volume Claims** no Kubernetes) para que a Durability seja mantida mesmo se o container falhar e for reiniciado.

---

## Referências

* [Artigo Original: Database Reliability Explained: Transactions, Deadlocks, Log-Based Recovery](https://dev.to/thushitha_tk_ffd2acfb6067/database-reliability-explained-transactions-deadlocks-log-based-recovery-3kle)
* [Padrão Saga - Eventual Consistency em Microsserviços](link-para-um-doc-saga-na-sua-base)