# Hash de Senhas no Backend: Guia e Boas Práticas

Este documento resume boas práticas para armazenar senhas de forma segura em aplicações backend, com exemplos em **Java** e **Go**, baseado no artigo *“Hash Passwords in Backend: A Guide for Every Tech Stack”*. ([dev.to](https://dev.to/mahmud-r-farhan/hash-passwords-in-backend-a-guide-for-every-tech-stack-2icc))

---

## Por que usar hash de senhas?

- Armazenar senhas em texto puro (plain text) é um risco grave de segurança.
- Hash transforma a senha em uma representação fixa, não reversível.
- Ao fazer login, a senha fornecida é processada da mesma forma e comparada com o hash armazenado.

---

## Principais algoritmos usados

| Nome      | Vantagens                                             | Cuidados / Desvantagens                         |
|------------|--------------------------------------------------------|---------------------------------------------------|
| **bcrypt** | Bom equilíbrio de segurança, tempo de hashing ajustável | Pode ser lento em hardware limitado               |
| **Argon2** | Muito resistente, moderno, uso recomendado em novos projetos | Menos difundido que bcrypt; cuidado na configuração |
| **PBKDF2** | Muito usado em ambientes corporativos                  | Dependendo da configuração, pode ser menos seguro que os dois anteriores |
| SHA‑256 / SHA‑512 | Simples e rápido                                | Sem salt & stretching: vulnerável a ataques de força bruta e rainbow tables |

---

## Exemplo em **Java** (Spring Boot usando bcrypt)

```java
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import javax.persistence.*;

@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;

    public void setPassword(String password) {
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
        this.password = encoder.encode(password);
    }

    public boolean checkPassword(String inputPassword) {
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
        return encoder.matches(inputPassword, this.password);
    }

    // getters e setters...
}
```

Dependência Maven necessária:

```xml
<dependency>
  <groupId>org.springframework.security</groupId>
  <artifactId>spring-security-crypto</artifactId>
  <version>5.x.x</version> <!-- usar versão compatível -->
</dependency>
```

---

## Exemplo em **Go** (usando bcrypt)

```go
package auth

import (
    "golang.org/x/crypto/bcrypt"
    "errors"
)

// HashPassword recebe uma senha em texto puro e retorna seu hash
func HashPassword(password string) (string, error) {
    // O custo (cost) define quantas iterações; o valor padrão ou 12‑14 é uma escolha razoável
    hashBytes, err := bcrypt.GenerateFromPassword([]byte(password), 14)
    if err != nil {
        return "", err
    }
    return string(hashBytes), nil
}

// CheckPassword compara senha em texto com hash armazenado
func CheckPassword(hashedPassword string, password string) bool {
    err := bcrypt.CompareHashAndPassword([]byte(hashedPassword), []byte(password))
    return err == nil
}
```

---

## Boas práticas

- Sempre use **salt** (sal) — bcrypt, Argon2 e PBKDF2 já lidam com salt internamente na maioria das implementações.
- Ajuste os parâmetros de custo de forma que dificulte ataques de força bruta, mas sem comprometer desempenho aceitável.
- Nunca armazene ou transmita senhas em texto puro (mesmo em logs ou em comunicações inseguras).
- Use algoritmos recentes e testados; evite SHA sem salt ou sem stretching.
- Se possível, use mecanismos de hashing recomendados pela comunidade e bibliotecas confiáveis.

---

## Considerações finais

Este guia fornece exemplos práticos para Java e Go, mas os princípios se aplicam a qualquer plataforma. O importante é:
- usar hashing seguro (bcrypt / Argon2 / PBKDF2),
- usar salt e iterar o hash,
- proteger os dados em trânsito e em repouso,
- nunca logar senhas em claro.

---

## Referências

- *Hash Passwords in Backend: A Guide for Every Tech Stack* – Mahmud R. Farhan ([dev.to](https://dev.to/mahmud-r-farhan/hash-passwords-in-backend-a-guide-for-every-tech-stack-2icc))
