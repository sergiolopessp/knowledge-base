# Padrão Flyweight

O Padrão Flyweight reduz o uso de memória ao compartilhar objetos com estado comum (intrínseco), mantendo estados específicos (extrínsecos) separados. É ideal para gerenciar muitos objetos semelhantes.

## Componentes Principais
- **Flyweight**: Interface ou classe abstrata que define operações.
- **Flyweight Concreto**: Implementa o flyweight com estado intrínseco.
- **Fábrica Flyweight**: Gerencia a criação e compartilhamento de flyweights.
- **Cliente**: Mantém o estado extrínseco e usa flyweights.

## Benefícios
- Reduz consumo de memória ao compartilhar objetos.
- Melhora desempenho em sistemas com muitos objetos semelhantes.
- Permite escalabilidade em aplicações com dados repetitivos.
- Separa estados intrínsecos e extrínsecos.

## Contras
- Pode aumentar complexidade do design.
- Gerenciamento de estado extrínseco pode ser complicado.
- Requer cuidado em sistemas multithread.

## Quando Usar
Use o padrão Flyweight quando você, como desenvolvedor, precisa gerenciar muitos objetos com características compartilhadas em um sistema, como em um e-commerce para representar produtos com atributos comuns (ex.: nome, preço) que se repetem, enquanto estados específicos (ex.: quantidade no carrinho) são mantidos separadamente.

## Exemplo em Java 21

### Flyweight
Define a interface para produtos com estado intrínseco.

```java
public interface ProdutoFlyweight {
    void exibir(String estadoExtrinseco);
}
```

### Flyweight Concreto
Armazena o estado intrínseco (nome, preço) e usa o estado extrínseco (quantidade).

```java
public record ProdutoConcreto(String nome, double preco) implements ProdutoFlyweight {
    @Override
    public void exibir(String estadoExtrinseco) {
        System.out.printf("Produto: %s, Preço: %.2f, Estado: %s%n", nome, preco, estadoExtrinseco);
    }
}
```

### Fábrica Flyweight
Gerencia a criação e compartilhamento de flyweights.

```java
import java.util.HashMap;
import java.util.Map;

public class FabricaProduto {
    private final Map<String, ProdutoFlyweight> flyweights = new HashMap<>();

    public ProdutoFlyweight getProduto(String nome, double preco) {
        String chave = nome + "_" + preco;
        return flyweights.computeIfAbsent(chave, k -> new ProdutoConcreto(nome, preco));
    }
}
```

### Uso
Demonstra o compartilhamento de flyweights para produtos com estados extrínsecos diferentes.

```java
public class Main {
    public static void main(String[] args) {
        var fabrica = new FabricaProduto();

        var produto1 = fabrica.getProduto("Notebook", 3500.0);
        produto1.exibir("Quantidade: 1");

        var produto2 = fabrica.getProduto("Notebook", 3500.0); // Reutiliza o mesmo flyweight
        produto2.exibir("Quantidade: 3");

        var produto3 = fabrica.getProduto("Smartphone", 2000.0);
        produto3.exibir("Quantidade: 2");
    }
}
```