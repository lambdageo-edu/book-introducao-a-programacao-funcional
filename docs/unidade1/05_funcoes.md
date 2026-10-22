# Estrutura de Funções: Condicionais, Guardas e Pattern Matching

Na programação funcional, as funções são os blocos de construção primários. Neste capítulo, exploraremos a fundo como declarar e estruturar funções em Haskell. Analisaremos mecanismos para controle de fluxo e tomada de decisão: desde condicionais básicas até a elegância matemática de guardas e o poder do casamento de padrões (*pattern matching*).

---

## 1. Declaração de Funções e Assinaturas de Tipos

Em Haskell, é uma boa prática declarar a **assinatura de tipo** da função logo acima de sua implementação. Isso funciona como documentação para o programador e como uma especificação que o compilador validará estritamente.

```haskell
-- Assinatura de tipo: recebe dois números inteiros e retorna outro
soma :: Int -> Int -> Int
soma x y = x + y
```

### Funções com Múltiplos Parâmetros (Currying)
Como você deve ter notado, a assinatura de tipo utiliza setas (`->`) para separar os argumentos e o retorno. Em Haskell, sob o capô, **toda função recebe exatamente um único argumento** e retorna uma nova função que espera o próximo. Esse conceito matemático é conhecido como **Currying** (em homenagem ao matemático Haskell Curry) e será detalhado no capítulo de funções de alta ordem.

---

## 2. Expressões Condicionais: `if-then-else`

Ao contrário das linguagens imperativas, onde o `if` é uma estrutura de controle (um comando), em Haskell o `if` é uma **expressão**. Isso significa que ele **obrigatoriamente deve retornar um valor**.

```haskell
sinal :: Int -> Int
sinal n = if n < 0 then -1 else if n == 0 then 0 else 1
```

### Regras do `if` em Haskell:
1. **O bloco `else` é obrigatório**: Como uma expressão precisa avaliar para algum valor em qualquer cenário, não podemos omitir o `else`.
2. **Tipos idênticos**: Os blocos `then` e `else` devem retornar valores do **mesmo tipo**.

---

## 3. Guardas (Guards)

Quando lidamos com múltiplos caminhos condicionais, encadear vários blocos `if-then-else` torna o código poluído e difícil de ler. Para resolver isso, Haskell fornece os **Guardas** (representados por barras verticais `|`), que testam predicados booleanos sequencialmente.

Reescrevendo a função `sinal` com guardas:

```haskell
sinal :: Int -> Int
sinal n | n < 0     = -1
        | n == 0    = 0
        | otherwise = 1
```

### Detalhes Sintáticos:
* O símbolo `=` é colocado apenas após a condição do guarda para ligá-la à sua expressão de retorno.
* A palavra-chave **`otherwise`** é equivalente a `True` e serve como a ramificação padrão (o equivalente ao `else` final).

---

## 4. Casamento de Padrões (Pattern Matching)

O **Casamento de Padrões** (*Pattern Matching*) é o mecanismo mais expressivo para tomada de decisões em linguagens funcionais. Em vez de testarmos condições lógicas gerais, definimos equações separadas para diferentes formatos ou valores de dados que a função pode receber.

Considere a negação lógica:

```haskell
negar :: Bool -> Bool
negar True  = False
negar False = True
```

O compilador analisa a chamada da função de cima para baixo. Se passarmos `True`, ele seleciona a primeira linha; se passarmos `False`, seleciona a segunda.

### O Padrão Curinga (Wildcard)
Podemos usar o caractere sublinhado (`_`) como um curinga para casar com "qualquer valor", descartando-o se não formos utilizá-lo. Isso é muito comum para definir casos padrão:

```haskell
ehDificil :: String -> Bool
ehDificil "Haskell" = False
ehDificil "Clojure" = False
ehDificil _         = True  -- Qualquer outra linguagem é considerada difícil por padrão
```

### Pattern Matching em Tuplas
Podemos desestruturar estruturas de dados complexas, como tuplas, diretamente nos argumentos da função:

