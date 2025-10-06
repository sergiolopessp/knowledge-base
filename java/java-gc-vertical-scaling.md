# Java Vertical Scaling com GCs Modernos (ZGC & Shenandoah)

**Autor:** Sergio Lopes | **Data:** Outubro de 2025
**Referências:** [DZone - Charge Vertical Scaling With the Latest Java GCs](https://dzone.com/articles/charge-vertical-scaling-with-the-latest-java-gcs)

## Contexto

Durante anos, a escolha do Garbage Collector (GC) impôs um "teto de vidro" na escalabilidade vertical (aumento de recursos de CPU/Memória) de aplicações Java, especialmente devido às pausas (Stop-The-World - STW) prolongadas em heaps de grande volume. Com a arquitetura Cloud moderna e memória na casa dos Terabytes, os GCs legados simplesmente não são viáveis para cargas de trabalho de baixa latência.

As gerações de GCs **ZGC** e **Shenandoah** representam uma mudança de paradigma, permitindo que a JVM utilize recursos massivos de hardware sem comprometer o SLO (Service Level Objective) de latência.

---

## Principais GCs para Vertical Scaling

Ambos GCs visam atingir pausas de coleta de lixo consistentemente baixas (geralmente abaixo de 10ms), independentemente do tamanho do heap. Isso é o que desbloqueia a verdadeira escalabilidade vertical em *containers* com grandes alocações de memória.

### 1. Z Garbage Collector (ZGC)

| Característica | Detalhe |
| :--- | :--- |
| **Introdução** | JDK 11 (experimental), Produção estável no JDK 15. |
| **Objetivo** | Ultra-baixa latência e suporte a heaps muito grandes (multi-terabytes). |
| **Mecanismo Chave** | Quase todo o trabalho de GC é realizado **concorrentemente** com a aplicação. Utiliza técnicas como **Colored Pointers** e **Load Barriers**. |
| **Pause Time** | Projetado para pausas **abaixo de 10ms** de forma consistente. |
| **Uso Ideal** | Aplicações que demandam latência extrema e operam com heaps gigantescos (ex: bancos de dados in-memory, grandes caches). |
| **Flag JVM** | `-XX:+UseZGC` |

### 2. Shenandoah GC

| Característica | Detalhe |
| :--- | :--- |
| **Introdução** | JDK 12 (experimental), Produção estável no JDK 15. |
| **Objetivo** | Minimização de pause time, com independência do tamanho do heap. |
| **Mecanismo Chave** | Realiza a compactação do heap **concorrentemente**. Utiliza um ponteiro de indireção adicional para cada objeto Java, permitindo que a aplicação e o GC se movam de forma independente. |
| **Pause Time** | Mantém pausas curtas e previsíveis (<< 10ms), seja o heap de 200MB ou 200GB. |
| **Uso Ideal** | Aplicações com heaps grandes onde a **previsibilidade** e consistência das pausas são mais importantes que a latência absoluta. Excelente para microsserviços em *containers*. |
| **Flag JVM** | `-XX:+UseShenandoahGC` |

---

## Implicação para a Engenharia Moderna

A grande sacada desses GCs não é apenas a performance, mas o **custo-benefício operacional**.

1.  **Scaling sem Rewrite:** Permite que empresas atinjam performance e escalabilidade de forma **vertical** (adicionando mais memória/CPU ao container existente) sem a necessidade de reescrever código complexo para escalabilidade horizontal.
2.  **Configuração de Infra:** O *tuning* de performance se move da camada de código de aplicação para a camada de configuração da JVM, simplificando o processo para equipes de SRE e DevOps.
3.  **Ambientes Cloud:** O uso eficiente de memória em VMs maiores (porém mais caras) se torna viável, otimizando a densidade de aplicações por host e, em muitos casos, reduzindo o custo total de propriedade (TCO) ao evitar a complexidade do *sharding* e distribuição.

**Recomendação:** Em projetos novos ou na modernização de legados, considere o ZGC/Shenandoah como ponto de partida para a estratégia de *tuning* da JVM. No universo de containers e orquestração (Docker/Kubernetes), a previsibilidade de pausas do Shenandoah o torna um forte candidato.