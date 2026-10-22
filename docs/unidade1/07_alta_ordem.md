# Funções de Alta Ordem, Currying e Composição

Neste capítulo, encerraremos nossa primeira unidade explorando o recurso que confere a máxima expressividade ao paradigma funcional: as **Funções de Alta Ordem** (*Higher-Order Functions*). Ao tratar funções como valores comuns, aprenderemos a parametrizar comportamentos, reutilizar padrões de iteração genéricos (`map`, `filter`, `fold`) e encadear transformações de dados de maneira matemática através da composição — descobrindo, no caminho, como *pensar sobre loops* em uma linguagem que não os possui.

---

## 1. O que são Funções de Alta Ordem?

Uma **Função de Alta Ordem** é simplesmente uma função que atende a pelo menos um dos seguintes requisitos:

1. Recebe uma ou mais funções como argumento.
2. Retorna uma função como resultado.

Essa flexibilidade é viabilizada pelo fato de que, em Haskell, as funções são **Cidadãs de Primeira Classe** (*First-Class Citizens*). Elas podem ser armazenadas em estruturas de dados, passadas como parâmetros e manipuladas exatamente como inteiros, booleanos ou strings.

Considere a função `duasVezes`, que recebe uma função `f` e aplica essa função consecutivamente a um valor `x`:

```haskell
duasVezes :: (a -> a) -> a -> a
duasVezes f x = f (f x)
```

Podemos utilizá-la passando comportamentos diferentes como argumentos:

```haskell
Prelude> duasVezes (*2) 4
16

Prelude> duasVezes reverse [1, 2, 3]
[1, 2, 3]
```

---

## 2. Funções Infixas e as Crases

Antes de aprofundar o currying, um recurso sintático que usaremos bastante: qualquer função de dois argumentos pode ser aplicada em **notação infixa** (entre os argumentos), envolvendo seu nome em crases (*backticks*):

```haskell
Prelude> elem 'a' "camogie"      -- notação prefixada
True
Prelude> 3 `elem` [1, 2, 4, 8]   -- notação infixa: lê-se "3 é elemento de..."
False
```

A melhoria de legibilidade fica mais evidente com funções do módulo `Data.List` como `isPrefixOf`, `isInfixOf` e `isSuffixOf`:

```haskell
Prelude> :module +Data.List
Prelude Data.List> "foo" `isPrefixOf` "foobar"
True
Prelude Data.List> "agulha" `isInfixOf` "palheiro cheio de agulhas"
True
```

A notação infixa é apenas uma conveniência sintática — não muda o comportamento da função. Escolha a forma que tornar o código mais legível em cada situação.

---

## 3. Currying e Aplicação Parcial

Em Haskell, **todas as funções recebem formalmente apenas um argumento**. Quando declaramos uma função que parece aceitar múltiplos parâmetros, como `soma :: Int -> Int -> Int`, o compilador na verdade cria uma função que recebe o primeiro `Int` e retorna outra função, que por sua vez recebe o segundo `Int` e finalmente calcula o resultado. 

Esse processo de transformar uma função de múltiplos parâmetros em uma cadeia de funções de um único parâmetro é chamado de **Currying**.

### A Seta `->` Só Tem Um Significado
Você pode se perguntar por que a seta `->` parece cumprir dois papéis na assinatura `dropWhile :: (a -> Bool) -> [a] -> [a]` — separar os argumentos entre si *e* separá-los do retorno. Na verdade, `->` tem um único significado: *uma função que recebe o tipo à esquerda e retorna o tipo à direita*. A seta é **associativa à direita**, então a assinatura acima é lida como:

```haskell
dropWhile :: (a -> Bool) -> ([a] -> [a])
```

Ou seja: `dropWhile` recebe um predicado e **retorna uma função** de listas para listas. Isso não é um detalhe teórico — é diretamente útil:

```haskell
Prelude> :module +Data.Char
Prelude Data.Char> :type dropWhile isSpace
dropWhile isSpace :: [Char] -> [Char]
Prelude Data.Char> map (dropWhile isSpace) [" a", "f", "   e"]
["a","f","e"]
```

