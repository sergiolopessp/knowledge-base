# Padrão Token-based Security (JWT/OAuth2)

Estratégia de autenticação e autorização que usa tokens (ex.: JWT, OAuth2) para verificar identidade e permissões, ideal para sistemas distribuídos.

## Sumário

- [Padrão Token-based Security](#padrão-token-based-security)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot e JWT](#exemplo-com-spring-boot-e-jwt)

## Componentes

- **Token**: String codificada (ex.: JWT com header, payload, signature).
- **Emissor**: Gera tokens após autenticação (ex.: servidor OAuth2, Auth Server).
- **Cliente**: Envia token em requisições (ex.: Authorization header).
- **Validador**: Verifica validade e permissões do token.

## Benefícios

- Escalabilidade: Tokens são stateless, sem necessidade de sessões no servidor.
- Segurança: Assinatura (JWT) ou validação centralizada (OAuth2).
- Interoperabilidade: Funciona em APIs, microservices e apps móveis.
- Flexibilidade: Suporta controle de acesso granular.

## Contras

- Complexidade: Configuração de emissão e validação.
- Revogação: Difícil revogar tokens individuais (JWT).
- Tamanho: Tokens podem aumentar o overhead de requisições.
- Segurança: Exige proteção contra roubo de tokens.

## Quando Usar

Use em APIs, microservices ou aplicações distribuídas que requerem autenticação segura e escalável (ex.: apps web, mobile, IoT).

## Exemplo com Spring Boot e JWT

### Configuração de Segurança

```java
package com.example.demo;

import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/login").permitAll()
                .anyRequest().authenticated()
            )
            .addFilterBefore(new JwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
        return http.build();
    }
}
```

### Filtro JWT

```java
package com.example.demo;

import io.jsonwebtoken.Jwts;
import jakarta.servlet.FilterChain;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.web.filter.OncePerRequestFilter;

public class JwtAuthenticationFilter extends OncePerRequestFilter {
    private final String SECRET_KEY = "my-secret-key";

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) {
        String header = request.getHeader("Authorization");
        if (header != null && header.startsWith("Bearer ")) {
            String token = header.replace("Bearer ", "");
            try {
                Jwts.parser().setSigningKey(SECRET_KEY).build().parseClaimsJws(token);
                SecurityContextHolder.getContext().setAuthentication(new JwtAuthentication(token));
            } catch (Exception e) {
                SecurityContextHolder.clearContext();
            }
        }
        chain.doFilter(request, response);
    }
}
```

### Geração de Token

```java
package com.example.demo;

import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class AuthController {
    private final String SECRET_KEY = "my-secret-key";

    @PostMapping("/login")
    public String login(@RequestBody LoginRequest request) {
        // Simula autenticação
        if ("user".equals(request.username()) && "password".equals(request.password())) {
            return Jwts.builder()
                .setSubject(request.username())
                .signWith(SignatureAlgorithm.HS512, SECRET_KEY)
                .compact();
        }
        return "Erro: Credenciais inválidas";
    }
}

record LoginRequest(String username, String password) {}
```