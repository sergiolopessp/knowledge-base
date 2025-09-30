# Padrão Abstract Factory

O Padrão Abstract Factory fornece uma interface para criar famílias de objetos relacionados ou dependentes sem especificar suas classes concretas. É útil para criar objetos compatíveis entre si.

## Sumário

- [Padrão Abstract Factory](#padrão-abstract-factory)
  - [Componentes Principais](#componentes-principais)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo em Java 21](#exemplo-em-java-21)
    - [Produtos Abstratos](#produtos-abstratos)
    - [Produtos Concretos](#produtos-concretos)
    - [Fábrica Abstrata](#fábrica-abstrata)
    - [Fábricas Concretas](#fábricas-concretas)
    - [Uso](#uso)

## Componentes Principais

- **Fábrica Abstrata**: Interface ou classe abstrata que define métodos para criar produtos.
- **Fábricas Concretas**: Implementam os métodos de criação para uma família específica.
- **Produtos Abstratos**: Interfaces para os tipos de objetos criados.
- **Produtos Concretos**: Implementações específicas dos produtos.

## Benefícios

- Garante compatibilidade entre objetos de uma mesma família.
- Reduz acoplamento entre cliente e classes concretas.
- Facilita a troca de famílias de produtos.
- Promove consistência na criação.

## Contras

- Aumenta o número de classes.
- Pode ser complexo para sistemas simples.
- Requer manutenção para novas famílias ou produtos.

## Quando Usar

Use o padrão Abstract Factory quando você, como desenvolvedor, precisa criar famílias de objetos relacionados em um sistema, como interfaces de usuário (UI) para diferentes plataformas (web, mobile) em um e-commerce. Por exemplo, ao criar botões e caixas de texto que devem ser consistentes dentro de uma plataforma, mas variar entre plataformas.

## Exemplo em Java 21

### Produtos Abstratos

Definem contratos para componentes de UI.

```java
public interface Botao {
    void render();
}

public interface CaixaTexto {
    void render();
}
```

### Produtos Concretos

Implementações específicas para cada plataforma.

```java
public record BotaoWeb(String label) implements Botao {
    @Override
    public void render() {
        System.out.println("Renderizando botão web: " + label);
    }
}

public record CaixaTextoWeb(String placeholder) implements CaixaTexto {
    @Override
    public void render() {
        System.out.println("Renderizando caixa de texto web: " + placeholder);
    }
}

public record BotaoMobile(String label) implements Botao {
    @Override
    public void render() {
        System.out.println("Renderizando botão mobile: " + label);
    }
}

public record CaixaTextoMobile(String placeholder) implements CaixaTexto {
    @Override
    public void render() {
        System.out.println("Renderizando caixa de texto mobile: " + placeholder);
    }
}
```

### Fábrica Abstrata

Define métodos para criar uma família de componentes de UI.

```java
public interface FabricaUI {
    Botao criarBotao(String label);
    CaixaTexto criarCaixaTexto(String placeholder);
}
```

### Fábricas Concretas

Implementam a criação de componentes para uma plataforma específica.

```java
public class FabricaUIWeb implements FabricaUI {
    @Override
    public Botao criarBotao(String label) {
        return new BotaoWeb(label);
    }

    @Override
    public CaixaTexto criarCaixaTexto(String placeholder) {
        return new CaixaTextoWeb(placeholder);
    }
}

public class FabricaUIMobile implements FabricaUI {
    @Override
    public Botao criarBotao(String label) {
        return new BotaoMobile(label);
    }

    @Override
    public CaixaTexto criarCaixaTexto(String placeholder) {
        return new CaixaTextoMobile(placeholder);
    }
}
```

### Uso

Demonstra a criação de componentes de UI para diferentes plataformas usando fábricas concretas.

```java
public class Main {
    public static void main(String[] args) {
        FabricaUI fabricaWeb = new FabricaUIWeb();
        var botaoWeb = fabricaWeb.criarBotao("Enviar");
        var caixaWeb = fabricaWeb.criarCaixaTexto("Digite seu nome");
        botaoWeb.render();
        caixaWeb.render();

        FabricaUI fabricaMobile = new FabricaUIMobile();
        var botaoMobile = fabricaMobile.criarBotao("Enviar");
        var caixaMobile = fabricaMobile.criarCaixaTexto("Digite seu nome");
        botaoMobile.render();
        caixaMobile.render();
    }
}
```
