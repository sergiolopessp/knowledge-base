# Padrão Observer

O Padrão Observer define uma dependência um-para-muitos entre objetos, onde um objeto (sujeito) notifica automaticamente seus observadores sobre mudanças em seu estado.

## Sumário

- [Padrão Observer](#padrão-observer)
  - [Componentes Principais](#componentes-principais)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo em Java 21](#exemplo-em-java-21)
    - [Observador](#observador)
    - [Sujeito](#sujeito)
    - [Sujeito Concreto](#sujeito-concreto)
    - [Observador Concreto](#observador-concreto)
    - [Uso](#uso)

## Componentes Principais

- **Sujeito**: Interface ou classe que mantém a lista de observadores e notifica mudanças.
- **Sujeito Concreto**: Implementa o sujeito e gerencia o estado.
- **Observador**: Interface que define o método de atualização.
- **Observador Concreto**: Implementa a lógica de resposta às mudanças.

## Benefícios

- Promove comunicação desacoplada entre sujeito e observadores.
- Permite adicionar/remover observadores dinamicamente.
- Suporta notificações automáticas de mudanças.
- Facilita extensibilidade para novos observadores.

## Contras

- Pode causar vazamentos de memória se observadores não forem removidos.
- Notificações frequentes podem impactar desempenho.
- Aumenta complexidade em sistemas com muitos observadores.

## Quando Usar

Use o padrão Observer quando você, como desenvolvedor, precisa notificar múltiplos componentes sobre mudanças de estado em um sistema, como em um e-commerce onde atualizações no estoque de produtos devem refletir automaticamente na interface do usuário ou em sistemas de alerta (ex.: notificar clientes sobre disponibilidade).

## Exemplo em Java 21

### Observador

Define a interface para atualização dos observadores.

```java
public interface Observador {
    void atualizar(String produto, int estoque);
}
```

### Sujeito

Define a interface para gerenciar observadores e notificações.

```java
public interface SujeitoEstoque {
    void adicionarObservador(Observador observador);
    void removerObservador(Observador observador);
    void notificarObservadores();
}
```

### Sujeito Concreto

Gerencia o estoque e notifica observadores sobre mudanças.

```java
import java.util.ArrayList;
import java.util.List;

public class Estoque implements SujeitoEstoque {
    private final List<Observador> observadores = new ArrayList<>();
    private String produto;
    private int quantidade;

    public void setEstoque(String produto, int quantidade) {
        this.produto = produto;
        this.quantidade = quantidade;
        notificarObservadores();
    }

    @Override
    public void adicionarObservador(Observador observador) {
        observadores.add(observador);
    }

    @Override
    public void removerObservador(Observador observador) {
        observadores.remove(observador);
    }

    @Override
    public void notificarObservadores() {
        for (Observador observador : observadores) {
            observador.atualizar(produto, quantidade);
        }
    }
}
```

### Observador Concreto

Implementa a lógica de resposta às mudanças no estoque.

```java
public class InterfaceUsuario implements Observador {
    private final String nome;

    public InterfaceUsuario(String nome) {
        this.nome = nome;
    }

    @Override
    public void atualizar(String produto, int estoque) {
        System.out.printf("%s: Estoque atualizado - Produto: %s, Quantidade: %d%n", nome, produto, estoque);
    }
}
```

### Uso

Demonstra a notificação de observadores quando o estoque muda.

```java
public class Main {
    public static void main(String[] args) {
        var estoque = new Estoque();
        var ui1 = new InterfaceUsuario("Interface Web");
        var ui2 = new InterfaceUsuario("Interface Mobile");

        estoque.adicionarObservador(ui1);
        estoque.adicionarObservador(ui2);

        estoque.setEstoque("Notebook", 10);
        estoque.setEstoque("Smartphone", 5);

        estoque.removerObservador(ui2);
        estoque.setEstoque("Tablet", 3);
    }
}
```
