# Versionamento de API (REST API Versioning)

_Knowledge Base Aggregation_
1. "The Complete Guide to API Versioning" ([dev.to](https://dev.to/muhabbat_dev/the-complete-guide-to-api-versioning-for-developers-9l1))
2. "Deep Dive: Strategies, Nightmares & Best Practices" ([dev.to](https://dev.to/mrgio7/backend-rest-api-versioning-a-deep-dive-into-strategies-nightmares-and-best-practices-22f0))

---

## O Dilema: Quando Versionar?

O versionamento é um "mal necessário". O objetivo deve ser sempre **evitar breaking changes**.

* **NÃO Versione:** Para mudanças aditivas (ex: novos campos no JSON, novos endpoints). Clientes bem implementados devem ignorar campos desconhecidos.
* **Versione:** Apenas para **Breaking Changes** (mudança de tipo de dado, remoção de campos, mudança de regra de negócio crítica).

---

## Estratégias de Versionamento (Decision Matrix)

Comparativo técnico para decisão arquitetural:

| Estratégia | Padrão URI | Exemplo | Prós | Contras | Recomendação Senior |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **URI Path** | `/v1/users` | Explícito na rota | Cacheável, fácil de testar/debugar | "Polui" a URL | **Alta** (Padrão de Mercado) |
| **Query Param** | `?v=1` | Parâmetro GET | Implementação rápida | Alguns proxies ignoram params no cache | **Baixa** |
| **Header** | `X-API-Ver` | Custom Header | URL limpa | Difícil testar via browser/curl simples | **Média** (APIs Internas) |
| **Media Type** | `Accept` | Content Negotiation | Purismo REST (HATEOAS) | Alta complexidade de doc e testes | **Baixa** (Overengineering comum) |

---

## Os "Pesadelos" Ocultos (Hidden Costs)

A experiência dita que o custo não está em *criar* a versão, mas em *mantê-la*. Ao decidir versionar, esteja ciente de:

1.  **Dependency Hell:** Manter lógica duplicada (Controllers, DTOs, Converters) para v1 e v2 aumenta a superfície de bugs exponencialmente.
2.  **Documentation Drift:** Manter o Swagger/OpenAPI atualizado para versões legadas é trabalhoso e frequentemente negligenciado.
3.  **Data Transformation:** Se o banco mudou para atender a v2, você terá custo de CPU para converter dados "on-the-fly" para entregar no formato da v1.

---

## Engenharia & Boas Práticas

### 1. Ciclo de Vida e Sunset (RFC 8594)
Não basta avisar por email. Use os cabeçalhos padrão da IETF para comunicar aos clientes que a API vai morrer.
* `Deprecation: @1735689600` (Data em Unix Epoch)
* `Sunset: <HTTP-Date>` (Data final de desligamento)
* `Link: <https://api.xyz/migration>; rel="deprecation"; type="text/html"`

### 2. SemVer (Semantic Versioning)
Adote `MAJOR.MINOR.PATCH` conceitualmente:
* **MAJOR (v1 -> v2):** Breaking changes.
* **MINOR:** Funcionalidade nova, retrocompatível (geralmente transparente na API REST).
* **PATCH:** Bug fixes (transparentes).

### 3. Organização de Código (Code Layout)
Evite espalhar `if (version == 1)` pelo código. Isole as camadas: