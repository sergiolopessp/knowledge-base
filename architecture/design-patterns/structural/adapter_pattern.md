# Padrão Adapter

O Padrão Adapter permite que classes com interfaces incompatíveis trabalhem juntas, convertendo a interface de uma classe em outra esperada pelo cliente.

## Componentes Principais
- **Target**: Interface esperada pelo cliente.
- **Adaptado**: Classe existente com interface incompatível.
- **Adapter**: Classe que converte a interface do adaptado para o target.
- **Cliente**: Usa a interface target para interagir com o adaptado.

## Benefícios
- Permite integração de sistemas incompatíveis.
- Aumenta a reusabilidade de código existente.
- Reduz a necessidade de refatoração de sistemas legados.
- Facilita manutenção e extensibilidade.

## Contras
- Adiciona complexidade com classes extras.
- Pode impactar desempenho devido à camada de adaptação.
- Requer manutenção para mudanças no adaptado.

## Quando Usar
Use o padrão Adapter quando você, como desenvolvedor, precisa integrar uma biblioteca ou sistema legado com uma interface diferente da esperada pelo seu código. Por exemplo, ao integrar um serviço de pagamento antigo que usa métodos diferentes em um sistema de e-commerce moderno que espera uma interface padronizada.

## Exemplo em Java 21

### Interface Target
Define o contrato esperado pelo cliente para processar pagamentos.

```java
public interface ProcessadorPagamento {
    void processarPagamento(double valor);
}
```

### Adaptado
Classe existente (legada) com interface incompatível.

```java
public class SistemaPagamentoLegado {
    public void executarTransacao(String id, double quantia