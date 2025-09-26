# Padrão Command

O Padrão Command encapsula uma solicitação como um objeto, permitindo parametrizar clientes com diferentes solicitações, enfileirar ou registrar operações e suportar desfazer.

## Sumário

- [Padrão Command](#padrão-command)
- [Componentes Principais](#componentes-principais)
- [Benefícios](#benefícios)
- [Contras](#contras)
- [Quando Usar](#quando-usar)
- [Exemplo em Java 21](#exemplo-em-java-21)
  - [Command](#command)
  - [Receiver](#receiver)
  - [Comando Concreto](#comando-concreto)
  - [Invoker](#invoker)
  - [Uso](#uso)

## Componentes Principais

- **Command**: Interface que define o método para executar a solicitação.
- **Comando Concreto**: Implementa a lógica específica da solicitação.
- **Invoker**: Solicita a execução do comando.
- **Receiver**: Classe que realiza a ação associada ao comando.
- **Cliente**: Cria e configura os comandos.

## Benefícios

- Desacopla o solicitante da execução.
- Permite enfileirar, registrar ou desfazer operações.
- Facilita extensão com novos comandos.
- Suporta composição de comandos complexos.

## Contras

- Aumenta o número de classes.
- Pode adicionar complexidade em sistemas simples.
- Gerenciamento de desfazer pode ser complicado.

## Quando Usar

Use o padrão Command quando você, como desenvolvedor, precisa desacoplar a lógica de execução de ações em um sistema, como em um e-commerce para gerenciar operações como adicionar produtos ao carrinho ou processar pagamentos. Por exemplo, ao implementar um sistema de checkout onde ações podem ser enfileiradas, registradas ou desfeitas (ex.: cancelar um item).

## Exemplo em Java 21

### Command

Define a interface para executar e desfazer comandos.

```java
public interface Comando {
    void executar();
    void desfazer();
}
```

### Receiver

Classe que contém a lógica de negócios para manipular o carrinho.

```java
public class Carrinho {
    public void adicionarProduto(String produto, int quantidade) {
        System.out.printf("Adicionado %d %s ao carrinho%n", quantidade, produto);
    }

    public void removerProduto(String produto, int quantidade) {
        System.out.printf("Removido %d %s do carrinho%n", quantidade, produto);
    }
}
```

### Comando Concreto

Implementa a adição de produtos ao carrinho com suporte a desfazer.

```java
public class ComandoAdicionarProduto implements Comando {
    private final Carrinho carrinho;
    private final String produto;
    private final int quantidade;

    public ComandoAdicionarProduto(Carrinho carrinho, String produto, int quantidade) {
        this.carrinho = carrinho;
        this.produto = produto;
        this.quantidade = quantidade;
    }

    @Override
    public void executar() {
        carrinho.adicionarProduto(produto, quantidade);
    }

    @Override
    public void desfazer() {
        carrinho.removerProduto(produto, quantidade);
    }
}
```

### Invoker

Gerencia a execução e o desfazer de comandos.

```java
import java.util.ArrayList;
import java.util.List;

public class GerenciadorCarrinho {
    private final List<Comand> comandos = new ArrayList<>();

    public void executarComando(Comando comando) {
        comandos.add(comando);
        comando.executar();
    }

    public void desfazerUltimo() {
        if (!comandos.isEmpty()) {
            var comando = comandos.removeLast();
            comando.desfazer();
        }
    }
}
```

### Uso

Demonstra a adição de produtos ao carrinho com a possibilidade de desfazer
