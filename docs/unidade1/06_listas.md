# Listas, Recursão e Compreensão

As listas são a estrutura de dados mais fundamental e amplamente utilizada na programação funcional. Neste capítulo, estudaremos a definição de listas em Haskell, aprenderemos a realizar operações recursivas sobre elas por meio de casamento de padrões, e exploraremos a sintaxe expressiva de compreensões de listas.

---

## 1. O que é uma Lista em Haskell?

Em Haskell, uma lista é uma coleção homogênea de elementos, o que significa que **todos os elementos devem ter exatamente o mesmo tipo**. A estrutura de dados de uma lista é inerentemente **recursiva** e implementada como uma lista simplesmente encadeada.

Uma lista em Haskell é definida formalmente por dois construtores:

1. **Lista Vazia (`[]`)**: Representa a ausência de elementos.
2. **O Construtor Cons (`:`)**: Um operador que recebe um elemento (a cabeça) e o anexa no início de outra lista (a cauda).

Por exemplo, a lista `[1, 2, 3, 4]` é avaliada de forma encadeada como:

```haskell
lista = 1 : 2 : 3 : 4 : []
```

### Açúcares Sintáticos (Syntax Sugar)
Escrever o operador `:` consecutivamente seria muito verboso. Haskell oferece açúcares sintáticos amigáveis para a criação de listas:

* **Listas Explícitas**: `[1, 2, 3, 4]`
* **Faixas de Valores (Ranges)**: `[1..10]` gera uma lista de 1 a 10.
* **Ranges com Passo**: `[0, 2..10]` gera a lista de pares `[0, 2, 4, 6, 8, 10]`.
* **Strings**: Lembre-se que `"Haskell"` é apenas uma notação abreviada para a lista de caracteres `['H', 'a', 's', 'k', 'e', 'l', 'l']`.

---

## 2. Funções Essenciais sobre Listas

A biblioteca padrão do Haskell (`Prelude` e o módulo `Data.List`) fornece dezenas de funções prontas para inspecionar e transformar listas. Como o pão e a manteiga da programação funcional, elas merecem atenção séria: se não tivermos essa caixa de ferramentas na ponta dos dedos, acabaremos perdendo tempo reinventando funções que já existem. Um exercício rápido e útil: após ler sobre cada função, tente escrever a definição dela você mesmo.

### Inspeção básica
* **`head`**: Retorna o primeiro elemento (a cabeça) da lista. Exige que a lista não esteja vazia.
* **`tail`**: Retorna a lista sem o seu primeiro elemento (a cauda).
* **`last`**: Retorna o último elemento. **`init`**: retorna tudo, exceto o último.
* **`null`**: Retorna `True` se a lista estiver vazia.
* **`length`**: Calcula o número de elementos na lista.
* **`(!!)`**: Acessa um elemento por índice (0-indexado). Ex: `[10, 20, 30] !! 1 = 20`.
* **`elem`** / **`notElem`**: Verifica a presença (ou ausência) de um valor. Ex: ``2 `elem` [1,2,3] = True``.

### Construção e combinação
* **`(++)`**: Concatena duas listas. Ex: `[1, 2] ++ [3, 4] = [1, 2, 3, 4]`.
* **`concat`**: Concatena uma lista de listas, removendo um nível de aninhamento. Ex: `concat [[1,2],[3]] = [1,2,3]`.
* **`reverse`**: Inverte a ordem dos elementos.
* **`zip`**: Combina duas listas em uma lista de pares, parando na mais curta. Ex: `zip [1,2,3] "ab" = [(1,'a'),(2,'b')]`.
* **`zipWith`**: Combina duas listas aplicando uma função a cada par. Ex: `zipWith (+) [1,2,3] [4,5,6] = [5,7,9]`.

### Sublistas
* **`take`**: Extrai os primeiros $n$ elementos de uma lista.
* **`drop`**: Remove os primeiros $n$ elementos de uma lista.
* **`splitAt`**: Combina `take` e `drop`, retornando o par. Ex: `splitAt 3 "foobar" = ("foo","bar")`.
* **`takeWhile`** / **`dropWhile`**: Tomam/descartam elementos do início *enquanto* um predicado for verdadeiro. Ex: `takeWhile odd [1,3,5,6,9] = [1,3,5]`.
* **`span`** / **`break`**: Tuplam os resultados de `takeWhile`/`dropWhile`; `span` consome enquanto o predicado é verdadeiro, `break` enquanto é falso.

### Listas de booleanos e predicados
* **`and`** / **`or`**: Generalizam `(&&)` e `(||)` para listas. Ex: `and [True,False] = False`.
* **`all`** / **`any`**: Recebem um predicado; `all` exige que valha para todos os elementos, `any` para pelo menos um. Ex: `all odd [1,3,5] = True`.

