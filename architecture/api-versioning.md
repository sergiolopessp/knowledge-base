# Versionamento de API (API Versioning)

_Resumo inspirado no artigo “The Complete Guide to API Versioning for Developers”_ ([dev.to](https://dev.to/muhabbat_dev/the-complete-guide-to-api-versioning-for-developers-9l1))

---

## Motivações para versionar APIs

- Sistemas evoluem: novas funcionalidades, refatorações, correções.  
- Manter compatibilidade com clientes antigos impede que “quebrem” com mudanças.  
- Mudanças “não‑quebrantes” (adicionar campos opcionais, novos endpoints) podem coexistir.  
- Mudanças “quebrantes” (mudar tipo, renomear campo, remover campos, mudar autenticação) exigem versionamento explícito.  

---

## Estratégias de versionamento

| Estratégia | Exemplos / como fica na API | Vantagens | Desvantagens |
|------------|-------------------------------|------------|----------------|
| **URI Path Versioning** | `GET /v1/users/123` / `GET /v2/users/123` | explícito, fácil de testar, visível | polui URI, pode “misturar versão com recurso” |
| **Query‑parameter Versioning** | `GET /users/123?version=1` | simples de implementar | menos “limpo”, risco de esquecer o parâmetro |
| **Header Versioning (custom header)** | `GET /users/123` com header `X-API-Version: 2` | URI permanece limpa | menos visível, exige clientes bem configurados |
| **Media Type Versioning / Content Negotiation** | `Accept: application/vnd.example.v1+json` | mais “RESTful”, aproveita mecanismos HTTP | sintaxe complexa, menos intuitivo para usuários |

---

## Boas práticas + recomendações

1. **Versão padrão (default)**  
   Se cliente não especificar versão, defina um fallback (por exemplo, v1 estável). Evita que clientes sem versão explícita sejam “quebrados” ao lançar nova versão.

2. **Política de depreciação (sunsetting)**  
   Você não pode manter versões antigas indefinidamente. Documente prazo e comunique aos usuários.  
   Ex: “v1 será descontinuada em 6 meses; à partir de 12 meses será desligada”.

3. **Comunicação clara**  
   Use changelogs públicos, guias de migração, anúncios por e‑mail ou portal de API.

4. **Organização de código / modularidade**  
   Organize módulos ou controllers por versão. Por exemplo:

   ```
   /api/
     /v1/
       UserController, ProductController
     /v2/
       UserController, ProductController
   ```

   Também pode haver lógica comum em módulos compartilhados, e apenas exceções específicas nas versões.

5. **Nunca misture mudanças quebrantes e não‑quebrantes no mesmo lançamento**  
   Mantenha releases coerentes e limpos.

6. **Documentação de API por versão**  
   Cada versão precisa de sua documentação (Swagger / OpenAPI / etc).

---

## Exemplo conceitual (em Java / Go)

```java
// Exemplo em Java com Spring Boot (versão via URI path)
@RestController
@RequestMapping("/api/v1/users")
public class UserControllerV1 {
    @GetMapping("/{id}")
    public UserV1 getUser(@PathVariable Long id) {
        // retorno no formato da versão 1
    }
}

@RestController
@RequestMapping("/api/v2/users")
public class UserControllerV2 {
    @GetMapping("/{id}")
    public UserV2 getUser(@PathVariable Long id) {
        // retorno no formato da versão 2
    }
}
```

```go
// Exemplo simplificado em Go (mux router)
router.HandleFunc("/api/v1/users/{id}", handlerV1.GetUser)
router.HandleFunc("/api/v2/users/{id}", handlerV2.GetUser)
```

Você pode extrair lógica comum para um pacote interno e cada versão faz adaptações.

---

## Considerações finais / orientação para escolha

- Para APIs **públicas**, **URI Path Versioning** costuma ser preferível: explícito, fácil de usar e documentar.  
- Para APIs **internas** (cliente e servidor sob controle), **header versioning** pode ajudar a manter URIs limpas.  
- O mais importante é: **escolher uma estratégia cedo**, documentar, aplicar consistentemente e evoluir com disciplina.

---

## Recursos e leitura adicional

- Artigo original: *The Complete Guide to API Versioning for Developers* ([dev.to](https://dev.to/muhabbat_dev/the-complete-guide-to-api-versioning-for-developers-9l1))  
- Boas práticas de versionamento em OpenAPI / Swagger  
- Exemplos de versionamento em frameworks Java / Spring, Go, Node.js  

---

_(Este documento pode evoluir conforme você aplica em projetos e coleta lições aprendidas. Sinta-se à vontade para converter exemplos em C++ se necessário.)_  
