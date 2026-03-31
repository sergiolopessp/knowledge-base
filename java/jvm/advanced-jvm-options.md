# Advanced JVM Options: Performance, Memory & Tuning

Como engenheiros de software, sabemos que a JVM é uma das máquinas virtuais mais robustas do mercado, mas seu comportamento "out-of-the-box" nem sempre é o ideal para workloads de alta performance ou ambientes de nuvem restritos. 

Este guia consolida o entendimento sobre as flags de runtime para otimização de performance, gerenciamento de memória e diagnóstico.

## 1. Categorias de Flags da JVM

As opções da JVM são divididas em três prefixos principais, cada um com um nível de estabilidade e suporte:

| Prefixo | Nome | Descrição |
| :--- | :--- | :--- |
| `-` | **Standard** | Opções padrão suportadas por todas as implementações da JVM. |
| `-X` | **Non-Standard** | Opções específicas da HotSpot. Podem mudar entre versões sem aviso prévio. |
| `-XX` | **Advanced** | Opções de controle granular para tuning de GC, JIT e limites do sistema. |

---

## 2. Gerenciamento de Memória (Heap & Stack)

O ajuste correto do Heap evita overhead de redimensionamento e falhas de `OutOfMemoryError` (OOM).

### Configurações de Heap
* `-Xms<size>`: Define o tamanho inicial do Heap.
* `-Xmx<size>`: Define o tamanho máximo do Heap.
    * **Best Practice:** Em produção, defina `-Xms` igual ao `-Xmx` para evitar pausas de GC causadas pelo redimensionamento do Heap pelo S.O.
* `-Xmn<size>`: Define o tamanho da *Young Generation*. Útil para aplicações com alta taxa de alocação de objetos de vida curta.

### Stack e Metaspace
* `-Xss<size>`: Define o tamanho da Stack por Thread. Reduzir este valor permite criar mais threads em máquinas com RAM limitada, mas aumenta o risco de `StackOverflowError`.
* `-XX:MetaspaceSize` e `-XX:MaxMetaspaceSize`: Controlam a área de memória que armazena metadados de classes (substituiu o antigo PermGen no Java 8+).

---

## 3. Seleção de Garbage Collector (GC)

A escolha do GC deve ser baseada no balanço entre **Throughput** (Vazão) e **Latency** (Latência).

| Flag | GC | Caso de Uso |
| :--- | :--- | :--- |
| `-XX:+UseG1GC` | **G1 (Garbage First)** | Equilíbrio entre latência e throughput. Padrão no Java 9+. |
| `-XX:+UseZGC` | **ZGC** | Ultra-low latency (pausas < 1ms), ideal para Heaps gigantescos. |
| `-XX:+UseParallelGC` | **Parallel** | Foco em throughput total (jobs em batch), aceitando pausas maiores. |
| `-XX:+UseSerialGC` | **Serial** | Ambientes com apenas 1 core ou micro-containers extremamente pequenos. |

---

## 4. Otimização em Containers (Docker/K8s)

Como **Docker Captain**, é vital garantir que a JVM não ignore os limites de recursos do cgroup.

* `-XX:+UseContainerSupport`: (Habilitado por padrão no Java 8u191+) Garante que a JVM leia os limites de memória e CPU do container em vez do host físico.
* `-XX:MaxRAMPercentage=80.0`: Define o Heap como uma porcentagem da memória total do container. Muito mais flexível do que fixar valores em MB com `-Xmx`.

---

## 5. Diagnóstico e Troubleshooting em Produção

Flags essenciais para "post-mortem" debugging:

```bash
# Gera um dump da memória automaticamente ao ocorrer um OOM
-XX:+HeapDumpOnOutOfMemoryError 
-XX:HeapDumpPath=/var/log/java/heapdump.hprof

# Log detalhado do GC (Sintaxe para Java 9+)
-Xlog:gc*:file=/var/log/java/gc.log:time,uptime,level,tags

# Exibe todas as flags XX no log de inicialização (ajuda a auditar o ambiente)
-XX:+PrintCommandLineFlags
```

## 6. JIT (Just-In-Time) Compiler
Controle como o código é compilado para código nativo:

```bash
-XX:+TieredCompilation: Permite que o código passe por vários níveis de otimização (C1 e C2). Habilitado por padrão.

-XX:CICompilerCount=N: Define o número de threads dedicadas à compilação JIT. Útil para acelerar o warm-up de aplicações em nuvem.
```

**Referências**

[Explaining Advanced JVM Options - Java Code Geeks](https://www.javacodegeeks.com/explaining-advanced-jvm-options.html)

[Oracle HotSpot VM Options](https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html)