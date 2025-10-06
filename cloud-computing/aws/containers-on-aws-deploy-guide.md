# Deploying Containers on AWS: Guia Completo

**Fonte:** *Hasan Ashab: Deploying Containers on AWS: The One Guide You Need* ([dev.to](https://dev.to/hasan_ashab/deploying-containers-on-aws-the-one-guide-you-need-18mc))

---

## Introdução

Este guia resume os diferentes serviços da AWS para deployment de containers, ajudando a escolher a solução certa com base em requisitos técnicos e custo operacional.

---

## Principais Opções da AWS para Containers

| Serviço | Quando usar / pontos fortes | Desvantagens ou trade-offs |
|---|---|---|
| **Amazon EKS (Elastic Kubernetes Service)** | Se você já usa Kubernetes ou quer portabilidade entre nuvens, necessidade de escalabilidade alta, controle sobre clusters. | Complexidade operacional, curva de aprendizado, custo de gestão de nodes, overhead de Kubernetes. |
| **Amazon ECS (Elastic Container Service)** | Gerenciamento mais simples de containers; se Docker + orquestração leve são suficientes. | Menos portabilidade comparado ao Kubernetes; menos flexível em cenários muito customizados. |
| **AWS Fargate** | Deploy serverless de containers; não precisa gerenciar hosts ou clusters. | Menos controle sobre infraestrutura, custo pode ser maior em cargas mais constantes ou pesadas. |
| **Amazon EC2 + Docker** | Máximo controle; bom para ambientes legacy ou quando se quer customização profunda. | Demanda de operações, manutenção de servidores, escalabilidade manual ou semi-manual. |
| **AWS Lightsail** | Ideal para pequenos projetos, protótipos ou MVPs que não demandam alta complexidade. | Menos recursos, menos integração com outros serviços avançados da AWS, menos flexibilidade. |
| **App Runner** | Deploy rápido e simples de aplicações containerizadas; bom para aplicações web com requisitos comuns. | Pode não atender casos de uso muito específicos ou de performance crítica; menos opções de customização de infraestrutura. |
| **Lambda (Containers)** | Para funções leves e event-driven; se você quer usar o paradigma serverless mesmo com container. | Limitações de tempo de execução, memória, e não ideal para workloads longas ou com dependências pesadas. |

---

## Passos para Escolher a Melhor Opção

1. **Entenda se você precisa de Kubernetes**  
   Se for usar muitos containers, escalabilidade, padrão de cluster, isolamento do time de infraestrutura — Kubernetes faz sentido.

2. **Serverless ou provisionado**  
   - Serverless: menos operação, escalabilidade automática, bom para variabilidade de carga.  
   - Provisionado: mais controle, embora mais overhead de gestão.

3. **Outros critérios importantes**  
   - Custo total (computação, rede, armazenamento)  
   - Tempo até deploy / facilidade de operação  
   - Necessidades de manutenção / upgrades  
   - Integração com monitoring, observabilidade, segurança  
   - Time e conhecimento disponível  

---

## Comparativo Simplificado / Fluxograma Decisório

> Perguntas-chave:  
> • Preciso de Kubernetes?  
> • Quero evitar gerenciar infraestrutura?  
> • Devo me preocupar com custo vs performance vs operação?  

Fluxograma sugerido:

```
Precisa de Kubernetes? → Sim → EKS  
              ↓ Não  
Quer evitar gerenciar infraestrutura? → Sim → Fargate / App Runner / Lambda  
              ↓ Não  
Preferência por controle profundo? → Sim → EC2 + Docker  
              ↓ Não  
Projectos pequenos ou MVP? → Lightsail ou ECS  
```

---

## Considerações Finais

- **Não existe “melhor” universal**: depende muito do contexto (tamanho da equipe, grau de maturidade, demanda de tráfego, SLAs).  
- Começar pequeno pode ser uma boa: testar, monitorar, iterar.  
- Automatizar (IaC), monitorar, versionar infraestrutura faz diferença.  
- Segurança, observabilidade e custo devem andar junto desde o início.

---

## Anotações Avançadas / Experiência Empírica

- Com Java ou Go, cuidado especial com *startup time* do container – se for Kubernetes ou Lambda, isso impacta.  
- Se usar microserviços, padronizar a imagem Docker ajuda (reduzir camadas, otimizar caching, imagem base leve).  
- Custo “oculto” de Kubernetes: overhead de CPU/RAM para controle, rede, logs, monitoramento.  
- Casos onde EC2 + Docker ainda fazem sentido: dependências específicas de hardware, tuning fino, ambientes isolados, migração gradual.

---

## Referências

- Deploying Containers on AWS: The One Guide You Need — Hasan Ashab ([dev.to](https://dev.to/hasan_ashab/deploying-containers-on-aws-the-one-guide-you-need-18mc))  
- Documentação oficial AWS: EKS, ECS, Fargate, App Runner, Lightsail  

---

## Tags / Palavras-chave

`aws`, `containers`, `kubernetes`, `serverless`, `ecs`, `eks`, `docker`, `infraestrutura`
