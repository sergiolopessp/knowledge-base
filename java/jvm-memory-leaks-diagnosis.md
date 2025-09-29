# Diagnóstico de Memory Leaks em JVM em Produção: ferramentas, técnicas e padrões de prevenção

## Introdução

Memory leaks em aplicações baseadas em JVM são um problema sutil e perigoso: muitas vezes os sintomas surgem gradualmente — degradação de performance, pausas de GC prolongadas, até culminar em `OutOfMemoryError`. Neste documento adaptamos e organizamos os conceitos do artigo *“Diagnosing JVM Memory Leaks in Production: Tools, Techniques, and Prevention Patterns”* da Java Code Geeks para uso prático e referência em tua knowledge base. ([javacodegeeks.com](https://www.javacodegeeks.com/2025/09/diagnosing-jvm-memory-leaks-in-production-tools-techniques-and-prevention-patterns.html))

O objetivo é permitir que equipes com forte experiência em Java tenham um guia técnico claro para **identificar**, **analisar** e **prevenir** leaks de memória em ambientes de produção.

---

## O que é um memory leak na JVM?

Mesmo com coleta de lixo (GC), é possível que objetos que **não são mais usados** continuem sendo referenciados indevidamente, impedindo a liberação de memória. Isso se diferencia de um simples alto uso de memória: é sobre **retenção indevida** (i.e. objetos “vivos” mas inúteis).

Com o tempo, isso faz o heap crescer além dos limites esperados, GC luta para recuperar espaço, pausas aumentam, a estabilidade da aplicação é comprometida e eventualmente ocorre `OutOfMemoryError`.

---

## Sintomas a observar em produção

- Uso de heap que cresce continuamente, mesmo após as coletas completas (full GC). Cada “queda” de memória não retorna ao baseline anterior.  
- Aumento na frequência de GCs completas, com cada vez menor ganho de memória recuperada.  
- Latência crescente, pausas perceptíveis da aplicação durante operações intensas de GC.  
- Crescimento gradual no uso de classes/instâncias específicos quando se compararem snapshots ao longo do tempo.

É importante distinguir vazamentos reais de cenários benignos (por exemplo, aquecimento de cachês ou crescimento legítimo de uso). O padrão que revela vazamento é exatamente a incapacidade do GC em reverter o uso de memória ao baseline ao longo do tempo.

---

## Ferramentas e técnicas para diagnóstico

| Técnica / Ferramenta | O que fornece | Como usar em produção | Cuidados / observações |
|---|---|---|---|
| **Heap Dump** | Snapshot de todos os objetos “vivos” (tipos, tamanhos, referências retidas) | Capturar um dump em momentos-chave (baseline saudável e no momento suspeito) | Pode causar pausas significativas e uso de disco elevado |
| **Análise de logs de GC** | Exibe uso de memória ao longo do tempo, latências de GC, quantidade de memória recuperada | Ativar logs detalhados (ex: `-Xlog:gc*` ou antigos `-XX:+PrintGCDetails`) e utilizar ferramentas como GCViewer, GCeasy | Logs muito verbosos ocupam espaço, rotacionamento deve ser planejado |
| **Monitoramento / Profiling leve** | Métricas em tempo real: heap, pausas de GC, instâncias por classe | Usar JMX + Prometheus, Micrometer, ou Java Flight Recorder / Mission Control em modo leve | Evitar profiling agressivo em produção |
| **Diff de snapshots** | Identificação de classes/instâncias que crescem ao longo do tempo | Comparar dumps ou histogramas de instâncias em diferentes momentos | Útil para leaks de crescimento lento |
| **Padrões de heap** | Visualizar baseline saudável vs comportamento anômalo | Observar gráfico de heap após GCs | Diferenciar crescimento normal (“saw-tooth”) de vazamento (baseline crescente) |

---

## Fluxo prático de diagnóstico

1. Estabelecer um baseline saudável logo após reinício.  
2. Ativar logs de GC e coletar métricas contínuas.  
3. Monitorar: heap retorna ao baseline ou cresce continuamente?  
4. Capturar heap dump no momento suspeito.  
5. Comparar dump com baseline e identificar classes/objetos que cresceram mais.  
6. Verificar cadeias de referência (coleções estáticas, listeners, ThreadLocals não limpos etc.).  
7. Corrigir: liberar recursos, aplicar limites em caches, usar referências fracas quando apropriado.  
8. Reimplantar e monitorar novamente.  

Exemplo: aplicação sobe de ~50 GB para ~54 GB em poucos dias, mesmo após full GC. Diagnóstico revela cache sem expiração. Correção: aplicar TTL/LRU. Heap volta a comportamento normal (“saw-tooth”).

---

## Padrões de prevenção

- **Cachês limitados / com expiração** (TTL, LRU).  
- **Uso criterioso de `WeakReference` e `SoftReference`**.  
- **Liberação explícita de recursos** (streams, conexões, ThreadLocals, listeners).  
- **Minimizar estado estático**.  
- **Evitar retenção acidental em logs/monitoramento**.  
- **Executar testes long-running** antes de produção.  

---

## Cuidados em produção

- Heap dumps/logs podem causar pausas ou overhead.  
- Dumps contêm dados sensíveis: proteger e restringir acesso.  
- Planejar retenção/rotação de dumps e logs.  
- Nem todo leak está no heap Java (DirectByteBuffer, JNI, metaspace também vazam).  
- Configuração inadequada de GC pode simular vazamento.  

---

## Exemplos úteis

- Ativar logs GC:
  ```bash
  java -Xlog:gc* -jar app.jar
  ```

- Capturar heap dump em produção:
  ```bash
  jcmd <pid> GC.heap_dump /tmp/heap.hprof
  ```

- Histograma de instâncias:
  ```bash
  jmap -histo <pid> | head -n 50
  ```

- Comparação de dumps com Eclipse MAT.  

---



## Referência

- *Diagnosing JVM Memory Leaks in Production: Tools, Techniques, and Prevention Patterns* — [Java Code Geeks](https://www.javacodegeeks.com/2025/09/diagnosing-jvm-memory-leaks-in-production-tools-techniques-and-prevention-patterns.html)