O valor `dropWhile isSpace` é, por si só, uma função que remove espaços iniciais de uma string — pronta para ser passada ao `map`.

### Aplicação Parcial
Como as funções funcionam por Currying, podemos chamá-las passando **menos argumentos** do que o total esperado. Isso nos retorna uma nova função especializada:

```haskell
-- Função base:
multiplicar :: Int -> Int -> Int
multiplicar x y = x * y

-- Aplicação parcial: passamos apenas o primeiro argumento
dobro :: Int -> Int
dobro = multiplicar 2

triplo :: Int -> Int
triplo = multiplicar 3
```

Cada argumento fornecido "corta" um elemento do início da assinatura. Veja com `zip3`, que junta três listas em uma lista de triplas:

```haskell
Prelude> :type zip3
zip3 :: [a] -> [b] -> [c] -> [(a, b, c)]
Prelude> :type zip3 "foo"
zip3 "foo" :: [b] -> [c] -> [(Char, b, c)]
Prelude> zip3 "foo" [1,2,3] [True,False,True]
[('f',1,True),('o',2,False),('o',3,True)]
```

A aplicação parcial é um mecanismo fantástico para criar funções auxiliares dinâmicas na hora de passar para funções de alta ordem — e frequentemente evita a criação cansativa de funções descartáveis.

### Seções: Aplicação Parcial de Operadores
Haskell fornece um atalho para aplicar parcialmente operadores infixos: colocando o operador entre parênteses, podemos fixar o argumento da esquerda **ou** da direita. Esse recurso é chamado de **seção** (*section*):

```haskell
Prelude> (1+) 2
3
Prelude> map (*3) [24, 36]
[72, 108]
Prelude> map (2^) [3, 5, 7]    -- fixa a base: 2^3, 2^5, 2^7
[8, 32, 128]
Prelude> map (^2) [3, 5, 7]    -- fixa o expoente: 3^2, 5^2, 7^2
[9, 25, 49]
```

Funções nomeadas também podem virar seções usando crases: `` (`elem` ['a'..'z']) `` é uma função que verifica se um caractere é letra minúscula.

### Funções Anônimas (Lambdas)
Podemos escrever funções sem nome — as **lambdas**, em referência ao cálculo lambda. A sintaxe usa uma barra invertida `\` (lembrando a letra λ), seguida dos argumentos e de uma seta `->`:

```haskell
Prelude> map (\x -> x * x + 1) [1, 2, 3]
[2, 5, 10]
```

Diferentemente das funções nomeadas, uma lambda só pode ter **uma única cláusula** — portanto, cuidado ao usar padrões em lambdas: certifique-se de que o padrão não pode falhar. Por questão de legibilidade, prefira a aplicação parcial ou uma função auxiliar nomeada (via `where`) quando a lambda começar a crescer: o nome bem escolhido informa ao leitor o que a função faz.

### Estudo de Caso: Quatro Versões da Mesma Função
Vamos escrever `estaEmAlguma`, que verifica se uma string (`agulha`) aparece dentro de alguma string de uma lista (`palheiro`). A evolução das quatro versões resume tudo o que vimos até aqui:

```haskell
import Data.List (isInfixOf)

-- Versão 1: função auxiliar nomeada
estaEmAlguma agulha palheiro = any contida palheiro
    where contida s = agulha `isInfixOf` s

-- Versão 2: lambda
estaEmAlguma2 agulha palheiro = any (\s -> agulha `isInfixOf` s) palheiro

-- Versão 3: aplicação parcial
estaEmAlguma3 agulha palheiro = any (isInfixOf agulha) palheiro

