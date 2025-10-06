# AWS EKS: Guia de Boas Práticas (Mindmap Synthesis)

**Contexto:** Este documento sintetiza o Mindmap de Boas Práticas do AWS EKS, essencial para garantir um *cluster* **Kubernetes** (K8s) seguro, escalável e otimizado em termos de custo, alinhado com os princípios de **Cloud-Native** e **DevOps**.

---

## 1. Segurança do Cluster e da Rede (The "S" in EKS)

A segurança deve ser o pilar, especialmente considerando que rodamos aplicações críticas em **Java**, **Go** ou **C++** neste ambiente:

| Área | Melhor Prática | Por que é importante? (Visão Sênior) |
| :--- | :--- | :--- |
| **Identidade e Acesso** | Usar **IAM Roles for Service Accounts (IRSA)**. | Permite que *Pods* acessem recursos da AWS (S3, DynamoDB, etc.) com o **Princípio do Menor Privilégio**, eliminando a necessidade de armazenar credenciais de acesso em *Secrets*. |
| **Rede (Networking)** | Aplicar **Network Policies** (via Calico ou Cilium). | Restringe o tráfego *Leste-Oeste* (entre *Pods*). Essencial para segmentação e contenção de falhas de segurança. |
| **Segurança do Plano de Controle**| Usar *Private Endpoint* para o *Control Plane* do EKS. | Evita a exposição do servidor API do Kubernetes à internet pública. |
| **Segurança do Pod** | Utilizar **Pod Security Standards (PSS)** ou **Gatekeeper** (OPA). | Garante que os containers não rodem como `root` e tenham capacidades restritas. Previne escalação de privilégios. |

---

## 2. Gerenciamento e Tipos de Workers (Escalabilidade e Docker)

O gerenciamento eficiente dos nós (EC2) onde seus **Containers Docker** rodam é crucial para custo e resiliência.

* **Cluster Autoscaler:** Ferramenta padrão para escalar e reduzir nós automaticamente. **Crucial para a otimização de custos em Cloud.**
* **Karpenter:** Uma alternativa moderna ao Cluster Autoscaler, mais rápida e que otimiza a seleção do tipo de instância (EC2) para os *Pods* pendentes.
* **Tipos de Nós:**
    * **Managed Node Groups:** Recomendado para a maioria dos casos. AWS gerencia o *lifecycle* dos nós.
    * **Fargate:** Utilizar para *workloads* sem estado (ex: APIs em **Go** ou **Java**). AWS gerencia *totalmente* o nó; paga-se apenas pelo Pod. Simplifica drasticamente a operação.

---

## 3. Custo, Log e Observabilidade

Em uma arquitetura de Microsserviços rodando em K8s, é vital saber quem está gastando o quê e o que está acontecendo:

* **Cost Monitoring (FinOps):** Usar `Labels` (etiquetas) em todos os recursos (Namespaces, Deployments) para rastrear o custo via AWS Cost Explorer.
* **Logging:** Usar **Fluent Bit** para coletar logs dos containers e enviá-los de forma centralizada (ex: CloudWatch, Elasticsearch).
* **Monitoring (Métricas):** Implementar **Prometheus** e **Grafana** para métricas operacionais e de aplicação.
    * *Exemplo Java/Go:* Expor métricas via **Micrometer** (Java) ou pacotes nativos (Go) para o Prometheus.

---

## 4. Gerenciamento de Aplicações (Deployment e CI/CD)

* **GitOps (Padrão Chave):** Utilizar ferramentas como **ArgoCD** ou **Flux** para gerenciar o estado do cluster. O *estado desejado* do cluster é definido em um repositório Git, e a ferramenta o aplica automaticamente.
    * *Benefício:* Facilita o Rollback e o histórico de configurações.
* **Helm:** Utilizar como gerenciador de pacotes para simplificar o deploy de aplicações complexas.
* **Atualizações de K8s:** Seguir um plano de atualização de *Control Plane* e *Node Groups* de forma controlada, garantindo a compatibilidade de **APIs** (importante para evitar interrupções nos microsserviços).

---

## Conexão com o Conhecimento Prévio

* **Docker:** O EKS orquestra seus containers Docker. O *Mindmap* ajuda a garantir que esses containers sejam executados com segurança (PSS) e escalabilidade (Autoscaler/Karpenter).
* **Arquitetura (Monolito vs. Microsserviços):** O EKS é a plataforma ideal para rodar **Microsserviços**, permitindo que serviços em **Go** e **Java** convivam e escalem de forma independente (o que o Monolito não permitiria).