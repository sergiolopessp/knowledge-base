# Java Annotations Demystified

As anotações em Java são metadados que fornecem dados sobre um programa, mas não fazem parte do programa em si. Elas não têm efeito direto na operação do código que anotam, servindo como "etiquetas" para o compilador, ferramentas de build ou para o runtime (via Reflection).

**Por que usar?**

Redução de Boilerplate: Substituem configurações complexas em XML (comum em versões antigas de Java EE).

Checagens de Compilação: Permitem que o compilador detecte erros precocemente (ex: @Override).

Mágica em Runtime: Essenciais para frameworks como Spring (DI) e Hibernate (ORM).

**Anotações Built-in Comuns**

@Override: Garante que você está de fato sobrescrevendo um método da superclasse.

@Deprecated: Marca código que não deve mais ser usado.

@SuppressWarnings: Silencia avisos específicos do compilador.

**Criando uma Anotação Customizada**

Para criar sua própria anotação, utilizamos as Meta-anotações:

@Target: Define onde a anotação pode ser aplicada (METHOD, TYPE, FIELD, etc).

@Retention: Define até quando a anotação persiste:

SOURCE: Apenas no código fonte (descartada pelo compilador).

CLASS: Mantida no .class, mas ignorada pela JVM.

RUNTIME: Disponível em tempo de execução via Reflection (a mais usada em frameworks).

Exemplo: @LogExecutionTime
```Java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecutionTime {
    String value() default "Method";
}
```
**Processando via Reflection (The "Magic Sauce")**

A anotação por si só é inerte. O poder vem do processador que a lê.

```Java

public void process(Object obj) throws Exception {
    for (Method method : obj.getClass().getDeclaredMethods()) {
        if (method.isAnnotationPresent(LogExecutionTime.class)) {
            LogExecutionTime annotation = method.getAnnotation(LogExecutionTime.class);
            
            long start = System.currentTimeMillis();
            method.invoke(obj);
            long end = System.currentTimeMillis();
            
            System.out.println("[" + annotation.value() + "] Took: " + (end - start) + "ms");
        }
    }
}
```
**Casos de Uso Reais**

Spring Boot: @RestController, @Autowired, @Service.

Lombok: @Data, @Getter, @Setter (manipulação de bytecode em tempo de compilação).

JPA/Hibernate: @Entity, @Table, @Id.

JUnit: @Test, @BeforeEach.

**Boas Práticas**

Use as built-in primeiro: Não reinvente a roda se @Deprecated ou @Override resolvem.

Seja descritivo: Nomes de anotações devem ser claros sobre seu propósito.

Cuidado com a performance: O processamento em RUNTIME via Reflection tem um custo (geralmente mitigado por caching nos frameworks, mas relevante em sistemas de ultra baixa latência).

Documente: Sempre explique o que o processador da anotação espera encontrar.