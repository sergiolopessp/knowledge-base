# Padrão Template Method

O Padrão Template Method define o esqueleto de um algoritmo em uma classe abstrata, permitindo que subclasses implementem etapas específicas sem alterar a estrutura geral.

## Sumário

- [Padrão Template Method](#padrão-template-method)
  - [Componentes Principais](#componentes-principais)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo em Java 21](#exemplo-em-java-21)
    - [Classe Abstrata](#classe-abstrata)
    - [Classes Concretas](#classes-concretas)
    - [Uso](#uso)

## Componentes Principais

- **Classe Abstrata**: Define o método template e métodos abstratos para etapas específicas.
- **Classes Concretas**: Implementam as etapas específicas do algoritmo.
- **Cliente**: Usa a classe abstrata para executar o algoritmo.

## Benefícios

- Promove reutilização de código comum.
- Define uma estrutura fixa para o algoritmo.
- Facilita extensão por subclasses.
- Segue o princípio aberto/fechado.

## Contras

- Pode limitar flexibilidade do algoritmo.
- Aumenta número de classes com subclasses.
- Requer manutenção para novas etapas.

## Quando Usar

Use o padrão Template Method quando você, como desenvolvedor, precisa definir um processo com etapas fixas em um sistema, como em um e-commerce para processar pedidos (ex.: validar, calcular total, notificar cliente), mas algumas etapas variam dependendo do tipo de pedido (ex.: físico ou digital).

## Exemplo em Java 21

### Classe Abstrata

Define o método template para processar pedidos.

```java
public abstract class ProcessadorPedido {
    // Método template
    public final void processar() {
        validar();
        calcularTotal();
        notificarCliente();
    }

    protected abstract void validar();
    protected abstract void calcularTotal();
    protected void notificarCliente() {
        System.out.println("Notificando cliente sobre o pedido");
    }
}
```

### Classes Concretas

Implementam as etapas específicas para diferentes tipos de pedidos.

```java
public class PedidoFisico extends ProcessadorPedido {
    @Override
    protected void validar() {
        System.out.println("Validando pedido físico: verificando estoque e endereço");
    }

    @Override
    protected void calcularTotal() {
        System.out.println("Calculando total com frete para pedido físico");
    }
}

public class PedidoDigital extends ProcessadorPedido {
    @Override
    protected void validar() {
        System.out.println("Validando pedido digital: verificando licença");
    }

    @Override
    protected void calcularTotal() {
        System.out.println("Calculando total sem frete para pedido digital");
    }
}
```

### Uso

Demonstra o uso do método template para diferentes tipos de pedidos.

```java
public class Main {
    public static void main(String[] args) {
        ProcessadorPedido pedidoFisico = new PedidoFisico();
        pedidoFisico.processar();

        System.out.println();

        ProcessadorPedido pedidoDigital = new PedidoDigital();
        pedidoDigital.processar();
    }
}
```
