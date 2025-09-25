# Padrão Builder

O Padrão Builder separa a construção de um objeto complexo de sua representação, permitindo criar diferentes configurações do mesmo objeto de forma controlada e legível.

## Componentes Principais
- **Produto**: Objeto complexo a ser construído.
- **Builder**: Interface ou classe abstrata que define os passos de construção.
- **Concrete Builder**: Implementa os passos de construção.
- **Diretor**: Opcional, orquestra a construção usando o builder.

## Benefícios
- Permite construção passo a passo de objetos complexos.
- Facilita a criação de diferentes configurações.
- Melhora a legibilidade e manutenção do código.
- Encapsula a lógica de construção.

## Contras
- Aumenta o número de classes.
- Pode ser desnecessário para objetos simples.
- Requer manutenção adicional para novos atributos.

## Quando Usar
Use o padrão Builder quando você, como desenvolvedor, precisa criar um objeto com muitos atributos opcionais em um sistema de e-commerce. Por exemplo, ao modelar um pedido com campos como cliente, produto, quantidade, preço e endereço de entrega, onde nem todos são obrigatórios. O Builder permite configurar apenas os campos necessários de forma clara, evitando construtores sobrecarregados ou setters repetitivos, especialmente ao lidar com diferentes tipos de pedidos (ex.: com ou sem entrega expressa).

## Exemplo em Java 21 com Lombok

### Produto
Representa o objeto complexo, um pedido com vários atributos, usando anotações do Lombok para reduzir boilerplate.

```java
import lombok.Builder;
import lombok.Getter;
import lombok.ToString;

@Getter
@ToString
@Builder
public class Pedido {
    private final String cliente;
    private final String produto;
    private final int quantidade;
    private final double precoUnitario;
    private final String enderecoEntrega;
}
```

### Uso
Demonstra a construção de um pedido configurando atributos individualmente, usando o builder gerado automaticamente pelo Lombok.

```java
public class Main {
    public static void main(String[] args) {
        var pedido = Pedido.builder()
                .cliente("João Silva")
                .produto("Notebook")
                .quantidade(2)
                .precoUnitario(3500.00)
                .enderecoEntrega("Rua Exemplo, 123")
                .build();

        System.out.println(pedido);
    }
}
```