### Strings (que são listas!)
* **`lines`** / **`unlines`**: Divide um texto em linhas / junta linhas com `\n`.
* **`words`** / **`unwords`**: Divide um texto em palavras (por espaços em branco) / junta palavras com espaço.

### Funções Parciais vs. Totais
Várias dessas funções se comportam mal com listas vazias:

```haskell
Prelude> head []
*** Exception: Prelude.head: empty list
```

Funções que só têm valor de retorno definido para um *subconjunto* das entradas válidas são chamadas de **funções parciais**; as que retornam resultados válidos para todo o domínio são **funções totais**. Chamar uma função parcial com uma entrada que ela não trata é provavelmente a maior fonte de bugs evitáveis em programas Haskell — saiba sempre se a função que você usa é parcial ou total. Uma alternativa segura é escrever versões totais com `Maybe` (ex: `safeHead :: [a] -> Maybe a`), exercício que faremos no fim do módulo.

!!! tip
    **Prefira `null` a `length` para testar se uma lista está vazia.** Como a lista é encadeada, `length` precisa percorrê-la inteira — e, com listas infinitas (comuns em Haskell!), `length xs > 0` entra em loop, enquanto `null xs` roda em tempo constante.

---

## 3. Pattern Matching e Recursão sobre Listas

Como a estrutura de uma lista é definida pelo construtor de cabeça/cauda (`:`), podemos utilizar o casamento de padrões para separar a cabeça do restante da lista utilizando a notação `(x:xs)` (onde `x` é a cabeça e `xs` é a cauda).

### Exemplo 1: Somando elementos de uma lista
Para calcular a soma de uma lista de números recursivamente, definimos dois casos:

1. **Caso Base**: A soma de uma lista vazia `[]` é `0`.
2. **Caso Recursivo**: A soma de uma lista não vazia `(x:xs)` é a cabeça `x` somada ao resultado da chamada recursiva para a cauda `xs`.

```haskell
somarLista :: Num a => [a] -> a
somarLista []     = 0
somarLista (x:xs) = x + somarLista xs
```

Pensar na estrutura da lista — vazia, ou um elemento seguido do restante — e tratar os dois casos separadamente é uma abordagem chamada **recursão estrutural**. O caso não-recursivo (lista vazia) é o **caso base**; o outro é o **caso recursivo** (ou *indutivo*). Essa técnica não se limita a listas: vale para qualquer tipo de dado algébrico, como veremos na Unidade 2.

### Exemplo 2: O Clássico Quicksort em Haskell
A expressividade do casamento de padrões e da recursão sobre listas permite implementar o famoso algoritmo de ordenação **Quicksort** de forma incrivelmente compacta em Haskell:

```haskell
quicksort :: Ord a => [a] -> [a]
quicksort []     = []
quicksort (x:xs) = quicksort menores ++ [x] ++ quicksort maiores
  where
    menores = [a | a <- xs, a <= x]
    maiores = [a | a <- xs, a >  x]
```

Nesta função, tomamos a cabeça `x` como o pivô. Usamos compreensões de lista (a notação `[a | a <- xs, a <= x]`, que lê-se "todos os `a` tirados de `xs` tais que `a <= x`", explicada na seção 5 deste capítulo) para filtrar os elementos da cauda `xs` que são menores que o pivô (`menores`) e maiores que o pivô (`maiores`), ordenando cada uma dessas partes de forma recursiva antes de juntar tudo.

### Recursão de Cauda e Acumuladores
Como Haskell não tem laços `for`/`while`, o equivalente de um loop com variável acumuladora é uma função auxiliar recursiva que carrega o acumulador como parâmetro. Compare com o loop em C que converte uma string de dígitos em inteiro (`acc = acc * 10 + dígito`):

```haskell
import Data.Char (digitToInt)

asInt :: String -> Int
asInt xs = loop 0 xs
  where
    loop acc []     = acc
    loop acc (x:xs) = loop (acc * 10 + digitToInt x) xs
```

Passar `0` inicial equivale a inicializar a variável no começo do loop; cada chamada recursiva consome um elemento e atualiza o acumulador. Como a última coisa que `loop` faz é chamar a si mesma, ela é uma função **recursiva de cauda** (*tail recursive*) — o compilador transforma essas chamadas para executarem em espaço constante (*tail call optimisation*), exatamente como um loop imperativo.

!!! note
    **A convenção do apóstrofo.** O apóstrofo é um caractere válido em nomes Haskell (pronuncia-se "linha", como em matemática). É idiomático usar `acc'` para "o novo valor de `acc`": `loop acc (x:xs) = let acc' = acc * 10 + digitToInt x in loop acc' xs`. Reconheça o padrão ao ler código — mas cuidado, um apóstrofo é fácil de não enxergar.

