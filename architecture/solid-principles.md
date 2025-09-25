# Princípios SOLID: Guia prático para código melhor

_Resumo inspirado em “Understanding the SOLID Principles: A Practical Guide for Better Code”_ ([dev.to](https://dev.to/aranka_maxilo_55bab8ad3c3/understanding-the-solid-principles-a-practical-guide-for-better-code-c7))

---

## O que são os princípios SOLID

Os princípios SOLID são cinco regras/orientações para design de software orientado a objetos, popularizados por Robert C. Martin (“Uncle Bob”). Eles ajudam a manter o código:

- limpo  
- manutenível  
- extensível  
- menos sujeito a bugs ao evoluir  

SOLID = **S**ingle Responsibility, **O**pen/Closed, **L**iskov Substitution, **I**nterface Segregation, **D**ependency Inversion.

---

## Detalhamento dos princípios

| Princípio | Definição | Exemplos / impacto prático |
|-----------|-----------|------------------------------|
| **Single Responsibility Principle (SRP)** | Uma classe/módulo deve ter **uma única razão para mudar**. Deve ter apenas uma responsabilidade. | Evita que mudanças em funcionalidades diferentes causem efeitos colaterais. Facilita testes e manutenção. |
| **Open/Closed Principle (OCP)** | Entidades de software devem estar **abertas para extensão, mas fechadas para modificação**. | Permite adicionar novos comportamentos sem mexer no código existente, reduz risco de quebrar algo que já funciona. |
| **Liskov Substitution Principle (LSP)** | Subclasses devem poder substituir suas superclasses sem alterar correção do programa. | Se uma subclasse falha em se comportar como a classe base, pode introduzir bugs. Ex: herança com métodos “inúteis” ou comportamento inesperado. |
| **Interface Segregation Principle (ISP)** | Clientes não devem ser forçados a depender de interfaces que não usam. Interfaces específicas em vez de genéricas demais. | Torna interfaces menores, mais coesas, evita classes que herdam métodos que não fazem sentido para elas. |
| **Dependency Inversion Principle (DIP)** | Depender de abstrações, não de implementações concretas. Módulos de alto nível não devem depender de módulos de baixo; ambos devem depender de abstrações. | Facilita trocar implementações, injeção de dependências, testar com mocks, desacoplamento. |

---

## Exemplos práticos (Java / Go / C++)

```java
// Exemplo Java: SRP & DIP
public interface NotificationService {
    void send(String message);
}

public class EmailNotificationService implements NotificationService {
    @Override
    public void send(String message) {
        // lógica de envio por e‑mail
    }
}

public class NotificationController {
    private final NotificationService service;

    public NotificationController(NotificationService service) {
        this.service = service;
    }

    public void notifyUser(String msg) {
        service.send(msg);
    }
}
```

```go
// Exemplo Go: ISP & OCP
type Worker interface {
    Work()
}

type Eater interface {
    Eat()
}

type HumanWorker struct {}
func (h HumanWorker) Work() { /*...*/ }
func (h HumanWorker) Eat() { /*...*/ }

type RobotWorker struct {}
func (r RobotWorker) Work() { /*...*/ }
// RobotWorker não implementa Eat porque não come — Interface segregation

// Para OCP: adicionar novo tipo de worker sem alterar os já existentes
```

```cpp
// Exemplo C++: Liskov Substitution & DIP
class Shape {
public:
    virtual double area() const = 0;
    virtual ~Shape() = default;
};

class Rectangle : public Shape {
    double width, height;
public:
    Rectangle(double w, double h): width(w), height(h) {}
    double area() const override { return width * height; }
};

class Circle : public Shape {
    double radius;
public:
    Circle(double r): radius(r) {}
    double area() const override { return 3.1415 * radius * radius; }
};

void printArea(const Shape& s) {
    std::cout << "Area: " << s.area() << std::endl;
}
```

---

## Boas práticas e armadilhas comuns

- Não abuse de abstrações: abstrações sem necessidade real podem complicar código.  
- Se seu design muda muito, talvez as responsabilidades não estejam bem definidas.  
- Evite herança quando composição resolve melhor (herança forçada pode violar LSP).  
- Consistência: aplique os princípios de forma combinada; isoladamente já ajudam, mas combinados é que resultam em arquitetura robusta.  
- Escrever testes unitários ajuda muito a detectar violações (por ex: se uma subclasse não satisfaz comportamento esperado da classe base).  

---

## Considerações finais

- Os princípios SOLID servem como guia, **não regras rígidas**; há casos onde uma regra pode ser violada por razões válidas (legado, performance, restrições específicas).  
- Avalie trade‑offs: simplicidade vs abstração, rapidez vs estrutura.  
- Documente seu estilo de design no projeto, compartilhe com equipe.

---

## Recursos & leitura adicional

- Artigo original: *Understanding the SOLID Principles: A Practical Guide for Better Code* ([dev.to](https://dev.to/aranka_maxilo_55bab8ad3c3/understanding-the-solid-principles-a-practical-guide-for-better-code-c7))  
- “Clean Code”, Robert C. Martin  
- Exemplos de SOLID em frameworks Java, Go, C++  
- Exercícios / refatorações de código legado para aplicar SOLID  
