# O Sistema de Tipos do Haskell

O sistema de tipos é uma das características mais marcantes e poderosas do Haskell. Neste capítulo, exploraremos o que são tipos, por que eles são fundamentais para a qualidade de software, como funciona o sistema de tipos estático do Haskell e quais são os tipos básicos e compostos fornecidos pela linguagem.

---

## 1. Por que se preocupar com tipos?

Na ciência da computação e na matemática, os sistemas de tipos servem para impor consistência e garantir a correção dos programas. De acordo com grandes teóricos da área:

> "Sistemas de tipos são geralmente formulados como coleções de regras para verificar a 'consistência' dos programas." (Benjamin Pierce, 2004)

> "O propósito fundamental de um sistema de tipos é prevenir a ocorrência de erros de execução durante a execução de um programa." (Luca Cardelli, *Type Systems*, 2004)

No nível mais baixo da computação, a memória do computador lida apenas com bytes brutos, sem qualquer estrutura inerente. O sistema de tipos fornece a **abstração** necessária para atribuirmos significado a esses bytes. Ele nos permite dizer que *"estes bytes representam texto"*, *"aqueles bytes representam um número decimal"*, e assim por diante. Ao introduzir essa abstração, o sistema de tipos impede misturas acidentais que causariam comportamentos indefinidos no software.

---

## 2. O Tríplice Pilar do Sistema de Tipos do Haskell

Haskell possui um sistema de tipos que é **forte**, **estático** e **inferido**. Vamos entender o significado e as vantagens de cada um desses conceitos:

```text
       ┌────────────────────────┐
       │   Sistema de Tipos     │
       │      do Haskell        │
       └──────────┬─────────────┘
                  │
         ┌────────┼────────┐
         ▼        ▼        ▼
     [Forte]  [Estático] [Inferido]
```

### 1. Tipagem Forte
Dizer que o Haskell possui tipagem **forte** significa que a linguagem garante que um programa não pode executar operações que não façam sentido com determinado tipo de dados. Por exemplo, tentar somar um número a um valor lógico (booleano) causará uma rejeição imediata:

```haskell
ghci> 1 + False
-- ERRO! O operador (+) exige que ambos os operandos sejam numéricos.
```

Diferentemente de linguagens como C ou JavaScript, o Haskell não realiza coerção automática de tipos (conversões implícitas). O compilador do C converteria silenciosamente um número inteiro (`int`) em ponto flutuante (`float`) se uma função exigisse um ponto flutuante, enquanto o Haskell exige que você chame explicitamente uma função de conversão (como `fromIntegral`, que converte um inteiro para qualquer tipo numérico).

### 2. Tipagem Estática
Ter um sistema de tipos **estático** significa que o compilador sabe o tipo de cada valor e expressão em tempo de compilação, ou seja, **antes** de qualquer linha de código ser executada. Se houver um erro de tipo, o compilador recusará gerar o programa executável, eliminando toda uma classe de falhas que, em outras linguagens, só seriam descobertas durante a execução do programa em produção.

### 3. Inferência de Tipos
Apesar de ser estaticamente tipado, você raramente é obrigado a declarar explicitamente o tipo de cada variável ou função. O compilador do Haskell utiliza um algoritmo avançado de dedução lógica (inferência de tipos de Hindley-Milner) que analisa a estrutura do seu código e descobre automaticamente os tipos corretos.

```haskell
-- Você não precisa escrever o tipo:
soma x y = x + y

-- O compilador deduz que x e y devem ser do tipo numérico.
```

---

## 3. Tipos Básicos em Haskell

A biblioteca padrão do Haskell (o `Prelude`) fornece diversos tipos de dados primitivos. Por convenção, **todos os nomes de tipos em Haskell começam com letra maiúscula**:

* **`Bool`**: Valores lógicos, contendo apenas os construtores `True` e `False`.
* **`Char`**: Um único caractere Unicode (ex: `'a'`, `'9'`, `'λ'`), delimitado por aspas simples.
* **`String`**: Sequências de texto (ex: `"Haskell"`), delimitadas por aspas duplas. Sob o capô, `String` é apenas um sinônimo de tipo para uma lista de caracteres (`[Char]`).
* **`Int`**: Inteiros com precisão limitada de acordo com a máquina (tipicamente de 32 ou 64 bits). É mais eficiente para cálculos rápidos.
* **`Integer`**: Inteiros com precisão arbitrária. Não possui limite de tamanho máximo e cresce dinamicamente até esgotar a memória do computador. Útil para cálculos matemáticos que exigem precisão absoluta (como criptografia).
* **`Float`**: Números decimais de ponto flutuante de precisão simples.
* **`Double`**: Números decimais de ponto flutuante de precisão dupla. Recomendado para a maioria das computações científicas por minimizar erros de arredondamento.

---

## 4. Tipos Compostos

Podemos combinar os tipos básicos para construir estruturas de dados mais complexas:

### 1. Listas
Uma lista é uma sequência de elementos que devem ser **todos do mesmo tipo**. A sintaxe utiliza colchetes:

```haskell
numeros :: [Int]
numeros = [1, 2, 3]

letras :: [Char]
letras = ['a', 'b', 'c']
```

