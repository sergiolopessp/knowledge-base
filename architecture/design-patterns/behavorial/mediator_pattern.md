# Padrão Mediator

O Padrão Mediator promove o desacoplamento entre objetos, centralizando a comunicação em um mediador que gerencia interações entre componentes, evitando acoplamento direto.

## Componentes Principais
- **Mediator**: Interface ou classe abstrata que define métodos para comunicação.
- **Mediator Concreto**: Implementa a lógica de coordenação entre componentes.
- **Colaborador**: Classes que interagem por meio do mediador.
- **Cliente**: Configura o mediador e os colaboradores.

## Benefícios
- Reduz acoplamento entre componentes.
- Centraliza a lógica de comunicação.
- Facilita manutenção e adição de novos componentes.
- Simplifica interações complexas.

## Contras
- O mediador pode se tornar excessivamente complexo.
- Introduz um ponto único de falha.
- Pode impactar desempenho em sistemas grandes.

## Quando Usar
Use o padrão Mediator quando você, como desenvolvedor, precisa gerenciar interações complexas entre múltiplos objetos em um sistema, como em um e-commerce onde componentes como carrinho, estoque e pagamento precisam se comunicar. Por exemplo, ao atualizar o estoque quando um item é adicionado ao carrinho, o mediador coordena as ações sem que os componentes se comuniquem diretamente.

## Exemplo em Java 21

### Mediator
Define a interface para comunicação entre colaboradores.

```java
public interface MediatorLoja {
    void notificar(String evento, Object colaborador);
}
```

### Mediator Concreto
Coordena as interações entre carrinho, estoque e pagamento.

```java
public class MediatorLojaConcreto implements MediatorLoja {
    private final Carrinho carrinho;
    private final Estoque estoque;
    private final Pagamento pagamento;

    public MediatorLojaConcreto(Carrinho carrinho, Estoque estoque, Pagamento pagamento) {
        this.carrinho = carrinho;
        this.estoque = estoque;
        this.pagamento = pagamento;
        this.carrinho.setMediator(this);
        this.estoque.setMediator(this);
        this.pagamento.setMediator(this);
    }

    @Override
    public void notificar(String evento, Object colaborador) {
        if (evento.equals("adicionarItem") && colaborador == carrinho) {
            estoque.verificarDisponibilidade();
        } else if (evento.equals("estoqueConfirmado") && colaborador == estoque) {
            pagamento.processar();
        } else if (evento.equals("pagamentoConcluido") && colaborador == pagamento) {
            carrinho.confirmar();
        }
    }
}
```

### Colaboradores
Classes que interagem por meio do mediador.

```java
public class Carrinho {
    private MediatorLoja mediator;

    public void setMediator(MediatorLoja mediator) {
        this.mediator = mediator;
    }

    public void adicionarItem(String produto) {
        System.out.println("Adicionando " + produto + " ao carrinho");
        mediator.notificar("adicionarItem", this);
    }

    public void confirmar() {
        System.out.println("Carrinho confirmado");
    }
}

public class Estoque {
    private MediatorLoja mediator;

    public void setMediator(MediatorLoja mediator) {
        this.mediator = mediator;
    }

    public void verificarDisponibilidade() {
        System.out.println("Estoque verificado: produto disponível");
        mediator.notificar("estoqueConfirmado", this);
    }
}

public class Pagamento {
    private MediatorLoja mediator;

    public void setMediator(MediatorLoja mediator) {
        this.mediator = mediator;
    }

    public void processar() {
        System.out.println("Pagamento processado");
        mediator.notificar("pagamentoConcluido", this);
    }
}
```

### Uso
Demonstra como o mediador coordena a adição de um item ao carrinho.

```java
public class Main {
    public static void main(String[] args) {
        var carrinho = new Carrinho();
        var estoque = new Estoque();
        var pagamento = new Pagamento();

        var mediator = new MediatorLojaConcreto(carrinho, estoque, pagamento);

        carrinho.adicionarItem("Notebook");
    }
}
```