-- Versão 4: seção com crases (a mais idiomática)
estaEmAlguma4 agulha palheiro = any (agulha `isInfixOf`) palheiro
```

Na versão 3, `isInfixOf agulha` é a função parcialmente aplicada: fixamos o primeiro argumento e obtemos uma função que espera apenas a string a examinar — exatamente o que `any` precisa. A versão 4 usa uma seção para preservar a legibilidade da notação infixa. As quatro são equivalentes; a progressão mostra como o currying e as seções eliminam código descartável.

---

## 4. O Trio de Ouro: Map, Filter e Fold

Diferentemente das linguagens tradicionais, Haskell não tem laços `for` nem `while`. Como processar coleções de dados, então? A resposta está em perceber que quase todos os loops que escrevemos na vida seguem **três padrões**: transformar cada elemento, selecionar alguns elementos, ou acumular um resultado. Haskell captura cada padrão em uma função de alta ordem — e é assim que devemos *pensar sobre loops* no paradigma funcional.

```text
       Lista de Entrada ─────► [ 1, 2, 3, 4, 5 ]
                                    │
       ┌───────────────┬────────────┴────────────┬──────────────┐
       ▼               ▼                         ▼              ▼
     [Map]          [Filter]                  [Foldr]        [Foldl]
  (Transformar)    (Selecionar)            (Acumular Dir) (Acumular Esq)
       │               │                         │              │
       ▼               ▼                         ▼              ▼
  [ 2,4,6,8,10 ]   [ 2, 4 ]                     15              15
```

### 1. `map`: Transformando Cada Elemento
Considere um loop C que eleva ao quadrado cada elemento de um array:

```c
void square(double *out, const double *in, size_t length)
{
    for (size_t i = 0; i < length; i++) {
        out[i] = in[i] * in[i];
    }
}
```

Em Haskell, escreveríamos esse "loop" com recursão explícita, como aprendemos no capítulo de listas:

```haskell
quadrados :: [Double] -> [Double]
quadrados (x:xs) = x*x : quadrados xs
quadrados []     = []
```

Agora um segundo "loop": converter cada letra de uma string para maiúscula:

```haskell
import Data.Char (toUpper)

maiusculas :: String -> String
maiusculas (x:xs) = toUpper x : maiusculas xs
maiusculas []     = []
```

Observe que as duas funções têm **exatamente a mesma estrutura** — só muda a operação aplicada à cabeça (`x*x` vs. `toUpper x`). Quando detectamos um idioma repetido, o passo natural em Haskell é abstraí-lo: extraímos a operação como parâmetro, e o que sobra é o `map`:

```haskell
map :: (a -> b) -> [a] -> [b]
map _ []     = []
map f (x:xs) = f x : map f xs
```

```haskell
quadrados2  = map (\x -> x * x)
maiusculas2 = map toUpper

Prelude> map (+10) [1, 2, 3]
[11, 12, 13]
```

### 2. `filter`: Selecionando Elementos
O segundo padrão de loop é verificar quais elementos satisfazem algum critério. Escrito com recursão explícita e guardas:

```haskell
somenteImpares :: [Int] -> [Int]
somenteImpares (x:xs) | odd x     = x : somenteImpares xs
                      | otherwise = somenteImpares xs
somenteImpares _                  = []
```

Novamente, o padrão é tão comum que o Prelude o abstrai — passando o *predicado* como parâmetro, obtemos o `filter`:

```haskell
filter :: (a -> Bool) -> [a] -> [a]
filter _ []     = []
filter p (x:xs)
    | p x       = x : filter p xs
    | otherwise = filter p xs
```

```haskell
Prelude> filter even [1..6]
[2, 4, 6]
```

### 3. `foldr` e `foldl`: Acumulando um Resultado
O terceiro padrão é **reduzir** a coleção a um único valor. Com recursão de cauda e acumulador (o "loop com variável acumuladora" do capítulo anterior):

```haskell
minhaSoma :: [Int] -> Int
minhaSoma xs = auxiliar 0 xs
    where auxiliar acc (x:xs) = auxiliar (acc + x) xs
          auxiliar acc _      = acc
```

Podemos descrever esse comportamento genericamente: *"faça algo com cada elemento, atualizando um acumulador, e retorne o acumulador ao final"*. Essa abstração é a família **`fold`** (o *reduce* de outras linguagens), que recebe uma função de passo, um valor inicial e a lista. Existem duas variantes, definidas assim:

```haskell
foldl :: (a -> b -> a) -> a -> [b] -> a
foldl passo acc (x:xs) = foldl passo (passo acc x) xs
foldl _     acc []     = acc

