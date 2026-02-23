# Exception Handling em Java — Guia para Iniciantes

Este guia aborda boas práticas, conceitos e exemplos para lidar com exceções em Java, especialmente em aplicações backend (por exemplo, Spring Boot). Serve para desenvolvedores que estão aprendendo como tornar o tratamento de erros mais organizado, legível e seguro.

---

## Sumário

- [Introdução](#introdução)  
- [Tipos de Exceção em Java](#tipos-de-exceção-em-java)  
- [Sintomas Comuns & Diagnóstico](#sintomas-comuns--diagnóstico)  
- [Ferramentas de Logging e Debug](#ferramentas-de-logging-e-debug)  
- [Tratamento Centralizado de Exceções (Spring Boot)](#tratamento-centralizado-de-exceções-spring-boot)  
- [Java Lambdas e o Tratamento de Exceções](#java-lambdas-e-o-tratamento-de-exceções)   
- [Boas Práticas](#boas-práticas)  
- [Resumo](#resumo)  

---

## Introdução

Quando comecei a desenvolver com Java + Spring Boot, minha aplicação caía com `NullPointerException` inesperadas em endpoints, retornando 500 Internal Server Error. O log era confuso, o cliente via só stack trace bruto, sem clareza do problema nem de como resolver.

Este guia mostra:

- como diagnosticar o que está errado  
- como evitar repetição de blocos `try‑catch` espalhados  
- como dar respostas significativas aos usuários da API  

---

## Tipos de Exceção em Java

- **Checked Exceptions** — devem ser declaradas ou tratadas no método (ex: `IOException`)  
- **Unchecked Exceptions** — subtipos de `RuntimeException` (ex: `NullPointerException`) que ocorrem em tempo de execução e não são obrigatoriamente tratadas  

---

## Sintomas Comuns & Diagnóstico

- Mensagem de erro como:

  ```
  java.lang.NullPointerException: Cannot invoke "String.length()" because "name" is null
      at com.example.demo.service.UserService.getUser(UserService.java:25)
      at com.example.demo.controller.UserController.getUser(UserController.java:18)
  ```

- Logs mostram valores nulos inesperados  
- Endpoint retorna 500 Internal Server Error sem contexto útil  

---

## Ferramentas de Logging e Debug

- Inserir logs de debug para parâmetros de entrada, antes de chamadas que podem falhar  
- Ex:  
  ```java
  System.out.println("DEBUG: nome recebido -> " + name);
  ```
- Usar frameworks de logging (SLF4J, Logback ou Log4j) para níveis diferentes (DEBUG, INFO, WARN, ERROR)  

---

## Tratamento Centralizado de Exceções (Spring Boot)

Em vez de múltiplos `try‑catch`, usar um controlador global para capturar exceções de forma organizada.

Exemplo:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(NullPointerException.class)
    public ResponseEntity<String> handleNullPointer(NullPointerException ex) {
        return new ResponseEntity<>("Oops! Um valor obrigatório estava faltando.", HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleGeneralException(Exception ex) {
        return new ResponseEntity<>("Algo deu errado: " + ex.getMessage(),
                                    HttpStatus.INTERNAL_SERVER_ERROR);
    }

}
```

---

## Java Lambdas e o Tratamento de Exceções
Trabalhar com expressões Lambda em Java simplifica a sintaxe, mas introduz um desafio: as interfaces funcionais da JDK não declaram exceções verificadas (checked exceptions). Isso impede o lançamento direto de exceções como IOException ou SQLException dentro de um Stream, por exemplo.

Interfaces como java.util.function.Function definem o método R apply(T t), sem a cláusula throws. Se o código dentro da Lambda lançar uma checked exception, o compilador exigirá o tratamento imediato ou uma mudança na estratégia.

**Estratégias de Implementação**
1. Wrapper Methods (Abordagem Centralizada)
Esta é a forma mais limpa de lidar com o problema, encapsulando a lógica de tratamento em um método utilitário que converte a exceção verificada em uma não verificada (RuntimeException).

```Java
public class LambdaWrapper {
    public static <T, R> Function<T, R> wrap(CheckedFunction<T, R, Exception> checkedFunction) {
        return t -> {
            try {
                return checkedFunction.apply(t);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        };
    }
}

@FunctionalInterface
public interface CheckedFunction<T, R, E extends Exception> {
    R apply(T t) throws E;
}

// Uso prático:
list.stream()
    .map(LambdaWrapper.wrap(path -> Files.readAllLines(path)))
    .collect(Collectors.toList());
```

2. Tratamento Local (Try-Catch Inline)
Útil para casos simples onde o erro deve ser logado e o fluxo deve continuar, ou quando um valor padrão (fallback) é aceitável. 

```Java
list.forEach(item -> {
    try {
        processItem(item);
    } catch (IOException e) {
        logger.error("Erro ao processar item: " + item, e);
        // Tratamento local ou skip
    }
});
```
3. Interfaces Funcionais Customizadas
Em arquiteturas de domínio fechado, pode-se definir interfaces que já suportam exceções específicas, evitando o uso de interfaces genéricas da JDK.

**Lições Aprendidas**

Separação de Preocupações: Mantenha a lógica de tratamento de erro fora da linha principal do Stream. Lambdas devem ser concisas, se o try-catch ocupar muitas linhas, extraia-o para um método.

RuntimeException vs. Checked: Ao usar wrappers, lembre-se de que converter para RuntimeException pode dificultar o tratamento específico por quem chama o método. Considere criar exceções de domínio (ex: PaymentException).

Legibilidade: No contexto de engenharia de software de longo prazo, a legibilidade supera a brevidade. Se uma Lambda se tornar complexa devido ao tratamento de erros, considere voltar ao loop for tradicional ou usar métodos de referência.


## Boas Práticas

- Nunca expor stack trace completo para o cliente em ambientes de produção  
- Sempre retornar respostas de erro com campos claros: código HTTP, mensagem legível, talvez um código interno  
- Validar entradas (por exemplo, com Bean Validation) para evitar exceções de valores nulos ou inválidos  
- Usar logs para capturar contexto (método, parâmetros, usuário, request id)  
- Usar tipos de exceção específicos em vez de capturar `Exception` genérico sempre que possível  

---

## Resumo

- Exceções são ferramentas de diagnóstico, não só ruídos — tratá‑las bem melhora manutenção e experiência do usuário  
- Identificar sintomas cedo, usar logs, triviar parâmetros de entrada  
- Centralizar tratamento de exceções com classes dedicadas (por exemplo, com `@ControllerAdvice`)  
- Produzir respostas úteis, evitar vazamento de informações internas  

---

_Referência adaptada: “A Beginner’s Guide to Exception Handling in Java” por Mohd Rehan no DEV Community_
