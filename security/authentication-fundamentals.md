# Fundamentos de Autenticação para Desenvolvedores

Este documento estabelece os conceitos básicos de autenticação (AuthN), diferenciando-os de autorização (AuthZ) e detalhando as melhores práticas para proteção de identidade e dados.

## 1. AuthN vs AuthZ
É fundamental não confundir os dois processos, especialmente em arquiteturas distribuídas:
* **Autenticação (AuthN):** Valida a identidade. Responde à pergunta: *"Quem é você?"*
* **Autorização (AuthZ):** Valida permissões. Responde à pergunta: *"O que você pode fazer?"*

> **Nota:** A autenticação deve sempre preceder a autorização.

## 2. Dados em Trânsito e em Repouso

### Dados em Trânsito (In Motion)
A proteção é feita via **TLS (Transport Layer Security)**. No contexto de sistemas modernos, o uso de HTTPS é mandatório para garantir a confidencialidade e integridade, evitando ataques de interceptação (Man-in-the-Middle).

### Dados em Repouso (At Rest)
Nunca armazene senhas em texto claro. A proteção deve ser feita através de **Hashing Unidirecional**.

| Característica | Criptografia | Hashing |
| :--- | :--- | :--- |
| **Natureza** | Bidirecional (Reversível) | Unidirecional (Irreversível) |
| **Objetivo** | Proteger dados que precisam ser lidos | Validar integridade e segredos |
| **Exemplo** | HTTPS, Armazenamento de PII | Armazenamento de senhas |

#### Salting e Algoritmos Recomendados
Para mitigar ataques de *Rainbow Tables*, deve-se utilizar um **Salt** (valor aleatório único por usuário).
* **Algoritmos Seguros:** Argon2 (preferencial), bcrypt ou PBKDF2.
* **Evite:** MD5, SHA-1 ou funções de hash rápidas sem salt.

## 3. Fatores de Autenticação (MFA)
A segurança é ampliada quando combinamos diferentes categorias de fatores:

1.  **Conhecimento (Algo que sabe):** Senhas, PINs.
2.  **Posse (Algo que tem):** Soft Tokens (TOTP), Hard Tokens, Push notifications.
3.  **Inerência (Algo que é):** Biometria (Impressão digital, FaceID).

## 4. Ecossistema de Autenticação (PaaS)
A escolha de um serviço de autenticação deve levar em conta o nível de controle desejado vs. velocidade de implementação:

* **Managed (SaaS):** Auth0, Clerk, Firebase Auth (Rápido, mas com maior *vendor lock-in*).
* **Infrastructure-focused:** AWS Cognito, Azure AD B2C.
* **Open Source / Self-Hosted:** Supabase Auth, Better Auth, Keycloak.

## 5. Tendências e Evolução (Passwordless)
O futuro da autenticação caminha para a eliminação de senhas através de protocolos como **FIDO 2.0** e **WebAuthn**, utilizando chaves criptográficas (Passkeys) em vez de segredos compartilhados. Isso resolve o problema de phishing e a fadiga de senhas dos utilizadores.

---
**Referências:**
* [Back to Basics: A Developer's Guide to Authentication](https://dev.to/prcsmae/back-to-basics-a-developers-guide-to-authentication-5cce)