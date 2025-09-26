# Padrão Singleton

O Padrão Singleton garante que uma classe tenha apenas uma instância e fornece um ponto global de acesso a ela. É usado para controlar o acesso a recursos compartilhados.

## Sumário

- [Padrão Singleton](#padrão-singleton)
  - [Componentes Principais](#componentes-principais)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo em Java 21](#exemplo-em-java-21)
    - [Singleton](#singleton)
    - [Uso](#uso)

## Componentes Principais

- **Singleton**: Classe que restringe a criação de instâncias a uma única e fornece acesso global a ela.

## Benefícios

- Garante uma única instância de uma classe.
- Fornece acesso global controlado.
- Reduz uso de memória ao evitar múltiplas instâncias.
- Útil para gerenciar recursos compartilhados, como conexões de banco de dados.

## Contras

- Pode introduzir acoplamento global.
- Dificulta testes unitários devido ao estado compartilhado.
- Pode ser problemático em ambientes multithread.
- Viola o princípio da responsabilidade única.

## Quando Usar

Use o padrão Singleton quando você, como desenvolvedor, precisa garantir que apenas uma instância de uma classe exista, como em um sistema de e-commerce para gerenciar uma conexão única com um serviço de cache ou banco de dados. Por exemplo, ao controlar o acesso a um gerenciador de configurações globais, onde múltiplas instâncias poderiam causar inconsistências.

## Exemplo em Java 21

### Singleton

A classe `ConfiguracaoSistema` usa inicialização estática (eager initialization) para garantir uma única instância, com métodos para acessar e modificar configurações.

```java
public class ConfiguracaoSistema {
    private static final ConfiguracaoSistema INSTANCE = new ConfiguracaoSistema();
    private String configuracao;

    private ConfiguracaoSistema() {
        this.configuracao = "Configuração padrão";
    }

    public static ConfiguracaoSistema getInstance() {
        return INSTANCE;
    }

    public String getConfiguracao() {
        return configuracao;
    }

    public void setConfiguracao(String configuracao) {
        this.configuracao = configuracao;
    }
}
```

### Uso

Demonstra o acesso à única instância da classe `ConfiguracaoSistema` e a manipulação de suas configurações.

```java
public class Main {
    public static void main(String[] args) {
        var config1 = ConfiguracaoSistema.getInstance();
        System.out.println("Configuração inicial: " + config1.getConfiguracao());

        config1.setConfiguracao("Nova configuração");

        var config2 = ConfiguracaoSistema.getInstance();
        System.out.println("Configuração após alteração: " + config2.getConfiguracao());
    }
}
```
