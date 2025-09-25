# Padrão Factory Method

O Padrão Factory Method define uma interface para criar objetos, mas permite que subclasses decidam qual classe instanciar. A criação é delegada às subclasses, diferentemente do Factory Pattern, que centraliza a lógica de criação.

## Componentes Principais
- **Produto**: Interface ou classe abstrata que define o tipo de objeto.
- **Produtos Concretos**: Implementações específicas do produto.
- **Fábrica Abstrata**: Define o método de criação (factory method).
- **Fábricas Concretas**: Subclasses que implementam o método de criação.

## Benefícios
- Permite que subclasses controlem a criação de objetos.
- Reduz acoplamento entre cliente e produtos.
- Facilita extensão para novos tipos de produtos.
- Promove consistência na criação.

## Contras
- Aumenta o número de classes.
- Pode complicar sistemas simples.
- Requer manutenção de subclasses para novos produtos.

## Quando Usar
Use o padrão Factory Method quando você, como desenvolvedor, precisa criar objetos em um sistema onde o tipo exato do objeto depende de uma condição ou configuração, como em um sistema de pagamento que suporta diferentes métodos (cartão, PayPal, etc.). Por exemplo, ao desenvolver um módulo de checkout em um e-commerce, onde cada método de pagamento exige uma lógica de criação específica, mas você quer delegar a decisão de qual classe instanciar para subclasses especializadas, facilitando a adição de novos métodos de pagamento no futuro.

## Diferença entre Factory Method e Factory Pattern
- **Factory Pattern**: Usa uma classe única (fábrica) com lógica centralizada (ex.: `switch`) para criar objetos. Exemplo: uma classe `FabricaPagamento` que instancia `PagamentoCartao` ou `PagamentoPayPal` com base em um parâmetro.
- **Factory Method**: Delega a criação às subclasses, onde cada uma implementa o método de criação para um tipo específico de produto. A lógica de criação é distribuída, não centralizada.

## Exemplo em Java 21

### Interface Produto
Define o contrato para processar pagamentos.

```java
public interface Pagamento {
    void processar(double valor);
}
```

### Produtos Concretos
Implementações específicas do pagamento, usando `record` do Java 21.

```java
public record PagamentoCartao(String numeroCartao) implements Pagamento {
    @Override
    public void processar(double valor) {
        System.out.printf("Processado %.2f com cartão: %s%n", valor, numeroCartao);
    }
}

public record PagamentoPayPal(String email) implements Pagamento {
    @Override
    public void processar(double valor) {
        System.out.printf("Processado %.2f com PayPal: %s%n", valor, email);
    }
}
```

### Fábrica Abstrata
Define o método abstrato `criarPagamento` que as subclasses implementarão.

```java
public abstract class FabricaPagamento {
    public abstract Pagamento criarPagamento(String identificador);

    public void processarPagamento(double valor, String identificador) {
        Pagamento pagamento = criarPagamento(identificador);
        pagamento.processar(valor);
    }
}
```