foldr :: (a -> b -> b) -> b -> [a] -> b
foldr passo ini (x:xs) = passo x (foldr passo ini xs)
foldr _     ini []     = ini
```

Com `foldl`, nossa soma vira uma linha — restam apenas as duas decisões que importam: o valor inicial do acumulador e como atualizá-lo:

```haskell
somaBonita :: [Integer] -> Integer
somaBonita xs = foldl (+) 0 xs
```

A diferença entre as variantes está na direção de associação:

* **`foldr` (Fold Right)**: Associa os elementos da direita para a esquerda. É útil para gerar listas e trabalhar com listas infinitas devido à avaliação preguiçosa.
* **`foldl` (Fold Left)**: Associa da esquerda para a direita, consumindo a lista da cabeça para o fim.

Para entender a diferença de associação, vamos expandir manualmente cada avaliação:

```haskell
foldl (+) 0 (1:2:3:[])
   == foldl (+) (0 + 1)             (2:3:[])
   == foldl (+) ((0 + 1) + 2)       (3:[])
   == foldl (+) (((0 + 1) + 2) + 3) []
   ==           (((0 + 1) + 2) + 3)          -- agrupa à esquerda

foldr (+) 0 (1:2:3:[])
   == 1 + (foldr (+) 0 (2:3:[]))
   == 1 + (2 + (foldr (+) 0 (3:[])))
   == 1 + (2 + (3 + (foldr (+) 0 [])))
   == 1 + (2 + (3 + 0))                      -- agrupa à direita
```

Há uma explicação intuitiva bonita de como `foldr` funciona: ele **substitui a lista vazia pelo valor inicial, e cada construtor `(:)` por uma aplicação da função**:

```haskell
1 : (2 : (3 : []))
1 + (2 + (3 + 0 ))
```

Isso explica por que muitas funções de lista são expressáveis com `foldr` — inclusive `map` e `filter`:

```haskell
meuMap :: (a -> b) -> [a] -> [b]
meuMap f = foldr (\x ys -> f x : ys) []

meuFilter :: (a -> Bool) -> [a] -> [a]
meuFilter p = foldr passo []
  where passo x ys | p x       = x : ys
                   | otherwise = ys
```

E até a concatenação de listas `(++)`: anexar `ys` ao fim de `xs` é simplesmente *substituir o `[]` final de `xs` por `ys`*:

```haskell
anexa :: [a] -> [a] -> [a]
anexa xs ys = foldr (:) ys xs
```

A classe de funções expressáveis com `foldr` é chamada de **recursiva primitiva** — e um número surpreendente de funções de manipulação de listas se encaixa nela (como desafio avançado: até `foldl` pode ser escrita usando `foldr`!).

### `foldl`, Avaliação Preguiçosa e *Space Leaks*
Por causa da avaliação preguiçosa, `foldl` não calcula as somas parciais imediatamente — ele acumula uma expressão adiada (*thunk*) cada vez maior: `(((0+1)+2)+3)...`, que só é reduzida no final. Para listas grandes, isso consome memória proporcional ao tamanho da lista. Nas versões antigas do GHC, isso estourava a pilha (`*** Exception: stack overflow`); nas atuais, a pilha cresce conforme a necessidade e o programa termina — mas o custo continua lá. O comando `:set +s` do GHCi mostra tempo e memória de cada expressão:

```haskell
ghci> :set +s
ghci> foldl (+) 0 [1..10000000]
50000005000000
(3.68 secs, 1,612,377,968 bytes)
```

Esse *thunking* invisível é chamado de **space leak** (vazamento de espaço) — um obstáculo clássico para novos programadores Haskell. A solução é simples: o módulo `Data.List` define **`foldl'`**, uma versão estrita que avalia o acumulador a cada passo:

```haskell
ghci> import Data.List
ghci> foldl' (+) 0 [1..10000000]
50000005000000
(0.23 secs, 880,075,936 bytes)
```

Os números exatos variam de máquina para máquina, mas a diferença de ordem de grandeza não. (Programas compilados com `ghc -O` costumam escapar do problema, porque o otimizador percebe que o acumulador pode ser estrito — mas não conte com isso.)