### 2. Tuplas
Uma tupla é uma sequência de tamanho fixo onde cada elemento pode ter **um tipo diferente**. A sintaxe utiliza parênteses:

```haskell
cadastro :: (String, Int, Bool)
cadastro = ("Sergio", 42, True)
```

O tipo de uma tupla registra o número, a posição e o tipo de seus elementos. Isso significa que `(Bool, Char)` e `(Char, Bool)` são tipos **distintos**, assim como `(Bool, Char)` e `(Bool, Char, Char)`. Tuplas de dois elementos são chamadas de *pares*; as de três, *triplas*. Na prática, tuplas com muitos elementos tornam o código pesado e são raras. Existe ainda o tipo especial `()` (pronunciado "unit"), uma tupla de zero elementos com um único valor, também escrito `()` — semelhante ao `void` do C.

Para pares, as funções `fst` e `snd` retornam o primeiro e o segundo elemento, respectivamente:

```haskell
Prelude> fst (1, 'a')
1
Prelude> snd (1, 'a')
'a'
```

!!! warning
    **Tuplas Haskell não são "listas imutáveis".** Se você vem do Python, não leve essa ideia para cá: `fst` e `snd` só funcionam para pares, e não é possível indexar ou iterar uma tupla como uma lista. Use tuplas para coleções *pequenas e de tamanho fixo* com tipos heterogêneos — por exemplo, para retornar múltiplos valores de uma função.

---

## 5. Polimorfismo Paramétrico e Variáveis de Tipo

Considere a função `last`, que busca o último elemento de uma lista. Ela funciona da mesma maneira não importa o tipo dos elementos:

```haskell
Prelude> last [1,2,3,4,5]
5
Prelude> last "baz"
'z'
```

Para expressar isso, sua assinatura contém uma **variável de tipo**:

```haskell
Prelude> :type last
last :: [a] -> a
```

Aqui, `a` é a variável de tipo (sempre iniciada com letra **minúscula**, em contraste com os nomes de tipos concretos, que começam com maiúscula). Lemos a assinatura como: *"recebe uma lista cujos elementos têm algum tipo `a`, e retorna um valor desse mesmo tipo `a`"*. Quando uma função tem variáveis de tipo na assinatura, dizemos que ela é **polimórfica**. Esse tipo de polimorfismo é chamado de **polimorfismo paramétrico** — a inspiração direta dos *generics* de Java/C# e dos *templates* de C++.

### Raciocinando sobre assinaturas polimórficas
O polimorfismo paramétrico traz um poder de raciocínio surpreendente: como a função não pode saber qual é o tipo real de `a`, ela não pode criar, inspecionar nem transformar esse valor. Observe `fst :: (a, b) -> a` — a única implementação razoável possível (fora loops infinitos ou falhas) é *retornar o primeiro elemento do par*. A assinatura sozinha praticamente determina o comportamento!

### A assinatura revela pureza
Em Haskell, os efeitos colaterais aparecem no tipo: se o resultado de uma função começa com `IO`, ela é **impura** (interage com o mundo externo); caso contrário, é **pura**:

```haskell
Prelude> :type lines
lines :: String -> [String]        -- pura
Prelude> :type readFile
readFile :: FilePath -> IO String  -- impura (lê do disco)
```

O sistema de tipos nos impede de misturar acidentalmente código puro e impuro — voltaremos a isso no capítulo de programas interativos (Unidade 2).

---

## 6. Classes de Tipos (Typeclasses)

Muitas funções em Haskell podem ser usadas em múltiplos tipos diferentes. Por exemplo, o operador `==` pode comparar inteiros, caracteres ou booleanos. Essa funcionalidade é governada por **Typeclasses** (Classes de Tipos), que definem comportamentos abstratos compartilhados por vários tipos:

```text
              ┌───────────────┐
              │  Typeclasses  │
              └───────┬───────┘
                      │
     ┌────────┬───────┼───────┬────────┐
     ▼        ▼       ▼       ▼        ▼
   [Eq]     [Ord]  [Show]   [Read]   [Num]
```

* **`Eq`**: Tipos cujos valores podem ser comparados por igualdade (`==` e `/=`).
* **`Ord`**: Tipos cujos valores possuem ordenação linear (`<`, `>`, `compare`).
* **`Show`**: Tipos cujos valores podem ser convertidos em uma `String` legível para impressão na tela.
* **`Read`**: Tipos cujos valores podem ser lidos e convertidos a partir de uma `String`.
* **`Num`**: Tipos que possuem comportamento numérico básico (como soma, subtração e multiplicação).

Nos próximos capítulos, estudaremos como definir nossas próprias funções e tipos algébricos personalizados que herdam esses comportamentos.

---

> **Nota de atribuição:** partes deste capítulo adaptam material de *Real World Haskell*, de Bryan O'Sullivan, Don Stewart e John Goerzen ([book.realworldhaskell.org](http://book.realworldhaskell.org/read/)), sob a licença [Creative Commons Attribution-Noncommercial 3.0](http://creativecommons.org/licenses/by-nc/3.0/).
