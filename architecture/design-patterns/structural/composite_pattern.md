# Padrão Composite

O Padrão Composite permite tratar objetos individuais e composições de objetos de forma uniforme, organizando-os em uma estrutura hierárquica de árvore.

## Sumário

- [Padrão Composite](#padrão-composite)
  - [Componentes Principais](#componentes-principais)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo em Java 21](#exemplo-em-java-21)
    - [Componente](#componente)
    - [Folha (Leaf)](#folha-leaf)
    - [Composto (Composite)](#composto-composite)
    - [Uso](#uso)

## Componentes Principais

- **Componente**: Interface ou classe abstrata que define operações comuns.
- **Folha (Leaf)**: Classe que representa objetos individuais sem filhos.
- **Composto (Composite)**: Classe que contém outros componentes (folhas ou outros compostos).
- **Cliente**: Interage com componentes através da interface comum.

## Benefícios

- Permite tratar objetos individuais e grupos uniformemente.
- Facilita a manipulação de estruturas hierárquicas.
- Simplifica adição de novos tipos de componentes.
- Promove flexibilidade na construção de árvores.

## Contras

- Pode tornar o design mais complexo.
- Difícil restringir componentes em certos níveis da hierarquia.
- Pode impactar desempenho em árvores grandes.

## Quando Usar

Use o padrão Composite quando você, como desenvolvedor, precisa representar hierarquias em um sistema, como uma estrutura de categorias de produtos em um e-commerce. Por exemplo, ao modelar uma árvore de categorias (ex.: Eletrônicos > Computadores > Notebooks), onde tanto categorias quanto produtos individuais podem ser tratados de forma uniforme.

## Exemplo em Java 21

### Componente

Define a interface para categorias e produtos.

```java
public interface ItemCatalogo {
    void exibir();
    double getPreco();
}
```

### Folha (Leaf)

Representa um produto individual.

```java
public record Produto(String nome, double preco) implements ItemCatalogo {
    @Override
    public void exibir() {
        System.out.printf("Produto: %s, Preço: %.2f%n", nome, preco);
    }

    @Override
    public double getPreco() {
        return preco;
    }
}
```

### Composto (Composite)

Representa uma categoria que pode conter outros itens (produtos ou subcategorias).

```java
import java.util.ArrayList;
import java.util.List;

public class Categoria implements ItemCatalogo {
    private final String nome;
    private final List<ItemCatalogo> itens = new ArrayList<>();

    public Categoria(String nome) {
        this.nome = nome;
    }

    public void adicionar(ItemCatalogo item) {
        itens.add(item);
    }

    public void remover(ItemCatalogo item) {
        itens.remove(item);
    }

    @Override
    public void exibir() {
        System.out.println("Categoria: " + nome);
        for (ItemCatalogo item : itens) {
            item.exibir();
        }
    }

    @Override
    public double getPreco() {
        return itens.stream().mapToDouble(ItemCatalogo::getPreco).sum();
    }
}
```

### Uso

Demonstra a criação de uma hierarquia de categorias e produtos, com exibição e cálculo de preço total.

```java
public class Main {
    public static void main(String[] args) {
        // Criando produtos (folhas)
        var notebook = new Produto("Notebook", 3500.0);
        var smartphone = new Produto("Smartphone", 2000.0);

        // Criando categorias (compostos)
        var eletronicos = new Categoria("Eletrônicos");
        var computadores = new Categoria("Computadores");

        // Montando a hierarquia
        computadores.adicionar(notebook);
        eletronicos.adicionar(computadores);
        eletronicos.adicionar(smartphone);

        // Exibindo a estrutura
        eletronicos.exibir();

        // Calculando preço total
        System.out.printf("Preço total: %.2f%n", eletronicos.getPreco());
    }
}
```
