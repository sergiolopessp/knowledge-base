# Java Streams: 5 Técnicas Poderosas
Este documento explora técnicas avançadas da API de Streams do Java que otimizam o processamento de coleções e melhoram a legibilidade do código, indo além das operações básicas de filter e map.

1 - Stream.iterate para Sequências Complexas
O método iterate permite criar streams infinitos ou finitos de forma declarativa, funcionando como um substituto moderno para o laço for.

```Java
// Gerando os primeiros 10 números da sequência de Fibonacci
Stream.iterate(new int[]{0, 1}, f -> new int[]{f[1], f[0] + f[1]})
      .limit(10)
      .map(f -> f[0])
      .forEach(System.out::println);
```
2 - Collectors.partitioningBy
Diferente do groupingBy, o partitioningBy é especializado em dividir dados em dois grupos baseados em um predicado booleano (true ou false). É extremamente eficiente para separações binárias.

```Java
Map<Boolean, List<Student>> passingFailing = students.stream()
    .collect(Collectors.partitioningBy(s -> s.getGrade() >= 6.0));
```

3 - Collectors.collectingAndThen
Esta técnica permite realizar uma transformação final logo após a coleta dos dados. É ideal para transformar uma lista resultante em uma visão imutável.

```Java
List<String> immutableList = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.collectingAndThen(Collectors.toList(), Collections::unmodifiableList));
```    
4 - Stream.takeWhile e Stream.dropWhile
Introduzidos no Java 9, esses métodos permitem processar elementos enquanto uma condição for verdadeira. Ao contrário do filter, eles param o processamento assim que a condição falha, o que é crucial para performance em streams ordenados e extensos.

takeWhile: Retorna os elementos até encontrar o primeiro que não satisfaz o predicado.

dropWhile: Descarta os elementos até encontrar o primeiro que satisfaz o predicado e retorna o restante.

```Java
List<Integer> numbers = List.of(1, 2, 3, 10, 4, 5);
numbers.stream()
       .takeWhile(n -> n < 5)
       .forEach(System.out::println); // Saída: 1, 2, 3
```
5 - Stream.flatMap para Achatamento de Estruturas
O flatMap é essencial quando lidamos com coleções aninhadas. Ele transforma cada elemento em um stream e então concatena todos esses streams em um único, facilitando o processamento de dados complexos.

```Java
List<String> distinctSkills = developers.stream()
    .flatMap(dev -> dev.getLanguages().stream())
    .distinct()
    .collect(Collectors.toList());
```    

Referências
[Java Code Geeks - Java Streams: 5 Powerful Techniques](https://www.javacodegeeks.com/2024/04/java-streams-5-powerful-techniques-you-might-not-know.html)

[Documentação Oficial do Java (Oracle)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html)