Em programas reais, prefira sempre `foldl'` a `foldl`.

#### Como o `foldl'` Funciona Por Baixo: a Função `seq`
O `foldl'` não é mágica — ele usa uma função chamada **`seq`** para *forçar* a avaliação do acumulador a cada passo, em vez de deixá-lo como uma promessa (thunk):

```haskell
seq :: a -> b -> b
```

`seq` avalia seu **primeiro** argumento (só o suficiente para saber que não é um erro nem um loop infinito — a chamada *avaliação em forma normal fraca*), descarta o resultado, e retorna o **segundo** argumento. Com isso, podemos definir nosso próprio `foldl'`:

```haskell
meuFoldl' :: (a -> b -> a) -> a -> [b] -> a
meuFoldl' _    acc []     = acc
meuFoldl' step acc (x:xs) =
    let novo = step acc x
    in  novo `seq` meuFoldl' step novo xs
```

A expressão `novo `seq` meuFoldl' step novo xs` força `novo` a ser calculado *imediatamente* — antes da chamada recursiva — impedindo que thunks se acumulem. Esse é o "truque" por trás de toda função "estrita" em Haskell: usar `seq` (ou a anotação `!` de *bang patterns*) para dizer ao compilador "calcule isso agora, não depois".

### Por que usar `map`, `filter` e `fold` em vez de recursão explícita?
Porque essas funções são onipresentes e têm comportamento regular e previsível: um leitor entende `foldr (+) 0` de imediato, enquanto uma recursão explícita exige leitura cuidadosa para descobrir o que está acontecendo. Detectar um idioma repetido e abstraí-lo em uma função de alta ordem — escrevendo menos código — é um aspecto central do estilo Haskell.

---

## 5. Composição de Funções e Point-Free Style

Lembra da função `sufixos` do capítulo de funções (as-patterns)? Ela faz quase o mesmo que a função `tails` de `Data.List` — só descarta o sufixo vazio final. Parece um desperdício reescrevê-la do zero, quando podemos **reutilizar** funções existentes: `tails` seguida de `init` (que remove o último elemento):

```haskell
sufixos2 xs = init (tails xs)
```

Dando um passo atrás, vemos um padrão: *aplicar uma função e, em seguida, aplicar outra ao seu resultado*. Vamos transformar esse padrão em uma função:

```haskell
compor :: (b -> c) -> (a -> b) -> a -> c
compor f g x = f (g x)

sufixos3 xs = compor init tails xs
sufixos4    = compor init tails      -- currying nos deixa omitir o xs
```

Não precisamos escrever `compor` nós mesmos: "colar" funções é tão comum que o Prelude fornece o operador **`(.)`** — que corresponde à composição matemática $(f \circ g)(x) = f(g(x))$:

```haskell
(.) :: (b -> c) -> (a -> b) -> a -> c
f . g = \x -> f (g x)
```

```haskell
sufixos5 = init . tails

-- Função que dobra um número e depois soma 1:
dobroMaisUm :: Int -> Int
dobroMaisUm = (+1) . (*2)
```

O `(.)` não é sintaxe especial da linguagem — é um operador comum, que poderíamos ter definido nós mesmos.

### Pipelines Reais de Composição
Podemos encadear quantas funções quisermos, desde que o tipo de saída de cada uma case com o tipo de entrada da próxima. Um quebra-cabeça: contar quantas palavras de uma string começam com letra maiúscula.

```haskell
import Data.Char (isUpper)

capCount = length . filter (isUpper . head) . words
```

```haskell
Prelude> capCount "Hello there, Mom!"
2
```

Para entender uma cadeia de composições, leia **da direita para a esquerda**, conferindo os tipos de cada estágio no GHCi:

```haskell
Prelude> :type words
words :: String -> [String]
Prelude> :type isUpper . head
isUpper . head :: [Char] -> Bool
Prelude> :type filter (isUpper . head)
filter (isUpper . head) :: [[Char]] -> [[Char]]
```

`words` quebra o texto em palavras; `filter (isUpper . head)` mantém só as que começam com maiúscula; `length` conta o resultado.

