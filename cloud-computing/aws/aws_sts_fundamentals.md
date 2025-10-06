# AWS Security Token Service (STS) - Conceitos e Uso

**Contexto:** O **AWS STS** é o serviço web que permite solicitar credenciais de segurança temporárias e com privilégios limitados para usuários ou serviços da AWS. Ele é o pilar da segurança no acesso **Cross-Account** e na federação de identidade, sendo crucial para o **Princípio do Menor Privilégio** em ambientes de produção.

---

## 1. O Conceito Central: Credenciais Temporárias

O STS não armazena identidades (isso é trabalho do IAM). Sua função é **emitir credenciais temporárias** (Token de Segurança) que consistem em:

1.  **Access Key ID**
2.  **Secret Access Key**
3.  **Session Token** (O componente que define a validade temporária)

Essas credenciais expiram (podem durar de 15 minutos a 12 horas) e não podem ser revogadas individualmente, forçando os desenvolvedores e aplicações a implementarem mecanismos de renovação e reduzindo drasticamente o risco de vazamento de credenciais de longa duração.

---

## 2. Principais APIs do STS e Casos de Uso

Existem três APIs principais que um engenheiro sênior deve dominar:

### 2.1. `AssumeRole` (O Mais Usado)

Permite que uma entidade (usuário IAM, serviço EC2, Lambda, EKS Pod via IRSA) **assuma** as permissões de uma **IAM Role** diferente.

* **Uso em Engenharia:** Fundamental para acesso **Cross-Account**. Por exemplo, um *microsserviço* (em Go ou Java) rodando na Conta A precisa fazer um *deployment* na Conta B (Produção). Ele assume a `Role` de *Deployment* da Conta B.
* **Mecanismo em EKS:** Usado implicitamente quando se configura **IRSA (IAM Roles for Service Accounts)**. O *Pod* assume a *Role* via `AssumeRoleWithWebIdentity`.

### 2.2. `GetSessionToken`

Usado para obter credenciais temporárias para o **seu próprio** usuário IAM.

* **Uso em Engenharia:** Geralmente usado por usuários federados ou quando um usuário IAM precisa de um token com MFA para ser usado temporariamente na CLI.

### 2.3. `AssumeRoleWithWebIdentity`

Permite que identidades externas (usuários Google, Facebook, um Identity Provider customizado, etc.) se autentiquem na AWS.

* **Uso em Engenharia:** Base para a federação de identidade. Por exemplo, um usuário autenticado em um IdP (como Auth0 ou Cognito) obtém um token, que é trocado por credenciais AWS temporárias via esta API.

---

## 3. Implicações de Segurança e DevOps

1.  **Menos Segredos no Código:** O uso do STS (especialmente `AssumeRole`) significa que não há necessidade de *hard-code* credenciais de longa duração em código **Java** ou **Go**, reduzindo o risco de vazamentos.
2.  **Transparência e Auditoria:** Cada sessão assumida via STS é registrada no **CloudTrail**, incluindo o *Caller ID* original e a *Role* assumida. Isso é essencial para trilhas de auditoria.
3.  **Escopo Limitado:** Ao usar o `AssumeRole`, você pode passar um parâmetro `Policy` para **reduzir ainda mais** os privilégios da Role que está sendo assumida, garantindo que o token temporário tenha o menor escopo de ação possível para aquela tarefa.

---

## 4. Integração com Linguagens

Em **Java** (SDK) ou **Go** (SDK v2), a maioria das bibliotecas e *clients* (como o SDK do S3) lida com a obtenção e renovação dessas credenciais temporárias automaticamente ao configurar o *client* com uma `Role` ou um *Profile* que tem permissão para assumir outra *Role*. O desenvolvedor raramente interage diretamente com a API do STS, mas deve entender seu mecanismo de funcionamento.