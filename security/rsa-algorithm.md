# Como funciona o RSA (Para Engenheiros e Curiosos)

**Tags:** #algorithm #security #cryptography #math #cs-fundamentals

O RSA (*Rivest-Shamir-Adleman*) é a base da criptografia assimétrica moderna. Como engenheiros de software, nós o utilizamos diariamente — seja no handshake SSL/TLS de nossos Load Balancers na nuvem, na autenticação SSH ao acessar servidores Linux ou assinando imagens Docker.

Embora em linguagens como Java (`java.security`) ou Go (`crypto/rsa`) isso seja abstraído, entender a matemática por trás é crucial para compreender a complexidade computacional e a segurança de nossas aplicações.

## O Conceito Básico

Diferente da criptografia simétrica (que usa a mesma chave para cifrar e decifrar), o RSA é **assimétrico**:
* **Chave Pública:** Encripta a mensagem (pode ser compartilhada com qualquer um).
* **Chave Privada:** Decripta a mensagem (DEVE ser mantida em segredo absoluto).

A mágica é que elas são **comutativas matematicamente**, mas computacionalmente inviáveis de serem revertidas sem a chave privada.

## A Matemática (Simplificada)

Para entender o RSA, precisamos de 4 conceitos fundamentais:

1.  **Números Primos:** Divisíveis apenas por 1 e por eles mesmos (ex: 2, 3, 5, 7, 11...).
2.  **Números Semiprimos:** O produto de dois números primos (ex: $21 = 3 \times 7$).
3.  **Fatoração:** O processo inverso. Dado 21, descobrir que os fatores originais são 3 e 7.
4.  **Módulo (Mod):** O resto da divisão (ex: $7 \mod 3 = 1$).

> **O Segredo do RSA:** É muito fácil multiplicar dois números primos gigantescos para obter um número semiprimo. Porém, é **computacionalmente "impossível"** (com a tecnologia atual) pegar esse número gigante e descobrir quais eram os dois primos originais (fatoração).

## O Algoritmo Passo-a-Passo

### 1. Gerando as Chaves

1.  **Escolha dois números primos ($p$ e $q$):**
    * Exemplo: $p = 7$, $q = 19$.

2.  **Calcule o produto ($N$):**
    * $N = p \times q \Rightarrow 7 \times 19 = 133$.
    * *$N$ será parte da Chave Pública.*

3.  **Calcule o Totiente de Euler ($T$):**
    * Como $p$ e $q$ são primos: $T = (p - 1) \times (q - 1)$.
    * $T = (7 - 1) \times (19 - 1) = 6 \times 18 = 108$.

4.  **Escolha a Chave Pública ($E$):**
    * $E$ deve ser coprimo de $T$ (sem fatores comuns além de 1) e $1 < E < T$.
    * Vamos escolher **29** (pois é primo e não divide 108).
    * **Chave Pública:** $(E, N) \Rightarrow (29, 133)$.

5.  **Calcule a Chave Privada ($D$):**
    * $D$ deve ser o inverso modular de $E$ base $T$.
    * Fórmula: $(D \times E) \mod T = 1$.
    * Calculando: Para $E=29$ e $T=108$, $D = 41$.
    * Prova: $(41 \times 29) = 1189$. E $1189 \div 108$ deixa resto **1**.
    * **Chave Privada:** $(D, N) \Rightarrow (41, 133)$.

### 2. Criptografando e Descriptografando

Imagine que a mensagem seja o número **60**.

#### Encriptação (Usando a Chave Pública $E$)
A fórmula é: $Cifra = Mensagem^E \mod N$

$$
60^{29} \mod 133 = \mathbf{72}
$$

A mensagem cifrada é **72**.

#### Decriptação (Usando a Chave Privada $D$)
A fórmula é: $Mensagem = Cifra^D \mod N$

$$
72^{41} \mod 133 = \mathbf{60}
$$

Recuperamos a mensagem original: **60**.

---

## Por que é seguro?

A segurança depende inteiramente da dificuldade de fatorar $N$.
No nosso exemplo, fatorar 133 é trivial (dá para fazer de cabeça: 7 e 19).

Mas no mundo real (certificados SSL, chaves SSH), usamos chaves de **2048 bits** ou **4096 bits**.
* Um número de 2048 bits tem aproximadamente **617 dígitos decimais**.
* Fatorar um número desse tamanho para descobrir $p$ e $q$ levaria mais tempo do que a idade do universo com os computadores clássicos atuais.

### Senior Note: Performance
O RSA é muito lento devido a essas exponenciações gigantescas. Por isso, na prática (como no HTTPS), usamos RSA apenas para trocar uma **chave simétrica** (como AES) no início da conexão. Depois, usamos AES para a transferência de dados, pois é muito mais rápido.