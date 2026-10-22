# Exercícios Fundamentais de Clojure: Do Básico à Prática

## Introdução

Este documento foi elaborado para guiar você na construção de uma compreensão sólida e prática dos conceitos fundamentais de Clojure. Através de uma série de exercícios cuidadosamente selecionados, você explorará desde a sintaxe básica e a manipulação de dados até a criação de funções e o uso de abstrações poderosas.

A ênfase é colocada na execução de cada exercício em um ambiente REPL (Read-Eval-Print Loop). Mais do que uma simples ferramenta, o REPL é um pilar do desenvolvimento em Clojure que fomenta um ciclo de edição orientado por feedback (feedback-driven editing). Em vez do rígido ciclo de compilação-execução-depuração de outras linguagens, o REPL permite uma conversa interativa com seu programa em execução, um alicerce da lendária produtividade das linguagens Lisp.

## 1. O Básico: REPL, Tipos e Operadores

Todo desenvolvimento em Clojure, em sua essência, começa com a manipulação de dados. Antes de construirmos lógicas complexas, precisamos nos sentir confortáveis com os blocos de construção fundamentais da linguagem: seus tipos de dados. Nesta seção, vamos explorar os tipos primitivos, como números e strings, e as coleções mais comuns, como vetores e mapas.

Um dos aspectos mais elegantes de Clojure é sua sintaxe uniforme baseada em S-expressions (Symbolic Expressions). Embora os parênteses possam parecer incomuns a princípio, eles fornecem uma estrutura simples e consistente para todas as operações na forma `(operador operando1 operando2)`, um design deliberado e poderoso que evita a mistura de notações infixas, prefixas e palavras-chave especiais encontradas em outras linguagens. Vamos colocar isso em prática.

### Exercício 1: Operações Aritméticas no REPL

Esta primeira tarefa demonstra a notação de prefixo e a avaliação aninhada de expressões, que são universais em Clojure.

**Tarefa:** Calcule o resultado da expressão matemática `(100 * (5 + 3)) / 8` diretamente no seu REPL.

### Exercício 2: Manipulação de Strings

Strings são dados onipresentes, e juntá-las é uma operação trivial, mas essencial.

**Tarefa:** Utilize a função `str` para concatenar as strings `"Clojure é uma linguagem "`, `"funcional "`, e `"poderosa."` em uma única frase.

### Exercício 3: Criando e Acessando Vetores

Vetores são coleções ordenadas e indexadas, ideais para listas de itens.

**Tarefa:** Use as funções `first` e `last` para extrair o primeiro e o último item, respectivamente, do vetor `["Aprender Clojure" "Construir um projeto" "Dominar programação funcional"]`. Tente também acessar o primeiro item usando a função `nth`.

### Exercício 4: Criando e Acessando Mapas

Mapas são coleções de pares chave-valor, perfeitos para representar entidades estruturadas. Em Clojure, é idiomático usar keywords (palavras-chave, como `:titulo`) para as chaves.

**Tarefa:**

1. Defina um mapa que represente um livro com as chaves `:titulo`, `:autor`, e `:ano`.
2. Extraia o valor da chave `:autor` utilizando duas técnicas diferentes (por exemplo, usando `get` e usando a keyword como função).

### Exercício 5: Modificando um Mapa (Imutavelmente)

Um conceito central em Clojure é a imutabilidade. As estruturas de dados nunca são modificadas no lugar. Em vez disso, funções como `assoc` retornam uma *nova* estrutura de dados com a alteração desejada, preservando a original intacta.

**Tarefa:** Usando o mapa do exercício anterior, crie um novo mapa que inclua a chave `:genero` com o valor `"Programação"`. Utilize a função `assoc`. Após a operação, verifique o conteúdo do mapa original para confirmar que ele não foi alterado.

## 2. Criando Nomes: Bindings Globais e Locais

À medida que nossos programas crescem, precisamos de maneiras de nomear valores. Clojure faz uma distinção clara entre nomes permanentes e nomes temporários. Para nomes permanentes, como constantes ou funções, usamos `def` para criar uma Var. Para nomes temporários que existem apenas dentro de uma expressão específica, usamos `let`, que cria um escopo léxico.

### Exercício 6: Definindo uma Constante com `def`

Use `def` para valores que devem ser referenciados globalmente, como constantes de configuração.

**Tarefa:** Defina uma Var global chamada `taxa-de-juros` com o valor `0.05`. Em seguida, escreva uma expressão que a utilize para calcular os juros sobre um principal de `1000`.

### Exercício 7: Usando `let` para Bindings Locais

Use `let` para dar nomes a resultados intermediários, tornando cálculos complexos mais legíveis e eficientes ao evitar recálculos.

**Tarefa:** Calcule o preço total de um produto, aplicando um desconto e depois um imposto. Use um formulário `let` para criar bindings locais para `valor-desconto` e `preco-com-desconto`, e então use-os para calcular o preço final (resultado esperado: aproximadamente `144.45`).

- `preco-base`: 150.0
- `percentual-desconto`: 0.10 (10%)
- `percentual-imposto`: 0.07 (7%)

## 3. A Essência do Funcional: Criando e Usando Funções

Funções são os blocos de construção primários em Clojure. Elas encapsulam lógica e transformam dados. Usamos a macro `defn` para criar funções nomeadas. O verdadeiro poder do paradigma funcional se revela com as funções de ordem superior (higher-order functions), como `map`, `filter` e `reduce`, que operam sobre a abstração de sequência.

