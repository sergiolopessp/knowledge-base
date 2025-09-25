# Padrão Memento

O Padrão Memento captura e externaliza o estado interno de um objeto sem violar seu encapsulamento, permitindo restaurá-lo posteriormente. É útil para implementar funcionalidades de "desfazer".

## Componentes Principais
- **Originador**: Classe que cria e restaura seu estado a partir de um memento.
- **Memento**: Objeto que armazena o estado do originador.
- **Cuidador**: Gerencia os mementos, sem acessar seu conteúdo.

## Benefícios
- Preserva o encapsulamento do estado.
- Facilita a implementação de desfazer/restaurar.
- Permite salvar múltiplos estados.
- Simplifica o gerenciamento de histórico.

## Contras
- Pode aumentar o consumo de memória com muitos mementos.
- Adiciona complexidade ao design.
- Requer cuidado em sistemas concorrentes.

## Quando Usar
Use o padrão Memento quando você, como desenvolvedor, precisa implementar funcionalidades de desfazer em um sistema, como em um e-commerce onde o usuário pode reverter alterações no carrinho de compras (ex.: remover um item e depois restaurá-lo). É ideal para salvar e restaurar estados sem expor a estrutura interna do objeto.

## Exemplo em Java 21

### Memento
Armazena o estado do carrinho.

```java
public record MementoCarrinho(String produto, int quantidade) {}
```

### Originador
Classe que gerencia o estado do carrinho e cria/restaura mementos.

```java
public class Carrinho {
    private String produto;
    private int quantidade;

    public void adicionarItem(String produto, int quantidade) {
        this.produto = produto;
        this.quantidade = quantidade;
        System.out.printf("Adicionado: %s, Quantidade: %d%n", produto, quantidade);
    }

    public MementoCarrinho salvar() {
        return new MementoCarrinho(produto, quantidade);
    }

    public void restaurar(MementoCarrinho memento) {
        this.produto = memento.produto();
        this.quantidade = memento.quantidade();
        System.out.printf("Restaurado: %s, Quantidade: %d%n", produto, quantidade);
    }
}
```

### Cuidador
Gerencia a lista de mementos para desfazer alterações.

```java
import java.util.ArrayList;
import java.util.List;

public class CuidadorCarrinho {
    private final List<MementoCarrinho> mementos = new ArrayList<>();

    public void adicionarMemento(MementoCarrinho memento) {
        mementos.add(memento);
    }

    public MementoCarrinho getUltimoMemento() {
        if (mementos.isEmpty()) return null;
        var memento = mementos.removeLast();
        return memento;
    }
}
```

### Uso
Demonstra a adição de itens ao carrinho, salvamento do estado e restauração.

```java
public class Main {
    public static void main(String[] args) {
        var carrinho = new Carrinho();
        var cuidador = new CuidadorCarrinho();

        carrinho.adicionarItem("Notebook", 1);
        cuidador.adicionarMemento(carrinho.salvar());

        carrinho.adicionarItem("Smartphone", 2);
        cuidador.adicionarMemento(carrinho.salvar());

        carrinho.restaurar(cuidador.getUltimoMemento()); // Restaura para Notebook
        carrinho.restaurar(cuidador.getUltimoMemento()); // Restaura para estado inicial
    }
}
```