```haskell
somarPares :: (Int, Int) -> (Int, Int) -> (Int, Int)
somarPares (x1, y1) (x2, y2) = (x1 + x2, y1 + y2)
```

### Padrões Exaustivos
Ao escrever uma série de padrões, é importante cobrir **todos os casos possíveis** do tipo. Se nenhuma equação casar com o valor recebido, o programa falha em tempo de execução:

```haskell
exemploRuim :: [Int] -> Int
exemploRuim (x:xs) = x + exemploRuim xs
-- exemploRuim []  ==> *** Exception: Non-exhaustive patterns
```

(O padrão `(x:xs)` casa com uma lista não vazia, separando o primeiro elemento `x` do restante `xs`; ele é o assunto do próximo capítulo. Aqui, falta a equação para a lista vazia `[]`.)

O GHC oferece a opção de compilação `-fwarn-incomplete-patterns` (ou `-Wall`), que emite um aviso quando uma sequência de padrões não cobre todos os construtores do tipo. Quando precisamos de um comportamento padrão para os casos restantes, usamos o curinga `_` como último padrão.

### As-Patterns: Nomeando o Todo e as Partes
Às vezes queremos desestruturar um valor **e** manter uma referência ao valor completo. O símbolo `@` (chamado *as-pattern*) faz exatamente isso: no padrão `xs@(_:xs')`, a variável `xs` é ligada à lista inteira que casou, enquanto `xs'` recebe apenas a cauda.

```haskell
-- Retorna todos os sufixos não vazios de uma lista
sufixos :: [a] -> [[a]]
sufixos xs@(_:xs') = xs : sufixos xs'
sufixos _          = []
```

```haskell
Prelude> sufixos "foo"
["foo","oo","o"]
```

Sem o as-pattern, teríamos de reconstruir a lista no corpo da função (`sufixos (x:xs) = (x:xs) : ...`), o que é menos legível e ainda aloca um novo nó de lista em tempo de execução — o as-pattern **compartilha** o valor original em vez de copiá-lo.

### Erros Comuns de Iniciantes com Padrões
Um equívoco clássico é tentar comparar um argumento com o valor de uma variável já definida:

```haskell
data Fruta = Maca | Laranja

maca = "maca"

qualFruta :: String -> Fruta
qualFruta f = case f of
                maca    -> Maca     -- ERRADO!
                laranja -> Laranja
```

O `maca` dentro do `case` **não** se refere à variável global `maca`: é uma nova variável de padrão local que casa com *qualquer* valor (um padrão *irrefutável*). O correto é comparar com valores literais:

```haskell
qualFruta f = case f of
                "maca"    -> Maca
                "laranja" -> Laranja
```

Outro erro comum: um nome pode aparecer **apenas uma vez** em um padrão. Não é possível escrever `saoIguais x x = True` para exprimir "os dois argumentos devem ser idênticos" — para isso, usamos guardas: `saoIguais x y | x == y = ...`.

---

## 5. Expressões `case`

O casamento de padrões que vimos acima é aplicado na própria definição das equações da função. Contudo, às vezes precisamos realizar um casamento de padrões dentro de uma expressão no meio da implementação. Para isso, utilizamos a construção **`case ... of`**:

```haskell
descreverNumero :: Int -> String
descreverNumero n = "O numero fornecido eh: " ++ case n of
    0 -> "Zero"
    1 -> "Um"
    _ -> "Muitos"
```

A estrutura `case` avalia a expressão e executa o casamento de padrões de cima para baixo, retornando o valor associado ao primeiro padrão compatível.

---

## 6. Combinando Padrões e Guardas

Padrões e guardas se complementam: o padrão verifica a *forma* do dado, e o guarda verifica *condições* sobre os valores extraídos. Escrever uma função como uma série de equações usando os dois mecanismos torna os casos explícitos logo de início. Compare a função `meuDrop` escrita com `if`:

```haskell
meuDrop n xs = if n <= 0 || null xs
               then xs
               else meuDrop (n - 1) (tail xs)
```

