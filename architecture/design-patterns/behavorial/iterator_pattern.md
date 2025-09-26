# Padrão Iterator

O Padrão Iterator fornece uma maneira de acessar elementos de uma coleção sequencialmente sem expor sua representação interna.

## Sumário

- [Padrão Iterator](#padrão-iterator)
- [Componentes Principais](#componentes-principais)
- [Benefícios](#benefícios)
- [Contras](#contras)
- [Quando Usar](#quando-usar)
- [Exemplo em Java 21](#exemplo-em-java-21)
  - [Iterator](#iterator)
  - [Agregado](#agregado)
  - [Agregado Concreto](#agregado-concreto)
  - [Uso](#uso)

## Componentes Principais

- **Iterator**: Interface que define métodos para navegar pela coleção.
- **Iterator Concreto**: Implementa a lógica de iteração.
- **Agregado**: Interface que define o método para criar um iterator.
- **Agregado Concreto**: Coleção que implementa o agregado e retorna o iterator.

## Benefícios

- Encapsula a lógica de iteração.
- Permite diferentes formas de percorrer a mesma coleção.
- Simplifica a interface do cliente com a coleção.
- Suporta iteração sem expor a estrutura interna.

## Contras

- Pode ser desnecessário para coleções simples.
- Adiciona complexidade em sistemas com iterações básicas.
- Pode impactar desempenho em coleções pequenas.

## Quando Usar

Use o padrão Iterator quando você, como desenvolvedor, precisa percorrer coleções complexas em um sistema, como uma lista de produtos em um e-commerce, sem expor a estrutura interna da coleção (ex.: array, lista encadeada). É ideal para permitir diferentes tipos de iteração (ex.: por categoria, preço) ou quando a coleção pode mudar de implementação.

## Exemplo em Java 21

### Iterator

Define a interface para navegar pela coleção.

```java
public interface Iterator<T> {
    boolean temProximo();
    T proximo();
}
```

### Agregado

Define a interface para criar um iterator.

```java
public interface Catalogo {
    Iterator<Produto> criarIterator();
}
```

### Agregado Concreto

Representa a coleção de produtos e retorna o iterator.

```java
public record Produto(String nome, double preco) {}

public class CatalogoProdutos implements Catalogo {
    private final Produto[] produtos;

    public CatalogoProdutos(Produto[] produtos) {
        this.produtos = produtos;
    }

    @Override
    public Iterator<Produto> criarIterator() {
        return new IteratorProduto(this);
    }

    // Classe interna para acessar produtos
    private class IteratorProduto implements Iterator<Produto> {
        private final CatalogoProdutos catalogo;
        private int indice = 0;

        public IteratorProduto(CatalogoProdutos catalogo) {
            this.catalogo = catalogo;
        }

        @Override
        public boolean temProximo() {
            return indice < catalogo.produtos.length;
        }

        @Override
        public Produto proximo() {
            return catalogo.produtos[indice++];
        }
    }
}
```

### Uso

Demonstra a iteração sobre uma coleção de produtos sem expor sua estrutura interna.

```java
public class Main {
    public static void main(String[] args) {
        var produtos = new Produto[] {
            new Produto("Notebook", 3500.0),
            new Produto("Smartphone", 2000.0)
        };

        var catalogo = new CatalogoProdutos(produtos);
        var iterator = catalogo.criarIterator();

        while (iterator.temProximo()) {
            var produto = iterator.proximo();
            System.out.printf("Produto: %s, Preço: %.2f%n", produto.nome(), produto.preco());
        }
    }
}
```
