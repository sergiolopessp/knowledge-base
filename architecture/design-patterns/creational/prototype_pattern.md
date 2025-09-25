# Padrão Prototype

O Padrão Prototype permite criar novos objetos copiando instâncias existentes (protótipos), evitando a criação de objetos do zero e reduzindo custos computacionais.

## Componentes Principais
- **Prototype**: Interface ou classe abstrata que define o método de clonagem.
- **Prototype Concreto**: Implementa a clonagem e contém o estado a ser copiado.
- **Cliente**: Usa o protótipo para criar novos objetos por cópia.

## Benefícios
- Reduz o custo de criação de objetos complexos.
- Permite criar objetos com configurações pré-definidas.
- Evita dependência de subclasses para criação.
- Facilita a duplicação de objetos com estado.

## Contras
- Clonagem profunda pode ser complexa para objetos com referências.
- Requer implementação cuidadosa do método de clonagem.
- Pode aumentar a complexidade em sistemas simples.

## Quando Usar
Use o padrão Prototype quando você, como desenvolvedor, precisa criar objetos semelhantes repetidamente em um sistema, como em um e-commerce para duplicar configurações de produtos (ex.: criar variações de um item com pequenas alterações, como cor ou tamanho) sem recriar do zero.

## Exemplo em Java 21 

Exemplo em Java 21 utilizando o Lombok para evitar boilerplates e deixar o código mais limpo

### Prototype
Define a interface para clonagem.

```java
public interface Prototype extends Cloneable {
    Prototype clonar();
}
```

### Prototype Concreto
Implementa a clonagem de um produto com atributos específicos, usando Lombok para reduzir boilerplate.

```java
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
@ToString
public class Produto implements Prototype {
    private String nome;
    private double preco;
    private String cor;

    public Produto(String nome, double preco, String cor) {
        this.nome = nome;
        this.preco = preco;
        this.cor = cor;
    }

    @Override
    public Prototype clonar() {
        try {
            return (Produto) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException("Erro ao clonar produto", e);
        }
    }
}
```

### Uso
Demonstra a criação de novos produtos por clonagem com pequenas alterações.

```java
public class Main {
    public static void main(String[] args) {
        var produtoOriginal = new Produto("Camiseta", 50.0, "Azul");
        System.out.println("Original: " + produtoOriginal);

        var produtoClonado = (Produto) produtoOriginal.clonar();
        produtoClonado.setCor("Vermelha");
        System.out.println("Clonado com alteração: " + produtoClonado);
    }
}
```