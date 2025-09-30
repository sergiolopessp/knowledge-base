# 🐳 Quick Reference: Melhores Práticas de Rede no Docker

## 🎯 O Essencial para Produção e Microservices

A rede é o tecido conjuntivo de qualquer arquitetura baseada em containers. Falhas ou má configurações aqui resultam em latência, problemas de segurança e indisponibilidade de serviços. Priorize sempre a **segurança por design** e a **comunicação resiliente**.

---

## 🔒 1. Segmentação e Isolamento (Princípio Zero-Trust)

Nunca confie na rede padrão (`bridge` default) ou no host. O isolamento deve ser o estado padrão, especialmente para serviços sensíveis.

| Prática | Objetivo | Insight do Engenheiro Sênior |
| :--- | :--- | :--- |
| **Criar Redes Dedicadas** | Utilize `docker network create` para cada aplicação ou camada de serviço (e.g., `app-frontend`, `app-backend`, `db-internal`). | Isso impede o tráfego não autorizado entre projetos e facilita a auditoria. Evite o `bridge` default, que é um risco de segurança. |
| **Isolamento via `iptables`** | Configure regras de firewall (via Host ou Engine) para restringir o tráfego *inter-container* e *host-to-container*. | Mesmo com redes dedicadas, um bom controle de `iptables` é a última linha de defesa para evitar exploração em caso de comprometimento de um container. |
| **Desativar Publicação Desnecessária** | Evite usar `-p` ou `ports` no Docker Compose se o serviço não precisa de acesso externo. | O ideal é que apenas um *proxy reverso* (ex: NGINX, Traefik) na rede `frontend` exponha portas. O tráfego interno deve ser resolvido via DNS. |

---

## 🔎 2. Descoberta de Serviços (Service Discovery)

Aplicações *cloud-native* (escritas em Java ou Go) precisam se comunicar usando nomes lógicos, não endereços IP.

### **Priorize o DNS Interno do Docker**

* **Regra:** Sempre conecte os containers na mesma rede definida (`--network <nome_da_rede>`).
* **Vantagem (Java/Go):** Os *runtimes* (JVM ou Go Runtime) podem resolver nomes de serviços (ex: `http://servico-b/api`) automaticamente, de forma confiável e sem a necessidade de um servidor DNS externo adicional (como Consul ou ZooKeeper) para a comunicação *simples* dentro do host.
* **Comando de Inspeção:** Use `docker network inspect <nome_da_rede>` para verificar os aliases e IPs.

## 🤝 3. Gerenciamento de Subnets (Evite Conflitos)

A sobreposição de subnets é uma fonte comum de erros de roteamento e interrupção de conexões.

* **Verificação:** Antes de criar uma rede, inspecione as redes existentes (incluindo as redes do Host e VPNs) para garantir que os blocos **CIDR** não se sobreponham.
    ```bash
    docker network ls
    docker network inspect <nome_da_rede>
    ```
* **Criação Explícita:** Ao criar uma rede dedicada, especifique o bloco de sub-rede para ter controle total:
    ```bash
    docker network create \
      --driver bridge \
      --subnet 172.20.0.0/16 \
      --gateway 172.20.0.1 \
      minha_rede_segura
    ```

## 🔒 4. Comunicação Segura e Multi-Host

Para ambientes de produção que abrangem múltiplas máquinas virtuais (VMs) na nuvem, a comunicação deve ser criptografada.

* **Overlay Networks (Docker Swarm/Kubernetes):** Utilize redes **Overlay** (ex: Flannel, Weave Net no K8s, ou o driver Swarm nativo).
* **Criptografia:** O driver `overlay` no Docker Swarm e muitas soluções de orquestração (via VXLAN/IPSec) fornecem criptografia de *inter-host* **out-of-the-box**.
* **Mandatório para Cloud:** Em uma infraestrutura de *Cloud Computing* (AWS, GCP, Azure), onde o tráfego entre VMs atravessa a rede pública do provedor, a utilização de uma **rede *overlay* criptografada** é uma prática de **segurança obrigatória** para atender a requisitos regulatórios e proteger dados em trânsito.