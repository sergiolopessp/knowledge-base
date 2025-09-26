# Padrão Decorator

O Padrão Decorator permite adicionar funcionalidades a objetos de forma dinâmica, estendendo comportamentos sem modificar a classe original. É uma alternativa flexível à herança.

## Sumário

- [Padrão Decorator](#padrão-decorator)
  - [Componentes Principais](#componentes-principais)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo em Java 21](#exemplo-em-java-21)
    - [Componente](#componente)
    - [Componente Concreto](#componente-concreto)
    - [Decorator](#decorator)
    - [Decoradores Concretos](#decoradores-concretos)
    - [Uso](#uso)

## Componentes Principais

- **Componente**: Interface ou classe abstrata que define o contrato.
- **Componente Concreto**: Classe base que será decorada.
- **Decorator**: Classe abstrata que implementa o componente e contém uma referência a ele.
- **Decoradores Concretos**: Classes que adicionam funcionalidades específicas.

## Benefícios

- Adiciona funcionalidades dinamicamente sem alterar o código original.
- Promove flexibilidade e reutilização.
- Evita explosão de subclasses para combinações de funcionalidades.
- Segue o princípio aberto/fechado.

## Contras

- Aumenta o número de classes.
- Pode complicar o design com muitos decoradores.
- Requer cuidado para manter a transparência da interface.

## Quando Usar

Use o padrão Decorator quando você, como desenvolvedor, precisa adicionar funcionalidades a objetos de forma flexível em um sistema, como em um e-commerce onde produtos podem ter descontos, taxas ou embalagens especiais aplicados dinamicamente. Por exemplo, ao calcular o preço de um produto com impostos ou promoções sem alterar a classe base.

## Exemplo em Java 21

### Componente

Define o contrato para o cálculo do preço de um produto.

```java
public interface Produto {
    double calcularPreco();
}
```

### Componente Concreto

Classe base que representa um produto simples.

```java
public record ProdutoSimples(String nome, double precoBase) implements Produto {
    @Override
    public double calcularPreco() {
        return precoBase;
    }
}
```

### Decorator

Classe abstrata que implementa a interface `Produto` e contém uma referência ao componente.

```java
public abstract class DecoradorProduto implements Produto {
    protected final Produto produto;

    public DecoradorProduto(Produto produto) {
        this.produto = produto;
    }

    @Override
    public double calcularPreco() {
        return produto.calcularPreco();
    }
}
```

### Decoradores Concretos

Adicionam funcionalidades específicas, como impostos ou descontos.

```java
public class DecoradorImposto extends DecoradorProduto {
    private final double taxa;

    public DecoradorImposto(Produto produto, double taxa) {
        super(produto);
        this.taxa = taxa;
    }

    @Override
    public double calcularPreco() {
        return produto.calcularPreco() * (1 + taxa);
    }
}

public class DecoradorDesconto extends DecoradorProduto {
    private final double desconto;

    public DecoradorDesconto(Produto produto, double desconto) {
        super(produto);
        this.desconto = desconto;
    }

    @Override
    public double calcularPreco() {
        return produto.calcularPreco() - desconto;
    }
}
```

### Uso

Demonstra como aplicar múltiplos decoradores para modificar o preço de um produto.

```java
public class Main {
    public static void main(String[] args) {
        Produto produto = new ProdutoSimples("Notebook", 1000.0);
        System.out.printf("Preço base: %.2f%n", produto.calcularPreco());

        produto = new DecoradorImposto(produto, 0.1); // Adiciona 10% de imposto
        System.out.printf("Preço com imposto: %.2f%n", produto.calcularPreco());

        produto = new DecoradorDesconto(produto, 50.0); // Aplica desconto de 50
        System.out.printf("Preço com imposto e desconto: %.2f%n", produto.calcularPreco());
    }
}
```
