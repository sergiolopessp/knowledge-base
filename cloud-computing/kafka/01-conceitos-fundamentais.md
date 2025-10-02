# Apache Kafka: Conceitos Fundamentais

O Apache Kafka é uma plataforma de *event streaming* distribuída, projetada para lidar com grandes volumes de dados em tempo real, permitindo que aplicações publiquem, assinem, armazenem e processem fluxos de eventos. Ele serve como o "sistema nervoso central" para dados em movimento em arquiteturas modernas, como microsserviços e sistemas em Cloud.

---

## 1. Evento (Mensagem/Registro)

É a menor unidade de dados, representando algo que aconteceu (ex.: clique em um site, leitura de sensor, transação).

| Componente | Descrição |
| :--- | :--- |
| **Chave (Key)** | Opcional, mas crucial para garantir que eventos relacionados (e.g., todos os eventos de um mesmo usuário ou conta) sejam processados na **mesma partição**, mantendo a **ordem**. |
| **Valor (Value)** | O dado real do evento (geralmente serializado em JSON, Avro, Protobuf, etc.). |
| **Timestamp** | O momento em que o evento ocorreu. |

---

## 2. Tópico (Topic)

É uma categoria de eventos/mensagens. Funciona como uma tabela em um banco de dados ou uma pasta em um sistema de arquivos.

* **Log Distribuído:** Um Tópico é, fundamentalmente, um log de *commit* particionado e replicado.
* **Imutabilidade:** Os eventos são anexados de forma sequencial e não podem ser alterados após a escrita (Append-Only Log).

---

## 3. Partição (Partition)

São a unidade de paralelismo e escalabilidade do Kafka. Um Tópico é dividido em uma ou mais Partições.

* **Escalabilidade:** Permitem que o log de eventos seja distribuído entre vários Brokers.
* **Ordem:** A ordem das mensagens é garantida **somente dentro de uma partição**. Não há garantia de ordem entre partições.
* **Offset:** Cada mensagem em uma Partição possui um identificador único, sequencial e imutável, chamado **Offset**. Os Consumers usam o Offset para rastrear seu progresso de leitura.

---

## 4. Cluster e Broker (Servidor)

O Kafka opera como um sistema distribuído composto por vários servidores (máquinas).

* **Broker:** É um único servidor Kafka. Ele armazena as Partições (e suas réplicas) e lida com as solicitações dos Producers e Consumers.
* **Cluster:** É um conjunto de Brokers trabalhando juntos, oferecendo alta disponibilidade e tolerância a falhas.
* **Líder (Leader) e Réplica (Follower):** Para cada Partição, um Broker é o **Líder** (responsável por todas as operações de leitura e escrita) e os outros são **Followers** (mantêm cópias idênticas para replicação e failover).

---

## 5. Produtor (Producer)

Aplicações cliente que escrevem (publicam) dados em Tópicos.

* **Roteamento:** O Producer decide para qual Partição enviar um evento. Se uma **Key** for fornecida, por padrão, o Kafka garante que eventos com a mesma Key irão para a mesma Partição (usando um hash da Key).
* **Asynchronous:** A produção de mensagens é tipicamente assíncrona para maximizar o *throughput*.
* **Ack (Acknowledgment):** Configuração de durabilidade (ex.: `acks=all` para garantir que a mensagem foi replicada para o Líder e todos os Followers).

---

## 6. Consumidor (Consumer) e Grupo de Consumo (Consumer Group)

Aplicações cliente que leem (assinam) dados de Tópicos.

* **Consumer Group:** É um conjunto de Consumers que trabalham em conjunto para ler as mensagens de um ou mais Tópicos.
* **Paralelismo:** Em um Grupo de Consumo, cada Partição é consumida por no máximo **um** Consumer. Isso garante o processamento exclusivo e em ordem da Partição.
    * *Regra de Ouro:* O número de Consumers em um Grupo não deve exceder o número de Partições do Tópico, pois Consumers excedentes ficarão ociosos.
* **Gerenciamento de Offset:** Os Consumers (ou o Kafka, dependendo da configuração) rastreiam o último *Offset* lido para cada Partição, permitindo retomar o processamento de onde pararam em caso de falha.

---

## 7. Escalabilidade e Tolerância a Falhas

* **Escalabilidade Horizontal:** Adicionar mais Partições e Brokers permite aumentar o *throughput* de escrita e leitura.
* **Tolerância a Falhas (Replication Factor):** O número de cópias (réplicas) de cada Partição. Em ambientes de produção, `replication.factor=3` é comum para garantir que o sistema sobreviva à falha de até dois Brokers.

---

## 8. Semântica de Entrega

Kafka oferece garantias ajustáveis:

| Semântica | Descrição | Configuração Comum |
| :--- | :--- | :--- |
| **At Most Once** | Mensagens podem ser perdidas, mas nunca duplicadas. (*Baixa latência, baixa durabilidade.*) | `acks=0` |
| **At Least Once** | Mensagens não serão perdidas, mas podem ser duplicadas (em caso de falha e reenvio do Producer). (*Padrão na maioria das configurações.*) | `acks=all`, sem idempotência |
| **Exactly Once** | A mensagem é entregue e processada exatamente uma vez, mesmo em caso de falha. (*Atingida com Producers Idempotentes e Transações.*) | `enable.idempotence=true` e uso de transações no Streams API |