Com a reformulação usando padrões e guardas:

```haskell
meuDrop n xs | n <= 0 = xs
meuDrop _ []          = []
meuDrop n (_:xs)      = meuDrop (n - 1) xs
```

Essa mudança de estilo nos permite enumerar *de frente* os casos em que a função se comporta de maneira diferente — enterrar as decisões dentro de expressões `if` torna o código mais difícil de ler.

---

## 7. A Regra Offside: Indentação com Significado

Em Haskell, o espaço em branco **tem significado**: a linguagem usa a indentação para delimitar blocos de código, dispensando chaves e pontos-e-vírgulas. Esse mecanismo é chamado de **regra offside**, e vale a pena entendê-lo antes que ele o surpreenda com um `parse error`.

As regras práticas:

1. **Declarações de nível superior devem começar todas na mesma coluna.** A primeira declaração do arquivo pode começar em qualquer coluna; as seguintes devem alinhar-se a ela.
2. **Uma linha mais indentada que a anterior é uma *continuação*** da declaração anterior — é assim que quebramos definições longas em várias linhas.
3. **Dentro de `let`, `where` e `do`**, o compilador memoriza a coluna do primeiro item: linhas na mesma coluna iniciam um novo item do bloco; linhas mais à direita continuam o item anterior; linhas mais à esquerda encerram o bloco.

```haskell
-- Correto: as definições do where alinhadas na mesma coluna
emprestar valor saldo = if valor < reserva * 0.5
                        then Just novoSaldo
                        else Nothing
    where reserva   = 100
          novoSaldo = saldo - valor
```

!!! note "`Just` e `Nothing`"
    `emprestar` devolve um valor do tipo `Maybe`: `Just novoSaldo` quando o empréstimo é possível, `Nothing` quando não é. `Maybe` é a forma de Haskell representar "um resultado que pode não existir", sem `null`. Por enquanto basta reconhecer os dois construtores; o tipo é definido no capítulo de [Tipos Algébricos](../unidade2/01_declarando_tipos_classes.md).

```haskell
-- ERRO: a segunda declaração está em coluna diferente da primeira
    primeiraDeclaracao = 1
  segundaDeclaracao = 2    -- parse error!
```

!!! warning
    **Use espaços, nunca tabs.** Editores interpretam tabulações com larguras diferentes (8 colunas no Unix, 4 no Windows), então um arquivo com tabs pode compilar na sua máquina e quebrar na do colega. O padrão da linguagem assume a convenção Unix — configure seu editor para inserir espaços em arquivos `.hs`.

A propósito: a regra offside não é obrigatória — Haskell aceita estruturação explícita com chaves e pontos-e-vírgulas (`let { a = 1; b = 2 } in a + b`) — mas ela praticamente nunca é usada em programas reais.

### Ocultamento (Shadowing)
Blocos `let` e `where` podem ser aninhados, e um nome interno pode **ocultar** um nome externo de mesmo identificador:

```haskell
bar = let x = 1
      in ((let x = "foo" in x), x)   -- resulta em ("foo", 1)
```

O `x` interno esconde o externo — mesmo nome, tipo e valor diferentes. O ocultamento também pode esconder **parâmetros da função**, uma fonte clássica de confusão:

```haskell
quux a = let a = "foo"
         in a ++ "eek!"    -- o parâmetro 'a' nunca é usado!
```

Ocultamento leva a bugs sutis; o GHC oferece a opção `-fwarn-name-shadowing` (incluída no `-Wall`) para avisar sempre que um nome ocultar outro.

No próximo capítulo, veremos como aplicar recursão e casamento de padrões sobre estruturas de dados recursivas como **Listas**.

---

> **Nota de atribuição:** partes deste capítulo adaptam material de *Real World Haskell*, de Bryan O'Sullivan, Don Stewart e John Goerzen ([book.realworldhaskell.org](http://book.realworldhaskell.org/read/)), sob a licença [Creative Commons Attribution-Noncommercial 3.0](http://creativecommons.org/licenses/by-nc/3.0/).