Um exemplo tirado de uma aplicação real: extrair nomes de macros (`DLT_EN10MB`, `DLT_AX25`...) de um arquivo de cabeçalho C cheio de linhas como `#define DLT_EN10MB 1 /* Ethernet */`:

```haskell
import Data.List (isPrefixOf)

macros :: String -> [String]
macros = foldr passo [] . lines
  where
    passo linha acc
      | "#define DLT_" `isPrefixOf` linha = segundaPalavra linha : acc
      | otherwise                         = acc
    segundaPalavra = head . tail . words
```

Repare como o programa inteiro é a combinação das peças que estudamos: composição (`foldr passo [] . lines`), fold com guarda, seção com crases e o pipeline `head . tail . words`.

!!! warning
    **Use `head` com sabedoria.** `segundaPalavra` chama duas funções parciais (`head` e `tail`) — mas aqui podemos *provar* por inspeção que é seguro: o guarda garante que a linha contém pelo menos duas palavras (`"#define"` e a macro). Esse é o tipo de raciocínio necessário sempre que chamamos funções parciais; se alguém alterar o guarda, o código pode passar a explodir de forma sutil.

### Estilo Point-Free (Livre de Pontos)
Note que em definições como `dobroMaisUm = (+1) . (*2)` e `capCount = length . filter (...) . words`, nós não declaramos o argumento da função. Apenas declaramos como as funções se combinam.

Essa omissão do argumento de entrada é chamada de **Point-Free Style** e resulta em códigos muito limpos, focados puramente na combinação de funções e no fluxo dos dados.

---

## 6. Dicas para Escrever Código Legível

Ao longo da unidade vimos três "níveis" de ferramentas para processar listas — e vale estabelecer uma hierarquia de preferência:

1. **Funções de biblioteca compostas** (`map`, `filter`, `take`, composições): cada uma faz *uma coisa só* e tem comportamento conhecido. O leitor foca na ideia do código, não na mecânica.
2. **Folds**: exigem um pouco mais de esforço para ler que uma composição de `map`/`filter`, mas se comportam de forma regular e previsível.
3. **Recursão explícita (de cauda)**: totalmente geral — e é esse o problema. Como um loop imperativo, pode estar fazendo *qualquer coisa*, e o leitor precisa examinar a definição inteira para descobrir o quê.

**Regra prática**: não use um fold se puder compor funções de biblioteca; prefira um fold a um loop recursivo explícito.

Quanto às **lambdas**: elas interrompem o "fluxo" da leitura. Geralmente é tão fácil definir uma função local nomeada em um `let`/`where` — e o nome bem escolhido funciona como uma minúscula documentação local.

---

## 7. Exercícios de Fixação

Adaptados do *Real World Haskell* (cap. 4):

1. Reescreva a função `asInt` (do capítulo de listas) usando `foldl'`. Depois, estenda-a para tratar o sinal negativo: `asInt "-31337"` deve retornar `-31337`.
2. A função `concat :: [[a]] -> [a]` concatena uma lista de listas. Escreva sua própria versão usando `foldr`.
3. Escreva sua própria versão recursiva de `takeWhile`; depois, reescreva-a usando `foldr`.
4. Escreva `minhaQualquer :: (a -> Bool) -> [a] -> Bool` (equivalente ao `any`) de duas formas: com recursão explícita e com fold.
5. Usando composição e funções de biblioteca (sem recursão explícita), escreva uma função que recebe um texto e retorna a última palavra de cada linha.
6. **(Desafio)** Escreva `foldl` em termos de `foldr`. Aviso: não é trivial! Tenha à mão o GHCi (para descobrir o que a função `id` faz), lápis e papel.

Com isso, encerramos a **Unidade 1**! Agora você domina as bases teóricas do paradigma funcional e a modelagem matemática básica no Haskell.

---

> **Nota de atribuição:** partes deste capítulo adaptam material de *Real World Haskell*, de Bryan O'Sullivan, Don Stewart e John Goerzen ([book.realworldhaskell.org](http://book.realworldhaskell.org/read/)), sob a licença [Creative Commons Attribution-Noncommercial 3.0](http://creativecommons.org/licenses/by-nc/3.0/).