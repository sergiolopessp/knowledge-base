# Entendendo @Configuration no Spring Boot

**Tags:** #java #spring-boot #architecture #ioc #dependency-injection

## O que é a anotação `@Configuration`?

A anotação `@Configuration` é um estereótipo de nível de classe no Spring Framework que indica ao container IoC (Inversion of Control) que aquela classe é uma fonte de definições de Beans.

Historicamente, ela veio para substituir a antiga e verbosa configuração via XML (`applicationContext.xml`), permitindo uma abordagem **Java-Config** (configuração programática), que é mais segura em termos de tipos e mais fácil de refatorar.

Em resumo:
> Uma classe anotada com `@Configuration` é onde nós declaramos os componentes (`@Bean`) que serão gerenciados pelo `ApplicationContext`.

---

## 🛠 Como utilizar

A utilização básica envolve anotar a classe e definir métodos anotados com `@Bean`. O Spring escaneará essa classe na inicialização e registrará os retornos desses métodos como instâncias no container.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

    @Bean
    public Database database() {
        return new Database("localhost", 3306);
    }

    @Bean
    public UserService userService() {
        // Injeção de dependência manual chamando o método bean
        return new UserService(database());
    }
}
```
### Principais Casos de Uso

Definição de múltiplos Beans: Agrupar serviços, repositórios ou configurações relacionadas (ex: SecurityConfig, DatabaseConfig).
Bibliotecas de Terceiros: Configurar classes que não nos pertencem (como RestTemplate, ObjectMapper ou DataSource) e, portanto, não podem ser anotadas diretamente com @Component.
Controle Fino: Quando precisamos de lógica condicional ou inicialização complexa de um objeto.
### O "Pulo do Gato": Proxying CGLIB e Singleton

Aqui entra o conhecimento de "velha guarda". A grande diferença entre usar @Configuration e apenas @Component (ou "Lite Mode") está no comportamento de proxy.
Quando o Spring inicializa, ele usa a biblioteca CGLIB para criar um proxy da classe @Configuration. Isso garante que as chamadas aos métodos @Bean sejam interceptadas.

Por que isso importa?

No exemplo acima, o método userService() chama database(). 
Sem @Configuration (ou em Lite Mode): O método database() seria executado como uma chamada Java comum, criando uma nova instância de Database a cada chamada.
Com @Configuration: O Spring intercepta a chamada. Ele verifica: "Eu já tenho um bean database no container?" 
Se SIM: Retorna a instância existente (Singleton).
Se NÃO: Cria, armazena e retorna.

Isso garante o escopo Singleton padrão do Spring, evitando bugs críticos de múltiplas instâncias de serviços que deveriam ser únicos (como pools de conexão).

O que acontece se removermos o @Configuration?
Se mantivermos apenas os @Bean mas removermos a anotação da classe:

```Java
// Sem @Configuration
public class AppConfig {
    
    @Bean
    public Database database() { ... }

    @Bean
    public UserService userService() {
        return new UserService(database()); // PERIGO: Cria NOVA instância de Database!
    }
}
```

Neste cenário (Lite Mode), o Spring ainda registra os beans, mas a "mágica" do CGLIB não ocorre. 
O userService terá uma instância de Database e o container terá outra. Isso quebra a consistência do grafo de dependências.

Dica: Use @Configuration sempre que precisar de beans que dependem uns dos outros dentro da mesma classe de configuração para garantir a integridade do ciclo de vida do Spring.