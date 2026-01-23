# Java String indexOf(): Guia de Referência

O método `indexOf()` da classe `String` é uma ferramenta fundamental para busca e manipulação de texto em Java. Ele retorna a primeira ocorrência do caractere ou substring especificada, ou `-1` caso não encontre correspondência.

## Sobrecargas do Método (Method Overloading)

Java oferece quatro variações principais do `indexOf()`, permitindo flexibilidade na busca:

1.  **`indexOf(int ch)`**: Busca um caractere específico (utiliza o valor Unicode).
2.  **`indexOf(int ch, int fromIndex)`**: Busca um caractere a partir de uma posição inicial.
3.  **`indexOf(String str)`**: Busca a primeira ocorrência de uma substring.
4.  **`indexOf(String str, int fromIndex)`**: Busca uma substring a partir de uma posição inicial.

## Exemplos Práticos

### 1. Busca Básica
```java
String texto = "Dominando o ecossistema Java!";

int pos1 = texto.indexOf("Java"); // Retorna 25
int pos2 = texto.indexOf('o');    // Retorna 1 (primeiro 'o' em 'Dominando')
int pos3 = texto.indexOf("Go");   // Retorna -1 (não encontrado)
```

### 2. Busca com Ponto de Partida (fromIndex)
Útil para encontrar múltiplas ocorrências em um loop ou pular partes conhecidas da string.

```Java

String msg = "Java é robusto. Java é escalável.";
int primeira = msg.indexOf("Java");             // 0
int segunda = msg.indexOf("Java", primeira + 1); // 16
```

### 3. Validação de Presença
O padrão clássico para checar se um elemento existe no texto.

```Java

if (filename.indexOf(".pdf") != -1) {
    // Processar arquivo PDF
}
```

## Casos de Uso Reais
Extração de Domínio de E-mail

```Java

String email = "dev.senior@cloud.com";
int atIndex = email.indexOf('@');

if (atIndex != -1) {
    String dominio = email.substring(atIndex + 1);
    // Resultado: cloud.com
}
```
Filtragem de Logs
Identificação rápida de níveis de severidade em strings de log.

```Java

String log = "ERROR 2026-01-23: Database connection failed";
if (log.indexOf("ERROR") == 0) {
    enviarAlertaCritico(log);
}
```

## Performance e Boas Práticas (Dicas de Engenheiro)

Char vs String: Prefira indexOf('a') em vez de indexOf("a") quando buscar um único caractere. A busca por primitiva (char) é marginalmente mais eficiente que a de um objeto String.

Regex: O indexOf() não aceita expressões regulares. Se precisar de busca por padrões (ex: [0-9]), utilize java.util.regex.Pattern.

Segurança: Ao usar o resultado do indexOf() em métodos como substring(), sempre valide se o retorno foi diferente de -1 para evitar StringIndexOutOfBoundsException.

Fonte: [Master Java indexOf() - Dev.to](https://dev.to/satyam_gupta_0d1ff2152dcc/master-java-indexof-your-ultimate-guide-to-finding-strings-1ohd)