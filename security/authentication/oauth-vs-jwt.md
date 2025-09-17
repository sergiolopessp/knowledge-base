---
title: "OAuth vs JWT: Qual protege melhor suas APIs?"
date: 2025-09-17
tags:
  - segurança
  - autenticação
  - OAuth
  - JWT
---

## Resumo

Este documento compara **OAuth** e **JWT** no contexto de segurança de APIs, destacando o que cada um oferece, suas diferenças, pontos fortes e fracos, além de casos de uso recomendados.

---

## O que é OAuth?

- **OAuth (Open Authorization)** é um protocolo que permite que aplicações de terceiros acessem dados de usuários sem exigir que essas aplicações tenham as credenciais (usuário/senha) do usuário.  
- Ele funciona como delegação de acesso, onde você autoriza permissões específicas em nome do usuário.  
- Exemplos comuns: login usando Google, Facebook ou GitHub, onde você autoriza que um site ou app acesse apenas determinados escopos do seu perfil ou dados. ([devinitiative](https://deviniciative.wordpress.com/2023/05/30/entendendo-a-diferenca-entre-jwt-oauth-e-saml/))

---

## O que é JWT?

- **JWT (JSON Web Token)** é um formato de token compacto, auto contido (self-contained), que transporta informação entre partes.  
- Pode ser usado para autenticação, autorização e troca segura de informações.  
- Importante: embora qualquer um possa ler o conteúdo de um JWT (se estiverem em texto claro), apenas partes que possuem a chave de assinatura conseguem **verificar** se ele é autêntico. ([devinitiative](https://deviniciative.wordpress.com/2023/05/30/entendendo-a-diferenca-entre-jwt-oauth-e-saml/))

---

## Principais diferenças (OAuth vs JWT)

| Característica         | OAuth                                              | JWT                                                |
|------------------------|-----------------------------------------------------|-----------------------------------------------------|
| Propósito              | Autorização de acesso de terceiros                 | Autenticação + transporte de dados                 |
| Tipos de token         | Access Token, Refresh Token                       | Token auto contido (claims dentro do token)        |
| Estado / Sessão        | Pode requerer estado (validação no servidor de autorização) | Pode ser usado de forma stateless                 |
| Expiração / renovação  | Uso de tokens de refresh para renovar access token | Token expira; reemissão via alguma lógica externa   |
| Uso típico             | APIs de terceiros, delegação de permissão          | Single Sign-On, autenticação de usuário em apps internos |

---

## Vantagens e desvantagens comparativas

### Vantagens do OAuth

- Permite separar autorização de autenticação, mitigando riscos ao delegar acesso limitado.  
- Escalável para cenários em que múltiplas aplicações precisam acessar recursos protegidos.  
- Usa refresh tokens para manter acesso sem expor credenciais repetidamente.

### Desvantagens do OAuth

- Maior complexidade de implementação (fluxos diferentes: Authorization Code, Implicit, Client Credentials etc.).  
- Se não for configurado corretamente, refresh tokens ou scopes mal definidos podem expor vulnerabilidades.

### Vantagens do JWT

- Simplicidade no uso para autenticação stateless: verificado apenas localmente sem necessidade de consultar persistência para cada requisição (dependendo do modelo).  
- Bom desempenho em cenários de microserviços ou servidores sem estado.  
- Fácil transmissão entre domínios ou serviços (APIs distribuídas).

### Desvantagens do JWT

- Se o token for comprometido (ex: vazamento da chave privada ou token interceptado), pode haver uso indevido até sua expiração.  
- Revogação de JWTs pode ser trabalhosa: como são auto contidos, deve-se implementar mecanismos externos (listas de revogação, blacklist, short-lived tokens).  
- O tamanho do token pode crescer se muitos dados forem incluídos nos claims.

---

## Cenários de uso recomendados

- Use **OAuth + JWT** quando você precisa de autorização de terceiros e quer que seus tokens de acesso sejam auto contidos. Por exemplo: apps que permitem login via provedores externos, com escopos específicos, mas que depois usam JWT internamente para autenticação entre serviços.  
- Use apenas **JWT** quando o contexto for mais simples, por exemplo autenticação entre front-end e back-end dentro de um domínio controlado, onde os tokens podem ser mantidos curtos, expirados rapidamente, e o risco de revogação é aceitável.

---

## Dicas práticas de segurança

1. Use tokens de acesso com tempo de vida curto.  
2. Implemente refresh tokens com segurança e validação server-side.  
3. Garanta que a assinatura do JWT use chaves fortes e protegidas (assim como algoritmos modernos, por exemplo RS256, ES256 etc.).  
4. Se for necessário revogar tokens, planeje uma estratégia (lista de revogação, invalidar token via blacklist ou armazenar identificador do token).  
5. Proteja contra ataques de replay e cross-site (por exemplo usando HTTPS, assinaturas, validação de nonce etc.).  
6. Evite colocar informações sensíveis nos claims do JWT, especialmente dados que não deveriam ficar visíveis para qualquer um que possa capturar o token.

---

## Conclusão

OAuth e JWT não são mutuamente excludentes; muitas vezes eles são complementares. A escolha depende dos requisitos de segurança, da arquitetura do sistema e dos trade-offs que se está disposto a assumir (complexidade vs controle). Em muitos casos, uma combinação bem feita de OAuth para autorização e JWT para transporte/autenticação de tokens auto contidos pode oferecer o melhor dos dois mundos.

---

## Referências

- *Entendendo a Diferença entre JWT, OAuth e SAML* — DevInitiative ([devinitiative](https://deviniciative.wordpress.com/2023/05/30/entendendo-a-diferenca-entre-jwt-oauth-e-saml/))  
- Documentação oficial do OAuth 2.0  
- RFC 7519 — JSON Web Token  
