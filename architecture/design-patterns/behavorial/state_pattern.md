# Padrão State

O Padrão State permite que um objeto altere seu comportamento quando seu estado interno muda, encapsulando estados como objetos distintos.

## Componentes Principais
- **Contexto**: Classe que mantém uma referência ao estado atual.
- **Estado**: Interface que define o comportamento associado a um estado.
- **Estados Concretos**: Classes que implementam comportamentos específicos para cada estado.

## Benefícios
- Encapsula comportamentos específicos de cada estado.
- Facilita adição de novos estados sem modificar o contexto.
- Elimina condicionais complexas baseadas em estado.
- Promove o princípio aberto/fechado.

## Contras
- Aumenta o número de classes.
- Pode ser desnecessário para sistemas com poucos estados.
- Requer manutenção para novos estados.

## Quando Usar
Use o padrão State quando você, como desenvolvedor, precisa gerenciar comportamentos que mudam com base no estado de um objeto em um sistema, como em um e-commerce onde um pedido pode estar em estados como "pendente", "pago" ou "enviado". Cada estado define ações específicas (ex.: cancelar é permitido apenas em "pendente").

## Exemplo em Java 21

### Estado
Define a interface para ações baseadas no estado do pedido.

```java
public interface EstadoPedido {
    void processar(Pedido pedido);
    void cancelar(Pedido pedido);
}
```

### Contexto
Classe que mantém o estado atual do pedido.

```java
public class Pedido {
    private EstadoPedido estado;

    public Pedido() {
        this.estado = new EstadoPendente();
    }

    public void setEstado(EstadoPedido estado) {
        this.estado = estado;
    }

    public void processar() {
        estado.processar(this);
    }

    public void cancelar() {
        estado.cancelar(this);
    }
}
```

### Estados Concretos
Implementam comportamentos para cada estado do pedido.

```java
public class EstadoPendente implements EstadoPedido {
    @Override
    public void processar(Pedido pedido) {
        System.out.println("Processando pedido: movendo para pago");
        pedido.setEstado(new EstadoPago());
    }

    @Override
    public void cancelar(Pedido pedido) {
        System.out.println("Cancelando pedido: movendo para cancelado");
        pedido.setEstado(new EstadoCancelado());
    }
}

public class EstadoPago implements EstadoPedido {
    @Override
    public void processar(Pedido pedido) {
        System.out.println("Processando pedido: movendo para enviado");
        pedido.setEstado(new EstadoEnviado());
    }

    @Override
    public void cancelar(Pedido pedido) {
        System.out.println("Não é possível cancelar: pedido já pago");
    }
}

public class EstadoEnviado implements EstadoPedido {
    @Override
    public void processar(Pedido pedido) {
        System.out.println("Pedido já enviado: nenhuma ação adicional");
    }

    @Override
    public void cancelar(Pedido pedido) {
        System.out.println("Não é possível cancelar: pedido já enviado");
    }
}

public class EstadoCancelado implements EstadoPedido {
    @Override
    public void processar(Pedido pedido) {
        System.out.println("Não é possível processar: pedido cancelado");
    }

    @Override
    public void cancelar(Pedido pedido) {
        System.out.println("Pedido já cancelado");
    }
}
```

### Uso
Demonstra a mudança de comportamento do pedido com base em seu estado.

```java
public class Main {
    public static void main(String[] args) {
        var pedido = new Pedido();

        pedido.processar(); // Move para pago
        pedido.processar(); // Move para enviado
        pedido.cancelar(); // Não permitido: enviado

        var pedido2