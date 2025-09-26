# Padrão Chain of Responsibility

O Padrão Chain of Responsibility permite que uma solicitação seja passada por uma cadeia de manipuladores, onde cada manipulador decide processá-la ou passá-la ao próximo. É útil para evitar acoplamento entre remetente e receptor.

## Sumário

- [Padrão Chain of Responsibility](#padrão-chain-of-responsibility)
- [Componentes Principais](#componentes-principais)
- [Benefícios](#benefícios)
- [Contras](#contras)
- [Quando Usar](#quando-usar)
- [Exemplo em Java 21](#exemplo-em-java-21)
  - [Manipulador](#manipulador)
  - [Pedido](#pedido)
  - [Manipuladores Concretos](#manipuladores-concretos)
  - [Uso](#uso)

## Componentes Principais

- **Manipulador**: Interface ou classe abstrata que define o método de manipulação e a referência ao próximo manipulador.
- **Manipuladores Concretos**: Classes que implementam a lógica de manipulação.
- **Cliente**: Inicia a solicitação e a envia à cadeia.

## Benefícios

- Desacopla remetente e receptor.
- Permite adicionar ou remover manipuladores dinamicamente.
- Flexível para ordenar ou combinar responsabilidades.
- Promove o princípio da responsabilidade única.

## Contras

- Pode resultar em solicitações não tratadas se a cadeia não for configurada corretamente.
- Aumenta complexidade com muitos manipuladores.
- Pode impactar desempenho em cadeias longas.

## Quando Usar

Use o padrão Chain of Responsibility quando você, como desenvolvedor, precisa processar uma solicitação em várias etapas em um sistema, como em um e-commerce para validar um pedido (ex.: verificar estoque, validar pagamento, confirmar entrega). Cada etapa pode ser tratada por um manipulador, e a solicitação passa pela cadeia até ser completamente processada.

## Exemplo em Java 21

### Manipulador

Define a interface para manipular solicitações de pedidos.

```java
public interface ManipuladorPedido {
    void setProximo(ManipuladorPedido proximo);
    void processar(Pedido pedido);
}
```

### Pedido

Representa a solicitação a ser processada.

```java
public record Pedido(String produto, int quantidade, double valor) {}
```

### Manipuladores Concretos

Cada manipulador verifica uma condição específica e passa a solicitação ao próximo, se necessário.

```java
public class VerificadorEstoque implements ManipuladorPedido {
    private ManipuladorPedido proximo;

    @Override
    public void setProximo(ManipuladorPedido proximo) {
        this.proximo = proximo;
    }

    @Override
    public void processar(Pedido pedido) {
        if (pedido.quantidade() > 0) {
            System.out.println("Estoque verificado para " + pedido.produto());
            if (proximo != null) proximo.processar(pedido);
        } else {
            System.out.println("Estoque insuficiente para " + pedido.produto());
        }
    }
}

public class VerificadorPagamento implements ManipuladorPedido {
    private ManipuladorPedido proximo;

    @Override
    public void setProximo(ManipuladorPedido proximo) {
        this.proximo = proximo;
    }

    @Override
    public void processar(Pedido pedido) {
        if (pedido.valor() > 0) {
            System.out.println("Pagamento validado: " + pedido.valor());
            if (proximo != null) proximo.processar(pedido);
        } else {
            System.out.println("Pagamento inválido para " + pedido.produto());
        }
    }
}

public class ConfirmadorEntrega implements ManipuladorPedido {
    private ManipuladorPedido proximo;

    @Override
    public void setProximo(ManipuladorPedido proximo) {
        this.proximo = proximo;
    }

    @Override
    public void processar(Pedido pedido) {
        System.out.println("Entrega confirmada para " + pedido.produto());
        if (proximo != null) proximo.processar(pedido);
    }
}
```

### Uso

Demonstra a configuração da cadeia e o processamento de um pedido.

```java
public class Main {
    public static void main(String[] args) {
        var estoque = new VerificadorEstoque();
        var pagamento = new VerificadorPagamento();
        var entrega = new ConfirmadorEntrega();

        // Configurando a cadeia
        estoque.setProximo(pagamento);
        pagamento.setProximo(entrega);

        // Processando um pedido
        var pedido = new Pedido("Notebook", 1, 3500.0);
        estoque.processar(pedido);

        // Pedido inválido
        var pedidoInvalido = new Pedido("Smartphone", 0, 2000.0);
        estoque.processar(pedidoInvalido);
    }
}
```
