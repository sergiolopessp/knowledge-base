# Formatos de Dados Essenciais para Engenharia e Análise

## TL;DR

A escolha do formato de dados correto é uma decisão arquitetural crítica que impacta performance, custo de armazenamento, escalabilidade e facilidade de integração entre sistemas. Este documento cobre os formatos mais comuns, desde os textuais legíveis por humanos (JSON, XML, CSV) até os formatos binários otimizados para Big Data (Parquet, Avro).

---

## Formatos Baseados em Texto

São legíveis por humanos, o que os torna excelentes para desenvolvimento, depuração e arquivos de configuração.

### 1. JSON (JavaScript Object Notation)

Padrão de mercado para APIs web e comunicação entre microsserviços. Leve e fácil de manipular em praticamente todas as linguagens modernas.

**Estrutura**: Pares de chave-valor e listas ordenadas.

**Exemplo**:
```json
{
  "id_usuario": 101,
  "nome": "Sérgio",
  "ativo": true,
  "grupos": [
    "admin",
    "developer"
  ]
}
```

-   **Prós**:
    -   Altamente legível e simples.
    -   Estrutura hierárquica (suporta objetos aninhados).
    -   Facilmente parseado pela maioria das linguagens.
-   **Contras**:
    -   Não é o mais compacto (verboso).
    -   Não possui um schema formal nativo (validação depende da aplicação).

### 2. XML (eXtensible Markup Language)

Mais antigo e verboso que o JSON. Usa tags para definir elementos. Era o padrão para serviços web (SOAP), mas hoje é mais comum em sistemas legados, configurações (ex: `pom.xml` do Maven) e documentos complexos.

**Estrutura**: Árvore de nós com tags, atributos e conteúdo.

**Exemplo**:
```xml
<usuario id="101">
  <nome>Sérgio</nome>
  <ativo>true</ativo>
  <grupos>
    <grupo>admin</grupo>
    <grupo>developer</grupo>
  </grupos>
</usuario>
```

-   **Prós**:
    -   Suporte a schemas (XSD) para validação de estrutura.
    -   Suporta namespaces e comentários.
-   **Contras**:
    -   Extremamente verboso, consome mais espaço e banda.
    -   Parsing mais complexo e lento que JSON.

### 3. CSV (Comma-Separated Values)

Formato tabular simples, ideal para exportar dados de planilhas e bancos de dados. Cada linha é um registro e cada coluna é separada por um delimitador (geralmente vírgula).

**Estrutura**: Linhas de texto plano com um delimitador.

**Exemplo**:
```csv
id_usuario,nome,ativo
101,Sérgio,true
102,Ana,false
```

-   **Prós**:
    -   Extremamente compacto e simples.
    -   Fácil de gerar e ler.
    -   Amplamente suportado por praticamente todas as ferramentas de dados.
-   **Contras**:
    -   Não possui estrutura de dados (sem tipos, sem hierarquia).
    -   Frágil: problemas com delimitadores dentro dos próprios dados.

---

## Formatos Binários Otimizados (Big Data)

Projetados para performance e armazenamento eficiente em grandes volumes de dados. Não são legíveis por humanos.

### 4. Apache Parquet

Um formato colunar. Em vez de armazenar dados por linha (como no CSV), ele armazena por coluna. Isso é extremamente eficiente para queries analíticas que leem apenas um subconjunto de colunas de uma tabela larga.

-   **Prós**:
    -   **Compressão superior**: Como dados na mesma coluna são do mesmo tipo, a compressão é muito mais eficaz.
    -   **Performance de leitura**: Queries que selecionam colunas específicas (`SELECT nome, email FROM...`) são muito rápidas, pois não precisam ler o disco inteiro.
    -   Padrão de fato no ecossistema Hadoop/Spark.
-   **Contras**:
    -   Não é legível por humanos.
    -   Overhead para escritas pequenas e pontuais (otimizado para *bulk writes*).

### 5. Apache Avro

Um formato baseado em linha. Foi projetado com foco em evolução de schema. Ele armazena o schema (em formato JSON) junto com os dados, o que permite que a estrutura dos dados mude ao longo do tempo sem quebrar os sistemas consumidores.

-   **Prós**:
    -   **Evolução de schema robusta**: Permite adicionar, remover ou modificar campos de forma retrocompatível. Essencial para pipelines de dados em produção.
    -   Compacto e rápido na serialização/desserialização.
    -   Excelente para streaming de dados (padrão em Kafka).
-   **Contras**:
    -   Menos eficiente que o Parquet para queries analíticas de colunas específicas.
    -   O schema sempre faz parte dos dados, o que pode gerar um pequeno overhead.

## Tabela Comparativa Rápida

| Formato | Tipo | Legibilidade | Schema | Principal Caso de Uso |
| :--- | :--- | :--- | :--- | :--- |
| **JSON** | Texto | Humano | Não nativo | APIs, Microsserviços |
| **XML** | Texto | Humano | Sim (XSD) | Sistemas legados, Configurações |
| **CSV** | Texto | Humano | Não | Exportação de dados, Planilhas |
| **Parquet** | Binário | Não | Sim | Data Warehousing, Analytics (OLAP) |
| **Avro** | Binário | Não | Sim | Streaming de dados (Kafka), Data Lakes |

### Dica de Engenheiro

A escolha não é sobre "qual é o melhor?", mas "qual é o melhor *para este job*?".
-   Para uma API REST que será consumida por um front-end? **JSON**, sem dúvida.
-   Para um dump diário de 10TB de um data warehouse que será analisado por cientistas de dados? **Parquet** vai economizar tempo e dinheiro.
-   Para um tópico no Kafka que recebe eventos de milhões de dispositivos e cujo schema pode mudar na próxima semana? **Avro** é a escolha segura e robusta.

Entender o trade-off entre legibilidade, performance e flexibilidade de schema é o que diferencia uma implementação de software comum de uma arquitetura de dados resiliente.