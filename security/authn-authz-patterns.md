# Authentication & Authorization em Backend Systems

**Tags:** #backend #security #architecture #golang #jwt #auth
**Date:** 2025-11-28
**Context:** Baseado no artigo [Authentication & Authorization in Backend Development](https://dev.to/riteshkokam/authentication-authorization-in-backend-development-4go) e experiências em sistemas distribuídos.

---

## 1. O Conceito Fundamental

Embora frequentemente usados de forma intercambiável, AuthN (Autenticação) e AuthZ (Autorização) resolvem problemas distintos na segurança de aplicações.

| Característica | Autenticação (AuthN) | Autorização (AuthZ) |
| :--- | :--- | :--- |
| **Pergunta Chave** | "Quem é você?" | "O que você pode fazer?" |
| **Objetivo** | Verificar identidade. | Verificar permissões de acesso. |
| **Dados** | Credenciais (Senha, Biometria, OTP). | Políticas, Roles, Claims. |
| **Ordem** | Ocorre primeiro. | Ocorre após a autenticação. |
| **Exemplo** | Login com usuário/senha ou OAuth. | Acesso de leitura vs. escrita em uma API. |

> **Notas:** Em sistemas distribuídos, tente desacoplar esses dois. Um *Identity Provider* (IdP) como Keycloak ou Auth0 resolve a AuthN, enquanto seu microsserviço deve focar apenas na AuthZ (validar se o token recebido tem o escopo necessário).

---

## 2. Estratégias de Implementação

### 2.1. Session-Based (Stateful)
O método tradicional. O servidor cria uma sessão e armazena o estado (geralmente em memória ou banco de dados) e envia um `Session-ID` via Cookie para o cliente.

* **Pros:** Fácil revogação (basta deletar a sessão do servidor).
* **Cons:** Difícil de escalar horizontalmente. Em um ambiente de containers (Docker/K8s), requer um store externo (ex: Redis) para compartilhar sessão entre réplicas, adicionando latência e complexidade.

### 2.2. Token-Based / JWT (Stateless)
O padrão moderno para REST e Microsserviços. O servidor gera um token assinado (JWT - JSON Web Token) contendo os dados do usuário (*payload*) e o envia ao cliente. O cliente envia esse token no header `Authorization: Bearer <token>` em cada requisição.



* **Pros:**
    * **Stateless:** O servidor não guarda nada. Perfeito para arquiteturas Cloud-Native e Auto-scaling.
    * **Desempenho:** Validação é apenas cálculo de assinatura (CPU), sem I/O de banco de dados.
    * **Interoperabilidade:** JSON é universal (Java, Go, Node, C++).
* **Cons:** Revogação complexa (requer *blocklists* ou tempos de expiração curtos).

---

## 3. Fluxo em Go (Exemplo Conceitual)

Dado que a performance do Go é ideal para *Auth Services*, um middleware típico de validação de JWT teria esta lógica:

```go
// Exemplo simplificado de Middleware em Go
func AuthMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // 1. Extrair o token do Header "Authorization"
        tokenString := extractToken(r)

        // 2. Validar assinatura (CPU intensive, mas rápido em Go)
        token, err := validateToken(tokenString)

        if err != nil || !token.Valid {
            http.Error(w, "Forbidden", http.StatusForbidden)
            return
        }

        // 3. Extrair Claims (AuthZ) e passar para o contexto
        ctx := context.WithValue(r.Context(), "userRole", token.Claims["role"])
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}
```

## 4. Melhores Práticas 
Nunca armazene senhas em texto plano: Use algoritmos de hash lentos e seguros. Em Java/Go, prefira Argon2 ou Bcrypt. Evite MD5 ou SHA-1.

HTTPS é obrigatório: Sem TLS, qualquer Basic Auth ou Token é trafegado em texto plano e pode ser interceptado (Man-in-the-Middle).

Princípio do Menor Privilégio: Na AuthZ, dê ao usuário apenas as permissões estritamente necessárias para a tarefa.

Cuidado com o Payload do JWT: O JWT é apenas codificado em Base64, não criptografado. Nunca coloque dados sensíveis (CPF, senhas, saldos) no payload, pois qualquer um pode ler.

Gerenciamento de Segredos: Em ambientes Docker/Kubernetes, injete as chaves de assinatura de tokens (Secret Keys) via Secrets ou Vault, nunca hardcoded no código.

## 5. Fundamentos de Tokens e Segurança de Armazenamento

Para implementar padrões de Autenticação e Autorização de forma segura, é crucial distinguir o propósito de cada token e como protegê-los no client-side.

### 5.1 Anatomia dos Tokens (ID vs Access vs Refresh)

| Token | Propósito | Destinatário | Tempo de Vida |
| :--- | :--- | :--- | :--- |
| **ID Token** | Identidade. Diz "quem" o usuário é (claims: nome, email). | Frontend | Curto |
| **Access Token** | Autorização. Diz "o que" o usuário pode acessar na API. | API Resource | Curto (5-15 min) |
| **Refresh Token** | Renovação. Usado para obter novos Access Tokens sem re-login. | IdP (Auth Server) | Longo (dias/mesas) |

### 5.2 Onde Armazenar? (A Armadilha do LocalStorage)

O armazenamento de tokens no navegador é um dos pontos mais críticos em arquiteturas modernas:

* **LocalStorage / SessionStorage**: ❌ **Não utilizar para tokens sensíveis.** São vulneráveis a ataques **XSS (Cross-Site Scripting)**. Qualquer script de terceiros injetado na página pode ler esses tokens e exfiltrá-los.
* **Memória (Variáveis JS)**: ✅ Recomendado para **Access e ID Tokens**. Embora o token seja perdido ao atualizar a página (refresh), isso limita a janela de exposição. O estado pode ser recuperado silenciosamente via Refresh Token.
* **HttpOnly, Secure Cookies**: ✅ Mandatório para **Refresh Tokens**. 
    * `HttpOnly`: Impede que o JavaScript acesse o cookie (protege contra XSS).
    * `Secure`: Garante que o cookie só trafegue via HTTPS.
    * `SameSite=Strict/Lax`: Protege contra ataques CSRF.

### 5.3 Estratégia de Renovação (Silent Refresh)

Para manter uma boa UX sem comprometer a segurança:
1. O **Access Token** expira rapidamente.
2. O Frontend detecta o erro 401 ou monitora o `exp` do token.
3. Uma chamada de background é feita para o endpoint de `/refresh`, enviando o **Refresh Token** (armazenado no Cookie seguro).
4. O servidor de autenticação valida o cookie e entrega um novo Access Token em memória para o frontend.

> **Insight de Arquitetura Cloud Native**: Para aplicações SPA críticas, considere o padrão **BFF (Backend for Frontend)**. Nele, os tokens nunca chegam ao navegador, o BFF gerencia a sessão via cookies criptografados e atua como um proxy seguro para as APIs de backend (Java/Go/Spring), simplificando a segurança no frontend.


## 6. Referências

[RFC 7519 - JSON Web Token (JWT)](https://datatracker.ietf.org/doc/html/rfc7519)

[OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/)