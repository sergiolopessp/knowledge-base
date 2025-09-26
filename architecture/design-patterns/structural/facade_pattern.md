# Padrão Facade

O Padrão Facade fornece uma interface simplificada para um conjunto complexo de classes ou subsistemas, facilitando o uso por clientes.

## Sumário

- [Padrão Facade](#padrão-facade)
  - [Componentes Principais](#componentes-principais)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo em Java 21](#exemplo-em-java-21)
    - [Subsistema](#subsistema)
    - [Facade](#facade)
    - [Uso](#uso)

## Componentes Principais

- **Facade**: Classe que oferece uma interface simplificada para interagir com o subsistema.
- **Subsistema**: Conjunto de classes com funcionalidades complexas que a facade encapsula.
- **Cliente**: Usa a facade para acessar o subsistema.

## Benefícios

- Simplifica a interação com sistemas complexos.
- Reduz acoplamento entre cliente e subsistema.
- Melhora a legibilidade e usabilidade do código.
- Facilita manutenção ao centralizar acesso.

## Contras

- Pode limitar flexibilidade ao esconder detalhes do subsistema.
- Risco de se tornar uma fachada excessivamente grande.
- Adiciona uma camada extra ao design.

## Quando Usar

Use o padrão Facade quando você, como desenvolvedor, precisa simplificar a interação com um sistema complexo, como em um e-commerce onde o cliente realiza um pedido envolvendo múltiplos subsistemas (estoque, pagamento, entrega). A facade oferece uma única interface para coordenar essas operações, escondendo a complexidade.

## Exemplo em Java 21

### Subsistema

Classes que representam subsistemas complexos.

```java
public class Estoque {
    public boolean verificarDisponibilidade(String produto, int quantidade) {
        System.out.println("Verificando estoque para " + produto + ": " + quantidade + " disponível");
        return true;
    }
}

public class Pagamento {
    public void processarPagamento(String metodo, double valor) {
        System.out.printf("Processando pagamento de %.2f via %s%n", valor, metodo);
    }
}

public class Entrega {
    public void agendarEntrega(String endereco) {
        System.out.println("Entrega agendada para: " + endereco);
    }
}
```

### Facade

Fornece uma interface simplificada para realizar um pedido.

```java
public class FacadeLoja {
    private final Estoque estoque;
    private final Pagamento pagamento;
    private final Entrega entrega;

    public FacadeLoja() {
        this.estoque = new Estoque();
        this.pagamento = new Pagamento();
        this.entrega = new Entrega();
    }

    public void realizarPedido(String produto, int quantidade, String metodoPagamento, double valor, String endereco) {
        if (estoque.verificarDisponibilidade(produto, quantidade)) {
            pagamento.processarPagamento(metodoPagamento, valor);
            entrega.agendarEntrega(endereco);
            System.out.println("Pedido concluído com sucesso!");
        } else {
            System.out.println("Produto indisponível no estoque.");
        }
    }
}
```

### Uso

Demonstra como o cliente usa a facade para realizar um pedido sem interagir diretamente com os subsistemas.

```java
public class Main {
    public static void main(String[] args) {
        var loja = new FacadeLoja();
        loja.realizarPedido("Notebook", 1, "Cartão", 3500.0, "Rua Exemplo, 123");
    }
}
```
