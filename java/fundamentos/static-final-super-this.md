# Understanding `static`, `final`, `super`, `super()`, `this()` e `this` em Java

Este artigo é um resumo comentado do post “Understanding static, final, super, super(), this() and this” do **DEV Community**.  
Ele aborda como essas palavras‑chave funcionam em Java, como usá-las corretamente, e exemplos práticos. ([dev.to](https://dev.to/masteringbackend/understanding-static-final-super-super-this-and-this-lco))

---

## 1. `final`

- Pode ser usado com **classes**, **métodos** ou **variáveis**.  
- **Variável** final: seu valor só pode ser atribuído uma vez (imutável depois).  
- **Classe** final: não pode ser estendida (não admite subclasses).  
- **Método** final: não pode ser sobrescrito por subclasses.  
- Para variáveis locais, pode-se declarar e depois inicializar (mas apenas uma vez).  

### Exemplo

```java
final class FinalClass {
    final int finalVariable = 100;

    final void finalMethod(final int param) {
        System.out.println("Inside finalMethod");
        System.out.println("Final variable value: " + finalVariable);
        System.out.println("Final parameter value: " + param);
    }
}
```

---

## 2. `static`

- Aplica-se a variáveis, métodos, blocos e classes internas.  
- Membros `static` pertencem à classe, não à instância. Há apenas uma cópia para toda a classe.  
- Pode-se acessar métodos ou variáveis estáticas diretamente via `NomeDaClasse.membro` sem instanciar.  
- Blocos estáticos (`static { … }`) são executados no carregamento da classe.  

### Exemplo

```java
public class BankAccount {
    static double interestRate;
    static int totalAccounts = 0;

    static {
        System.out.println("Bank system initialized.");
        interestRate = 4.5;
    }

    public BankAccount() {
        totalAccounts++;
    }

    public static void showTotalAccounts() {
        System.out.println("Total Bank Accounts: " + totalAccounts);
    }
}
```

---

## 3. `this`

- `this` é a referência para o objeto corrente.  
- Usado para diferenciar entre variáveis de instância e parâmetros com o mesmo nome.  
- Também pode invocar outro método do mesmo objeto: `this.metodo()`.  

### Exemplo

```java
public class Student {
    String name;
    int age;

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void displayInfo() {
        System.out.println("Name: " + this.name);
        System.out.println("Age: " + this.age);
    }
}
```

---

## 4. `this()`

- Usado dentro de construtores para chamar outro construtor da **mesma classe**.  
- Deve ser a **primeira linha** no construtor.  

### Exemplo

```java
public class Student {
    String name;
    int age;

    public Student() {
        this("Unknown", 0);
        System.out.println("Default constructor called");
    }

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
        System.out.println("Parameterized constructor called");
    }
}
```

---

## 5. `super`

- `super` referencia o objeto da **classe pai imediata**.  
- Pode chamar métodos ou acessar variáveis do super‑tipo: `super.metodo()` ou `super.variavel`.  

### Exemplo

```java
class Parent {
    void show() {
        System.out.println("Parent method");
    }
}

class Child extends Parent {
    void show() {
        System.out.println("Child method");
    }

    void display() {
        super.show();  // chama o método da classe pai
        show();        // chama o método da classe atual
    }
}
```

---

## 6. `super()`

- Usado para chamar o construtor da **classe pai** dentro de um construtor da subclasse.  
- Deve ser a **primeira instrução** no construtor da subclasse (se usado).  

### Exemplo

```java
class Parent {
    Parent(String name) {
        System.out.println("Parent constructor with name: " + name);
    }
}

class Child extends Parent {
    Child() {
        super("Ayush");
        System.out.println("Child constructor");
    }
}
```

---

## 7. Encadeamento de construtores (Constructor chaining)

- Usando `this()` dentro da mesma classe ou `super()` para subir até a classe pai, os construtores podem chamar uns aos outros em cadeia.  
- Exemplo de encadeamento múltiplo:

```java
public class Student {
    public Student() {
        this("Ayush");
        System.out.println("Constructor 1");
    }

    public Student(String name) {
        this(name, 22);
        System.out.println("Constructor 2: Name = " + name);
    }

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
        System.out.println("Constructor 3: Name = " + name + ", Age = " + age);
    }
}
```

---

## ✅ Dicas de uso e boas práticas

- Prefira **final** para variáveis que não devem mudar, reforçando imutabilidade quando possível.  
- Use `static` com cautela: força acoplamento global.  
- Evite ambiguidade de nomes para não abusar de `this`.  
- Use `this()` ou `super()` para evitar duplicação entre construtores.  
- Sempre que usar `super()` ou `this()`, eles devem estar **na primeira linha** do construtor correspondente.

---

## Referências

- “Understanding static, final, super, super(), this() and this” — DEV Community ([dev.to](https://dev.to/masteringbackend/understanding-static-final-super-super-this-and-this-lco))  
