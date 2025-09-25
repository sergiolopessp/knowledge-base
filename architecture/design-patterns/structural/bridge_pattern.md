# Padrão Bridge

O Padrão Bridge desacopla uma abstração de sua implementação, permitindo que ambas variem independentemente. É útil para evitar explosão de classes em hierarquias complexas.

## Componentes Principais
- **Abstração**: Define a interface de alto nível para o cliente.
- **Abstração Refinada**: Estende a abstração com funcionalidades específicas.
- **Implementador**: Interface para a implementação concreta.
- **Implementações Concretas**: Classes que implementam a interface do implementador.

## Benefícios
- Separa abstração da implementação, permitindo mudanças independentes.
- Reduz acoplamento entre componentes.
- Facilita adição de novas abstrações ou implementações.
- Evita explosão de classes em hierarquias.

## Contras
- Aumenta complexidade com classes adicionais.
- Pode ser desnecessário para sistemas simples.
- Requer planejamento para definir interfaces corretamente.

## Quando Usar
Use o padrão Bridge quando você, como desenvolvedor, precisa separar a lógica de alto nível da implementação em um sistema, como em um e-commerce onde diferentes tipos de carrinhos de compras (ex.: padrão, premium) podem usar diferentes métodos de armazenamento (ex.: banco de dados, cache). Isso permite adicionar novos carrinhos ou métodos de armazenamento sem modificar as classes existentes.

## Exemplo em Java 21

### Implementador
Define a interface para os métodos de armazenamento.

```java
public interface Armazenamento {
    void salvar(String dados);
}
```

### Implementações Concretas
Implementações específicas para diferentes métodos de armazenamento.

```java
public class ArmazenamentoBanco implements Armazenamento {
    @Override
    public void salvar(String dados) {
        System.out.println("Salvando no banco de dados: " + dados);
    }
}

public class ArmazenamentoCache implements Armazenamento {
    @Override
    public void salvar(String dados) {
        System.out.println("Salvando no cache: " + dados);
    }
}
```

### Abstração
Define a interface de alto nível para o carrinho de compras.

```java
public abstract class CarrinhoCompras {
    protected final Armazenamento armazenamento;

    protected CarrinhoCompras(Armazenamento armazenamento) {
        this.armazenamento = armazenamento;
    }

    abstract void adicionarItem(String item);
}
```

### Abstração Refinada
Estende a abstração para tipos específicos de carrinhos.

```java
public class CarrinhoPadrao extends CarrinhoCompras {
    public CarrinhoPadrao(Armazenamento armazenamento) {
        super(armazenamento);
    }

    @Override
    public void adicionarItem(String item) {
        armazenamento.salvar("Item padrão: " + item);
    }
}

public class CarrinhoPremium extends CarrinhoCompras {
    public CarrinhoPremium(Armazenamento armazenamento) {
        super(armazenamento);
    }

    @Override
    public void adicionarItem(String item) {
        armazenamento.salvar("Item premium: " + item);
    }
}
```

### Uso
Demonstra como diferentes carrinhos usam diferentes métodos de armazenamento.

```java
public class Main {
    public static void main(String[] args) {
        Armazenamento banco = new ArmazenamentoBanco();
        Armazenamento cache = new ArmazenamentoCache();

        CarrinhoCompras carrinhoPadrao = new CarrinhoPadrao(banco);
        carrinhoPadrao.adicionarItem("Notebook");

        CarrinhoCompras carrinhoPremium = new CarrinhoPremium(cache);
        carrinhoPremium.adicionarItem("Smartphone");
    }
}
```