---

## 4. Exemplo Trabalhado: `splitLines` Portável

Vamos juntar as peças do capítulo — funções da caixa de ferramentas, pattern matching e recursão — em um problema real. A função `lines` do Prelude divide um texto em linhas, mas só reconhece o `\n` do Unix; um arquivo gerado no Windows (`\r\n`) fica com "sujeira":

```haskell
Prelude> lines "a\r\nb"
["a\r","b"]
```

Vamos escrever uma versão portável, que aceita as duas convenções:

```haskell
splitLines :: String -> [String]
splitLines [] = []
splitLines cs =
    let (pre, suf) = break isLineTerminator cs
    in  pre : case suf of
                ('\r':'\n':rest) -> splitLines rest
                ('\r':rest)      -> splitLines rest
                ('\n':rest)      -> splitLines rest
                _                -> []

isLineTerminator :: Char -> Bool
isLineTerminator c = c == '\r' || c == '\n'
```

Como ler este código:

* **`break isLineTerminator cs`** particiona a string no primeiro terminador de linha: `pre` é a linha atual, `suf` é o restante (começando pelo terminador, se houver).
* O **`case`** inspeciona o sufixo: se começa com `\r\n`, `\r` ou `\n`, descartamos o terminador e continuamos recursivamente em `rest`; se não há terminador (string vazia), encerramos.
* Note a **organização**: a lógica importante vem primeiro, e a auxiliar `isLineTerminator` fica no final — o nome legível permite entender o código antes mesmo de ler sua definição.

A melhor forma de entender uma função assim é **experimentar as peças no GHCi**:

```haskell
Prelude> break (== ' ') "foo bar"
("foo"," bar")
Prelude> splitLines "foo\r\nbar"
["foo","bar"]
Prelude> splitLines "foo"
["foo"]
```

Esse hábito de testar cada pedaço interativamente traz um benefício quase acidental: como é complicado testar funções grandes no GHCi, tendemos a escrever **funções menores** — o que melhora ainda mais a legibilidade do código.

---

## 5. Compreensão de Listas

A **Compreensão de Listas** é uma notação matemática poderosa para filtrar e transformar coleções de dados, baseada na definição matemática de conjuntos. Sua sintaxe básica é:

```text
[ expressao | gerador, filtros ]
```

### Componentes:
* **Gerador (`x <- lista`)**: Extrai valores da lista um por um.
* **Filtros (Predicados Booleanos)**: Expressões booleanas que determinam se o valor gerado deve ser incluído na computação.

### Exemplos de Compreensão:

1. **Dobrar apenas os números ímpares de 1 a 10**:
   ```haskell
   Prelude> [x * 2 | x <- [1..10], odd x]
   [2, 6, 10, 14, 18]
   ```

2. **Gerar todas as coordenadas de um tabuleiro 3x3 (Produto Cartesiano)**:
   ```haskell
   Prelude> [(x, y) | x <- [1..3], y <- [1..3]]
   [(1,1), (1,2), (1,3), (2,1), (2,2), (2,3), (3,1), (3,2), (3,3)]
   ```

No próximo capítulo, veremos como abstrair loops e processamentos repetitivos de listas utilizando **Funções de Alta Ordem**.

---

## 6. Exercícios de Fixação

Adaptados do *Real World Haskell* (cap. 3 e 4):

1. Escreva versões "seguras" (totais) das funções parciais de lista: `safeHead :: [a] -> Maybe a`, `safeTail :: [a] -> Maybe [a]`, `safeLast :: [a] -> Maybe a` e `safeInit :: [a] -> Maybe [a]`.
2. Escreva uma função que calcula a **média** de uma lista de números. (Dica: use `fromIntegral` para converter o tamanho da lista.)
3. Escreva uma função que transforma uma lista em um **palíndromo**: dada `[1,2,3]`, deve retornar `[1,2,3,3,2,1]`. Depois, escreva uma função que verifica se uma lista *é* um palíndromo.
4. Defina uma função `intercala :: a -> [[a]] -> [a]` que junta uma lista de listas usando um valor separador: `intercala ',' ["foo","bar","baz"]` deve resultar em `"foo,bar,baz"`.
5. Escreva uma função `splitWith :: (a -> Bool) -> [a] -> [[a]]` que age como `words`, mas divide a lista em cada elemento para o qual o predicado retorna `False`.

---

> **Nota de atribuição:** partes deste capítulo adaptam material de *Real World Haskell*, de Bryan O'Sullivan, Don Stewart e John Goerzen ([book.realworldhaskell.org](http://book.realworldhaskell.org/read/)), sob a licença [Creative Commons Attribution-Noncommercial 3.0](http://creativecommons.org/licenses/by-nc/3.0/).