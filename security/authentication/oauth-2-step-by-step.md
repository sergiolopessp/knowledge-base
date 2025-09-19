# OAuth 2.0: passo a passo

Esse documento explica de forma clara e estruturada como funciona o **OAuth 2.0** (RFC 6749), seus componentes, fluxos de autorização, uso de tokens, vantagens e cuidados de segurança.

---

## Sumário

1. Cenários de uso  
2. Definições importantes (terminologia)  
3. Conceito central do OAuth 2.0  
4. Fluxo de operação (overview)  
5. Os 4 principais *grant types*  
   1. Authorization Code  
   2. Implicit  
   3. Resource Owner Password Credentials  
   4. Client Credentials  
6. Renovação de token (Refresh Token)  
7. Comparações úteis / quando usar o quê  
8. Questões de segurança, boas práticas  
9. Exemplos de requisições HTTP  

---

## 1. Cenários de uso

- Aplicações que desejam acessar recursos em nome de um usuário em outro serviço (“Cloud Photo Printing” acessando fotos do Google, por exemplo).  
- Evitar que aplicações terceiros tenham acesso direto à senha do usuário.  
- Permitir escopos, validade e possibilidade de revogação de autorização de forma granular.  

---

## 2. Definições importantes

| Termo | Descrição |
|---|---|
| Resource Owner | O usuário que possui os dados ou recursos protegidos. |
| Client | Aplicação de terceiros que quer acessar recursos em nome do usuário. |
| Authorization Server | Servidor que autentica o usuário / proprietário do recurso e emite tokens. |
| Resource Server | Servidor que mantém os recursos protegidos. Pode coincidir com o Authorization Server ou ser separado. |
| User Agent | Normalmente navegador ou cliente que intermedia interações do usuário. |
| Escopo (scope) | Define permissões que o client está solicitando. |
| Token de acesso (Access Token) | Token que permite acesso a recursos protegidos por um tempo. |
| Refresh Token | Token que permite obter novo access token sem pedir nova autorização do usuário. |

---

## 3. Conceito central do OAuth 2.0

OAuth 2.0 introduz uma camada de autorização separada da autenticação do usuário, permitindo que um cliente (aplicação) obtenha autorização limitada para agir em nome do usuário, sem expor credenciais sensíveis (ex: senha), com controle sobre escopo, validade, revogação, etc.

---

## 4. Fluxo de operação

Fluxo básico:

1. Cliente solicita autorização do usuário.  
2. Usuário consente / concede autorização.  
3. Cliente, usando autorização, pede um token ao Authorization Server.  
4. Authorization Server valida e emite token(s).  
5. Cliente usa token para acessar Resource Server.  
6. Resource Server valida token e responde com os recursos solicitados.  

---

## 5. Grant Types / Modos de autorização

### 5.1 Authorization Code

- O usuário é redirecionado para o Authorization Server.  
- O Authorization Server retorna *authorization code* após consentimento.  
- Cliente troca esse código por *access token* (e possivelmente *refresh token*) no backend.  
- Mais seguro, porque credential do usuário nunca exposta diretamente ao client.  

### 5.2 Implicit (Simplificado)

- Token de acesso obtido diretamente do Authorization Server via navegador, sem passar por backend do cliente.  
- Menos seguro: token visível no browser, risco de exposição.  

### 5.3 Resource Owner Password Credentials

- Cliente recebe diretamente usuário e senha do usuário.  
- Só usar quando o cliente é altamente confiável e quando não há outras opções.  
- Alto risco se mal implementado.  

### 5.4 Client Credentials

- Sem usuário envolvido: cliente age por si só.  
- Usado para serviços internos ou machine‑to‑machine.  

---

## 6. Renovação de token (Refresh Token)

- Quando *access token* expira, cliente utiliza *refresh token* para obter novo *access token*.  
- O *scope* do novo token não pode exceder o anterior.  
- Boas práticas: expiração razoável, revogação, armazenar de forma segura.  

---

## 7. Comparações úteis / Quando usar OAuth 2.0

- OAuth 2.0 vs JWT: JWT é um formato de token, pode ser utilizado dentro de OAuth como *access token*. O OAuth traz todo o protocolo de autorização, fluxos, etc. (ver também `oauth-vs-jwt.md`).  
- Quando não usar OAuth: casos bem simples, sem múltiplos serviços terceiros, sem necessidade de escopo ou revogação, etc.

---

## 8. Segurança e boas práticas

- Sempre usar HTTPS para transmissão de tokens.  
- Validar *redirect_uri* rigorosamente.  
- Usar *state* para evitar CSRF nos fluxos que redirecionam.  
- Manter os *grant types* com menor privilégio possível.  
- Tokens de curta duração. Uso cuidadoso de refresh tokens.  
- Armazenamento seguro de tokens no cliente / servidor.  

---

## 9. Exemplos de requisições HTTP

### Authorization Code

```http
GET /authorize?response_type=code&client_id=CLIENT_ID&redirect_uri=https%3A%2F%2Fclient.example.com%2Fcb&scope=somescope&state=xyz HTTP/1.1
Host: auth.example.com
```

Resposta de redirect:

```http
HTTP/1.1 302 Found
Location: https://client.example.com/cb?code=AUTH_CODE&state=xyz
```

Troca de código por token:

```http
POST /token HTTP/1.1
Host: auth.example.com
Content-Type: application/x-www-form-urlencoded
Authorization: Basic CLIENT_ID:CLIENT_SECRET

grant_type=authorization_code&code=AUTH_CODE&redirect_uri=https%3A%2F%2Fclient.example.com%2Fcb
```

Resposta:

```json
{
  "access_token": "ACCESS_TOKEN_VALUE",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "REFRESH_TOKEN_VALUE",
  "scope": "somescope"
}
```

---

## 10. Resumo / Takeaways

- OAuth 2.0 é sobre **autorização**, não autenticação (embora por vezes usado para ambas).  
- Escolher grant type certo é crítico para segurança.  
- Implementar fluxos seguros, escopos, validade e revogação.  
- JWT pode ser usado como formato de token dentro de OAuth, mas não substitui o protocolo.

---

## Referências

- RFC 6749 – The OAuth 2.0 Authorization Framework  
- Leapcell, *Mastering OAuth 2.0: Step by Step* (https://dev.to/leapcell/mastering-oauth-20-step-by-step-2726)  
