### Java File I/O: Guia de Escrita (Modern Java 2025)
A manipulação de arquivos em Java evoluiu significativamente. O foco está na utilização das APIs introduzidas a partir do Java 11 e 17, que oferecem maior performance e segurança de memória.

### O Padrão Moderno: Files.writeString (Java 11+)

Ideal para cenários onde você precisa escrever uma String completa de forma atômica. É a abordagem mais concisa para arquivos de texto pequenos e médios.

```Java

import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.StandardOpenOption;

Path path = Path.of("exemplo.txt");
String content = "Conteúdo para o arquivo";

// Escrita simples (sobrescreve por padrão)
Files.writeString(path, content);

// Para Append (adicionar ao final)
Files.writeString(path, content, StandardOpenOption.APPEND);
```

### Escrita em Lote: Files.write com Iterables

Excelente para logs ou processamento de dados onde as informações já estão organizadas em listas.

```Java

List<String> lines = List.of("Linha 1", "Linha 2", "Linha 3");
Files.write(Path.of("output.txt"), lines);
```

### Alta Performance: BufferedWriter

Para fluxos de dados contínuos ou arquivos muito grandes, o BufferedWriter continua sendo imbatível ao reduzir o número de chamadas de sistema (I/O).

```Java

try (BufferedWriter writer = Files.newBufferedWriter(path)) {
    writer.write("Escrita otimizada com buffer");
}
```

### Boas Práticas e Performance

*Try-with-resources*: Essencial para garantir que os file handles sejam fechados, especialmente em ambientes containerizados (Docker).

*Charset*: Utilize sempre StandardCharsets.UTF_8 explicitamente se não estiver usando os métodos padrão da classe Files.

*NIO vs IO*: Prefira sempre as classes do pacote java.nio.file (Path, Files) sobre a antiga java.io.File.