# Padrão Visitor

O Padrão Visitor permite definir novas operações para uma estrutura de objetos sem modificar suas classes, separando a lógica de operação da estrutura de dados.

## Componentes Principais
- **Visitor**: Interface que define métodos para visitar cada tipo de elemento.
- **Visitor Concreto**: Implementa as operações específicas para cada elemento.
- **Elemento**: Interface que define o método para aceitar visitantes.
- **Elemento Concreto**: Classes que implementam o elemento e aceitam visitantes.
- **Estrutura de Objetos**: Coleção que contém os elementos a serem visitados.

## Benefícios
- Separa operações da estrutura de objetos.
- Facilita adição de novas operações sem alterar classes.
- Permite acumular estado durante a visitação.
- Segue o princípio aberto/fechado.

## Contras
- Aumenta complexidade com classes adicionais.
- Requer modificação de elementos para novos tipos.
- Pode violar encapsulamento se visitantes acessarem estado interno.

## Quando Usar
Use o padrão Visitor quando você, como desenvolvedor, precisa realizar operações variadas sobre uma estrutura de objetos em um sistema, como em um e-commerce para gerar relatórios (ex.: calcular preços ou gerar descrições) sobre diferentes tipos de produtos sem modificar suas classes.

## Exemplo em Java 21

### Elemento
Define a interface para aceitar visitantes.

```java
public interface Produto {
    void aceitar(Visitor visitor);
}
```

### Elementos Concretos
Representam diferentes tipos de produtos.

```java
public record Livro(String nome, double preco) implements Produto {
    @Override
    public void aceitar(Visitor visitor) {
        visitor.visitar(this);
    }
}

public record Eletronico(String nome, double preco) implements Produto {
    @Override
    public void aceitar(Visitor visitor) {
        visitor.visitar(this);
    }
}
```

### Visitor
Define a interface para operações em cada tipo de produto.

```java
public interface Visitor {
    void visitar(Livro livro);
    void visitar(Eletronico eletronico);
}
```

### Visitor Concreto
Implementa uma operação específica (calcular preço com desconto).

```java
public class VisitorDesconto implements Visitor {
    @Override
    public void visitar(Livro livro) {
        double precoComDesconto = livro.preco() * 0.9; // 10% de desconto
        System.out.printf("Livro: %s, Preço com desconto: %.2f%n", livro.nome(), precoComDesconto);
    }

    @Override
    public void visitar(Eletronico eletronico) {
        double precoComDesconto = eletronico.preco() * 0.8; // 20% de desconto
        System.out.printf("Eletrônico: %s, Preço com desconto: %.2f%n", eletronico.nome(), precoComDesconto);
    }
}
```

### Uso
Demonstra a aplicação de um visitante em uma lista de produtos.

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        var produtos = List.of(
            new Livro("Java Programming", 100.0),
            new Eletronico("Smartphone", 2000.0)
        );

        var visitor = new VisitorDesconto();

        for (var produto : produtos) {
            produto.aceitar(visitor);
        }
    }
}
```