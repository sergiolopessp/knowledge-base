# Padrão Strategy

O Padrão Strategy define uma família de algoritmos, encapsula cada um e os torna intercambiáveis. Permite que o algoritmo varie independentemente dos clientes que o utilizam.

# Sumário

- [Padrão Strategy](#padrão-strategy)
    - [Componentes Principais](#componentes-principais)
    - [Benefícios](#benefícios)
    - [Contras](#contras)
    - [Quando Usar](#quando-usar)
    - [Exemplo em Java 21](#exemplo-em-java-21)
        - [Interface Strategy](#interface-strategy)
        - [Estratégias Concretas](#estratégias-concretas)
        - [Contexto](#contexto)
        - [Uso](#uso)

## Componentes Principais
- **Contexto**: Classe que usa a estratégia.
- **Strategy**: Interface que define o algoritmo.
- **Estratégias Concretas**: Implementações da interface de estratégia.

## Benefícios
- Promove flexibilidade e reusabilidade.
- Evita condicionais para seleção de algoritmos.
- Facilita adição de novas estratégias.
- Encapsula algoritmos, melhorando manutenção.

## Contras
- Aumenta número de classes no projeto.
- Pode complicar design para casos simples.
- Requer conhecimento das estratégias pelo cliente.

## Quando Usar
Use o padrão Strategy quando você, como desenvolvedor, precisa implementar comportamentos intercambiáveis em um sistema, como diferentes métodos de pagamento em um e-commerce. Por exemplo, ao processar pagamentos com cartão, PayPal ou transferência bancária, onde o cliente pode escolher o método dinamicamente. O padrão é ideal para evitar condicionais extensas (ex.: `if-else`) e permitir a adição de novos métodos de pagamento sem alterar o código do contexto.

## Exemplo em Java 21

### Interface Strategy
A interface `EstrategiaPagamento` define o contrato para as estratégias de pagamento, com um método `pagar` que aceita um valor.

```java
public interface EstrategiaPagamento {
    void pagar(double valor);
}
```

### Estratégias Concretas
As classes `PagamentoCartao` e `PagamentoPayPal` são implementações da interface, usando `record` do Java 21 para concisão. Cada uma implementa o método `pagar` com a lógica específica.

```java
public record PagamentoCartao(String numeroCartao) implements EstrategiaPagamento {
    @Override
    public void pagar(double valor) {
        System.out.printf("Pago %.2f com cartão: %s%n", valor, numeroCartao);
    }
}

public record PagamentoPayPal(String email) implements EstrategiaPagamento {
    @Override
    public void pagar(double valor) {
        System.out.printf("Pago %.2f com PayPal: %s%n", valor, email);
    }
}
```

### Contexto
A classe `CarrinhoCompras` atua como o contexto, mantendo uma referência à estratégia de pagamento e delegando a execução do método `pagar`.

```java
public class CarrinhoCompras {
    private EstrategiaPagamento estrategiaPagamento;

    public void setEstrategiaPagamento(EstrategiaPagamento estrategia) {
        this.estrategiaPagamento = estrategia;
    }

    public void finalizarCompra(double valor) {
        estrategiaPagamento.pagar(valor);
    }
}
```

### Uso
O código na classe `Main` demonstra a flexibilidade do padrão, alternando entre estratégias de pagamento (`PagamentoCartao` e `PagamentoPayPal`) dinamicamente.

```java
public class Main {
    public static void main(String[] args) {
        var carrinho = new CarrinhoCompras();

        carrinho.setEstrategiaPagamento(new PagamentoCartao("1234-5678-9012-3456"));
        carrinho.finalizarCompra(100.0);

        carrinho.setEstrategiaPagamento(new PagamentoPayPal("exemplo@email.com"));
        carrinho.finalizarCompra(50.0);
    }
}
```