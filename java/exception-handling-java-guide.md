# Exception Handling em Java — Guia para Iniciantes

Este guia aborda boas práticas, conceitos e exemplos para lidar com exceções em Java, especialmente em aplicações backend (por exemplo, Spring Boot). Serve para desenvolvedores que estão aprendendo como tornar o tratamento de erros mais organizado, legível e seguro.

---

## Sumário

- [Introdução](#introdução)  
- [Tipos de Exceção em Java](#tipos-de-exceção-em-java)  
- [Sintomas Comuns & Diagnóstico](#sintomas-comuns--diagnóstico)  
- [Ferramentas de Logging e Debug](#ferramentas-de-logging-e-debug)  
- [Tratamento Centralizado de Exceções (Spring Boot)](#tratamento-centralizado-de-exceções-spring-boot)  
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