### Exercício 8: Criando uma Função Nomeada com `defn`

Vamos criar uma função simples para encapsular uma fórmula matemática.

**Tarefa:** Crie uma função chamada `calcular-area-circulo` que recebe um `raio` como parâmetro e retorna a área do círculo (π * r²). Use a constante `Math/PI` do Java, que está diretamente acessível em Clojure. Teste sua função com um raio de `10`.

### Exercício 9: Usando `map` e Funções Anônimas

A função `map` aplica uma função a cada item de uma coleção para produzir uma nova sequência de resultados.

**Tarefa:** Dado o vetor `[1 2 3 4 5]`, use `map` para criar uma nova sequência onde cada número é o dobro do original. Demonstre o uso de uma função anônima com a sintaxe completa `(fn ...)` e a sintaxe abreviada `#(...)`.

### Exercício 10: Filtrando Dados com `filter`

A função `filter` recebe uma função predicado (que retorna `true` ou `false`) e uma coleção, retornando uma nova sequência contendo apenas os itens aprovados.

**Tarefa:** Dado o vetor de pessoas abaixo, use `filter` para encontrar apenas aquelas com 18 anos ou mais.

```clojure
(def pessoas [{:nome "Ana" :idade 25}
              {:nome "Bruno" :idade 17}
              {:nome "Carla" :idade 32}
              {:nome "Daniel" :idade 16}])

```

### Exercício 11: Agregando Dados com `reduce`

A função `reduce` é usada para "reduzir" uma coleção a um único valor, acumulando um resultado.

**Tarefa:** Dado o vetor `[10 20 30 40]`, use `reduce` para calcular a soma total de todos os elementos.

## 4. Estrutura e Clareza: Desestruturação (Destructuring)

A desestruturação é uma técnica idiomática em Clojure que permite extrair valores de estruturas de dados (como vetores e mapas) de forma concisa. Ela pode ser usada em `let` e diretamente na lista de parâmetros de uma função.

### Exercício 12: Desestruturação de Vetor com `let`

**Tarefa:** Dado o vetor `["João" "Silva" 30]`, use `let` com desestruturação de vetor para extrair os valores para os bindings `nome`, `sobrenome` e `idade`. Em seguida, construa a frase `"O usuário João Silva tem 30 anos."`.

### Exercício 13: Desestruturação de Mapa em Argumentos de Função

A desestruturação é especialmente útil em listas de parâmetros de funções.

**Tarefa:** Crie uma função `descrever-produto` que recebe um mapa de produto (ex: `{:nome "Notebook" :preco 3500.0 :estoque 50}`). Use a desestruturação de mapa na lista de parâmetros para extrair os valores das chaves `:nome` e `:preco` e retornar uma string formatada como `"O produto Notebook custa R$ 3500.0."`.

## 5. Exercícios de Síntese: Combinando Conceitos

Os exercícios finais desafiam você a combinar os múltiplos conceitos aprendidos até agora.

### Exercício 14: Processando uma Lista de Compras

Este problema clássico combina a transformação de dados (`map`) com a agregação (`reduce`).

**Tarefa:** Dada a lista de compras abaixo, escreva uma função `calcular-total` que calcule o custo total. A função deve:

1. Usar `map` para calcular o subtotal de cada item (`:preco` * `:quantidade`).
2. Usar `reduce` para somar todos os subtotais.

```clojure
(def lista-de-compras
  [{:produto "Maçã" :preco 2.5 :quantidade 4}
   {:produto "Pão" :preco 5.0 :quantidade 1}
   {:produto "Leite" :preco 4.2 :quantidade 2}])

```

### Exercício 15: Função Variádica para Estatísticas Simples

Funções variádicas são aquelas que aceitam um número variável de argumentos (usando `&`).

Na lista de parâmetros, o `&` indica que todos os argumentos restantes serão reunidos em uma sequência: em `(defn f [& nums] ...)`, a chamada `(f 1 2 3)` liga `nums` a `(1 2 3)`. Sem nenhum argumento, `nums` vale `nil` (e `(count nil)` e `(reduce + nil)` valem `0`).

**Tarefa:** Crie uma função variádica chamada `estatisticas` que aceita qualquer número de argumentos numéricos. Ela deve retornar um mapa contendo o número de argumentos (`:contagem`), a soma total (`:soma`) e a média (`:media`). Teste-a chamando `(estatisticas 10 20 30 40 50)`, que deve resultar em `{:contagem 5, :soma 150, :media 30}`, e `(estatisticas)`. Cuidado: no segundo caso, calcular a média diretamente dá divisão por zero. Faça a função retornar `{:contagem 0, :soma 0, :media 0}`.

## Conclusão

Parabéns por completar esta série de exercícios! Você agora possui uma base sólida nos pilares da programação em Clojure: a manipulação de estruturas de dados imutáveis, a criação de bindings, a definição de funções, a abstração de sequência e o poder das funções de ordem superior. Estes são os fundamentos sobre os quais toda a lógica em Clojure é construída.

Continue a explorar a vasta biblioteca padrão da linguagem e a abraçar a filosofia de uma "linguagem de programação programável". A jornada para a maestria funcional é recompensadora, e você deu os primeiros passos mais importantes.