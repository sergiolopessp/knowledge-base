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


## 5. Referências

[RFC 7519 - JSON Web Token (JWT)](https://datatracker.ietf.org/doc/html/rfc7519)

[OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/)