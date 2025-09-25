# Padrão Proxy

O Padrão Proxy atua como um intermediário para outro objeto, controlando acesso, adicionando funcionalidades ou otimizando interações, sem alterar a interface original.

## Componentes Principais
- **Subject**: Interface que define o contrato para o proxy e o objeto real.
- **Objeto Real**: Classe que implementa a funcionalidade principal.
- **Proxy**: Classe que implementa a interface Subject, controlando acesso ao objeto real.
- **Cliente**: Interage com o proxy como se fosse o objeto real.

## Benefícios
- Controla acesso ao objeto real (ex.: segurança, lazy loading).
- Permite adicionar funcionalidades (ex.: logging, caching).
- Reduz custo de operações pesadas.
- Mantém a interface original.

## Contras
- Adiciona uma camada extra, aumentando complexidade.
- Pode introduzir latência em chamadas.
- Requer manutenção para sincronizar com o objeto real.

## Quando Usar
Use o padrão Proxy quando você, como desenvolvedor, precisa controlar ou otimizar o acesso a um recurso caro em um sistema, como em um e-commerce para gerenciar o carregamento de imagens de produtos sob demanda (lazy loading) ou adicionar autenticação antes de acessar um serviço de pagamento.

## Exemplo em Java 21

### Subject
Define o contrato para carregar e exibir imagens.

```java
public interface Imagem {
    void exibir();
}
```

### Objeto Real
Classe que carrega uma imagem do disco, operação potencialmente custosa.

```java
public class ImagemReal implements Imagem {
    private final String nomeArquivo;

    public ImagemReal(String nomeArquivo) {
        this.nomeArquivo = nomeArquivo;
        carregarDoDisco();
    }

    private void carregarDoDisco() {
        System.out.println("Carregando imagem do disco: " + nomeArquivo);
    }

    @Override
    public void exibir() {
        System.out.println("Exibindo imagem: " + nomeArquivo);
    }
}
```

### Proxy
Controla o acesso à `ImagemReal`, implementando lazy loading.

```java
public class ProxyImagem implements Imagem {
    private ImagemReal imagemReal;
    private final String nomeArquivo;

    public ProxyImagem(String nomeArquivo) {
        this.nomeArquivo = nomeArquivo;
    }

    @Override
    public void exibir() {
        if (imagemReal == null) {
            imagemReal = new ImagemReal(nomeArquivo);
        }
        imagemReal.exibir();
    }
}
```

### Uso
Demonstra o uso do proxy para carregar imagens sob demanda.

```java
public class Main {
    public static void main(String[] args) {
        Imagem imagem = new ProxyImagem("foto_produto.jpg");
        // Imagem não é carregada até ser exibida
        System.out.println("Imagem criada, mas não carregada.");
        imagem.exibir(); // Carrega e exibe a imagem
        imagem.exibir(); // Reutiliza a imagem já carregada
    }
}
```