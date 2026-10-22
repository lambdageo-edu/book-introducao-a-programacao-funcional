# Lista de Exercícios: Haskell Básico

Esta página contém todas as listas de exercícios práticos da Unidade 1, servindo como material de estudo e fixação dos conceitos fundamentais de Haskell.

---


## 📘 Lista 1: Funções Básicas e Recursão em Haskell

### 1. Funções básicas

1. Defina uma função `quad` que receba um número e devolva o seu quadrado.
2. Escreva uma função `hipotenusa` que receba os catetos de um triângulo retângulo e devolva a hipotenusa.
    - **Dica:** use `sqrt`.

---

### 2. Definições locais (`where` e `let`)

1. Reescreva a função `hipotenusa` usando `where` para definir variáveis auxiliares.
2. Reescreva a mesma função usando `let … in …`.

---

### 3. Condicionais e guards

1. Defina a função `sinal` usando `if-then-else` que devolva:
    - `-1` se o número for negativo
    - `0` se for zero
    - `1` se for positivo
2. Reescreva a função `sinal` usando **guards**.
3. Implemente a função `classificaNota` que recebe um número (0–10) e retorna:
    - `"Reprovado"` se menor que 5
    - `"Recuperação"` se entre 5 e 6.9
    - `"Aprovado"` se maior ou igual a 7.

---

### 4. Pattern Matching

1. Reescreva a função `negar :: Bool -> Bool` usando **pattern matching**.
2. Defina a função `diasSemana` que recebe um número de 1 a 7 e retorna o nome do dia correspondente (1 → "Domingo", 2 → "Segunda", etc.).
    - Use **padrões explícitos**.
3. Defina a função fatorial com **pattern matching** nos casos base.

---

### 5. Recursão

1. Implemente a função `potencia b e` que calcula $b^e$ usando recursão (sem usar `(^)`).
2. Escreva uma função `mdc a b` que calcule o máximo divisor comum (algoritmo de Euclides).
3. Defina a sequência de Fibonacci de forma recursiva simples.
4. Refaça a função de Fibonacci usando a técnica de **recursão em cauda**.

---

## 📙 Lista 2: Listas e Compreensão em Haskell

### 1. Definição e Criação de Listas

1. Escreva manualmente a lista `[5,6,7]` usando apenas o construtor `(:)`.
2. Use **syntax sugar** para criar:
    - a lista de números de 1 a 20.
    - a lista de números pares de 0 a 20.
    - a lista de múltiplos de 3 de 0 a 30.
3. Represente a string `"HASKELL"` como lista de caracteres.

---

### 2. Funções e Operadores sobre Listas

1. Dada a lista `lista = [10..20]`, calcule:
    - `head lista`
    - `tail lista`
    - `take 5 lista`
    - `drop 7 lista`
    - `lista !! 3`
2. Escreva expressões em Haskell que retornem:
    - o tamanho da lista `[1..100]`.
    - a soma dos números de 1 a 50.
    - o produto dos números de 1 a 5.
3. Mostre duas formas diferentes de construir a lista `[1..10]` a partir da concatenação de sublistas.

---

### 3. Pattern Matching

1. Implemente uma função `primeiroOuZero :: [Int] -> Int` que retorna o primeiro elemento da lista, ou `0` se a lista for vazia.
2. Implemente uma função `segundoElemento :: [a] -> Maybe a` que retorna o segundo elemento de uma lista (ou `Nothing` se não existir).

---

### 4. Recursão em Listas

1. Implemente a função `meuLength :: [a] -> Int` que calcula o tamanho de uma lista usando recursão.
2. Implemente a função `meuSum :: Num a => [a] -> a` que soma os elementos de uma lista usando recursão.
3. Reescreva a função `reverse` utilizando recursão.
4. Implemente a função `meuDrop :: Int -> [a] -> [a]`, removendo os `n` primeiros elementos de uma lista.

---

### 5. Funções com Vários Argumentos

1. Implemente a função `meuZip :: [a] -> [b] -> [(a,b)]` que une duas listas em uma lista de pares.
2. Teste sua função `meuZip` com as listas `[1,2,3]` e `['a','b','c']`.

---

### 6. Compreensão de Listas

1. Gere, usando compreensão de listas:
    - uma lista com os quadrados dos números de 1 a 10.
    - uma lista com apenas os números pares de 1 a 20.
    - o produto cartesiano entre `[1,2,3]` e `['a','b']`.
2. Defina a função `divisores :: Int -> [Int]` que retorna a lista de todos os divisores de um número.
3. Defina a função `ehPrimo :: Int -> Bool` que verifica se um número é primo usando `divisores`.
4. Usando compreensão de listas, gere todos os pares `(x,y)` com `1 <= x < y <= 10` tais que `x + y` seja par.

---

## 📗 Lista 3: Funções de Alta Ordem e Composição

### Parte 1 – Aquecendo com HOFs

1. Defina a função `duasVezes :: (a -> a) -> a -> a` e teste com:
    - `duasVezes (*3) 2`
    - `duasVezes reverse [1,2,3]`
2. Usando **aplicação parcial**, defina:
    - `triplica = (*3)`
    - `mais10 = (+10)`
        
        Teste essas funções em valores diferentes.
        

---

### Parte 2 – Map

1. Use `map` para:
    - Somar 1 a todos os elementos da lista `[10,20,30]`
    - Converter todos os números de `[1..5]` em valores booleanos indicando se são pares.
2. Defina uma função `maiusculas :: [String] -> [String]` que transforma todas as palavras de uma lista em maiúsculas usando `map`.

---

### Parte 3 – Filter

1. Use `filter` para:
    - Selecionar apenas os números maiores que 100 da lista `[50,150,200,80,120]`.
    - Remover os espaços em branco de uma string.
2. Combine `map` e `filter`:
    
    Defina `quadradosPares :: [Int] -> [Int]` que devolve os quadrados apenas dos números pares de uma lista.
    

---

### Parte 4 – Fold (Reduce)

1. Reescreva com `foldr`:
    - A soma (`sum`) de uma lista
    - O produto (`product`) de uma lista
    - O tamanho (`length`) de uma lista
2. Defina uma função `concatena :: [String] -> String` usando `foldr` que junte todas as strings de uma lista em uma só.

---

### Parte 5 – Outras HOFs

1. Use:
    - `all` para verificar se todos os números de `[2,4,6,8]` são pares.
    - `any` para verificar se há algum múltiplo de 7 em `[10..20]`.
    - `takeWhile` para pegar os primeiros números pares de `[2,4,6,7,8,10]`.
    - `dropWhile` para descartar os primeiros números pares da mesma lista.

---

### Parte 6 – Composição

1. Defina `dobroMaisUm = (+1) . (*2)` e teste em `[1..5]` com `map`.
2. Escreva uma função `processaLista :: [Int] -> [Int]` que:
    - multiplique todos os números por 2,
    - some 1,
    - filtre apenas os números maiores que 10.

    Use **composição** para encadear os passos.

3. (Desafio) Defina `compose :: [a -> a] -> (a -> a)` que recebe uma lista de funções e retorna a composição delas.
    
    Teste com:
    

```haskell
compose [(+1), (*2), (^2)] 3
-- deve aplicar (^2), depois (*2), depois (+1)

```

---

---

