# Fundamentos em Clojure

## 1. Introdução e Conexão de Paradigmas

Seja bem-vindo(a) à sua primeira imersão em Clojure! 

Nas Unidades 1 e 2, você trabalhou com **Haskell**, uma linguagem de tipagem estática forte, pura e baseada em avaliação preguiçosa. Em Haskell, a segurança do código vem majoritariamente do compilador, que realiza verificações estritas de tipos antes mesmo de o código ser executado.

Agora, na **Unidade 3**, fazemos uma transição filosófica importante ao entrar no ecossistema **Lisp** com o **Clojure**. Embora ambas as linguagens compartilhem os pilares da programação funcional (como imutabilidade e funções como cidadãs de primeira classe), a abordagem do Clojure é diferente:

1. **Tipagem Dinâmica e Pragmática**: Clojure não realiza checagem de tipos em tempo de compilação da forma estrita do Haskell. Isso confere uma flexibilidade incrível e agilidade de prototipagem.
2. **Desenvolvimento Focado no REPL**: O REPL (Read-Eval-Print Loop) em Clojure não é apenas um local para testar funções (como o GHCi); ele é o próprio núcleo do desenvolvimento, onde você altera o programa em execução sem precisar reiniciar o processo.
3. **Código como Dados (Homoiconicidade)**: Em Clojure, a própria sintaxe do código é estruturada como listas e mapas. Isso abre as portas para uma metaprogramação absurdamente poderosa através de **Macros**. Em Haskell, gerar código exige uma extensão à parte (o *Template Haskell*, que usamos rapidamente no `$quickCheckAll` do capítulo de instalação); em Clojure, é natural, porque o código já é uma estrutura de dados.
4. **Ambiente Hospedado**: Clojure roda sobre a Java Virtual Machine (JVM), dando acesso nativo ao ecossistema robusto e maduro do Java.

Prepare-se para abrir sua mente e descobrir uma nova forma de projetar softwares funcionais, preditivos e altamente flexíveis.

### 1.1 O Que é Programação Funcional (e por que Clojure é especial?)

A programação funcional é mais do que um conjunto de técnicas; é uma maneira de pensar sobre a resolução de problemas com código. Em vez de dar ao computador uma sequência de passos que mudam o estado do programa, nós o instruímos através da composição de funções.

### 1.2. Computação como Funções Matemáticas

No coração do paradigma funcional está uma ideia simples, mas profunda. Como descrito em *Clojure in Action*, a programação funcional "trata a computação como a aplicação de funções matemáticas". Em um programa funcional, idealmente, não há estado que se altera arbitrariamente. Para um iniciante, isso significa que seu código se torna muito mais previsível. Uma função, quando chamada com os mesmos argumentos, sempre retornará o mesmo resultado, tornando o comportamento do programa mais fácil de entender e depurar.

### 1.3. O Pilar da Imutabilidade

Em Clojure, a imutabilidade é a regra, não a exceção. Isso significa que as estruturas de dados principais, como vetores e mapas, "uma vez criadas, não podem ser alteradas" (*Clojure in Action*).

Mas como programamos se não podemos mudar nada? A resposta é simples: quando uma "mudança" é necessária, Clojure não modifica a estrutura de dados original. Em vez disso, ele cria e retorna uma *nova* estrutura de dados que contém a alteração.

O principal benefício dessa abordagem é a segurança. O código torna-se imune a uma vasta classe de bugs relacionados à mutação não gerenciada e é inerentemente mais seguro para ser executado em ambientes concorrentes (com múltiplos threads), um desafio comum na programação moderna.

### 1.4. Por que escolher Clojure?

Clojure é uma linguagem de propósito geral que combina o melhor de vários mundos, tornando-a uma escolha atraente para uma ampla gama de tarefas.

- **Um Lisp Moderno:** Clojure pertence à família de linguagens Lisp, o que lhe confere uma de suas características mais poderosas: a capacidade de metaprogramação através de macros. Isso significa que você pode escrever código que escreve código, estendendo a própria linguagem.
- **Abraça o Paradigma Funcional:** Clojure não foi adaptado para a programação funcional; ele foi projetado desde o início com os princípios de imutabilidade e funções puras em seu núcleo.
- **Roda na JVM:** Por ser executado na Java Virtual Machine (JVM), Clojure tem acesso direto a um ecossistema vasto e maduro de bibliotecas e frameworks Java, combinando o poder de uma linguagem moderna com a estabilidade e o alcance de uma das plataformas mais testadas do mundo.

Agora que entendemos a teoria por trás de Clojure, é hora de colocá-la em prática e ter nossa primeira conversa com a linguagem.

## 2. Sua Primeira Conversa com Clojure: O REPL e a Sintaxe

Uma das melhores partes de aprender Clojure é a interatividade. Você não precisa escrever um programa inteiro para ver os resultados; você pode "conversar" com a linguagem em tempo real.

### 2.1. O REPL: Seu Ambiente Interativo

O **REPL** (Read-Eval-Print Loop ou Ciclo de Leitura-Avaliação-Impressão) é um ambiente interativo onde você pode digitar código Clojure, executá-lo e ver o resultado instantaneamente. O REPL é uma ferramenta fundamental no fluxo de trabalho de um desenvolvedor Clojure. Ele permite experimentação rápida, desenvolvimento incremental e oferece um feedback imediato, tornando o processo de aprendizado e desenvolvimento muito mais dinâmico e produtivo.

Podemos usar o [tryclojure.org](https://tryclojure.org/) como nosso REPL.

![image.png](img-clojure-fundamentos/image.png)

### 2.2. Preparando o Ambiente de Desenvolvimento

Para o desenvolvimento de sistemas, o REPL online não será suficiente; nesses casos, vamos precisar instalar o Clojure na nossa própria máquina.

1. Acesse o site oficial para as instruções de instalação: `https://clojure.org/guides/getting_started`.
2. Siga o processo de acordo com seu sistema operacional:
    - **Mac:** Utilize o Homebrew com o comando `brew install clojure`.
    - **Linux:** Baixe o arquivo TARGZ, descompacte-o e adicione o diretório `bin` ao seu PATH.
    - **Windows:** Utilize o instalador específico fornecido no site.
3. Após a instalação, abra seu terminal e execute o interpretador interativo (REPL).
4. Experiência na web https://tryclojure.org/ ou https://onecompiler.com/clojure

### 2.3. Desmistificando os Parênteses: A Sintaxe de Clojure

À primeira vista, o código Clojure pode parecer estranho por causa dos parênteses. No entanto, essa sintaxe, conhecida como S-expressions (expressões simbólicas), é a fonte de uma simplicidade e poder incríveis.

1. A estrutura fundamental é `(função argumento1 argumento2 ...)`.
2. Considere o exemplo `(+ 13 3)`. Isso é chamado de **notação prefixada**, onde a operação (ou função) vem *antes* dos operandos (ou argumentos). Em Clojure, `+` não é um operador especial; é apenas mais uma função. Essa uniformidade torna a linguagem mais simples e consistente.
3. Todas as operações seguem essa mesma lógica — inclusive imprimir na tela, como no exemplo abaixo.
4. Longe de serem um problema, os parênteses são, como afirma *Clojure in Action*, "uma grande fonte das capacidades de Clojure", pois permitem que o código seja tratado como uma estrutura de dados, algo que exploraremos mais adiante.

```clojure
(println "Bem vindo ao sistema de estoque")

```

Agora que você entende a estrutura básica de uma chamada de função, vamos ver como podemos dar nomes aos valores que usamos.

## 3. Dando Nomes às Coisas: Símbolos, Escopo e Keywords

Para escrever programas úteis, precisamos de uma forma de nomear valores e conceitos. Clojure nos oferece algumas ferramentas para isso, cada uma com um propósito específico.

### 3.1. Nomes Globais com `def`

A forma `def` é usada para criar nomes globais, chamados de "Vars". Esses nomes são definidos no nível mais alto do seu programa e são acessíveis de qualquer lugar. Use `def` para coisas que precisam ter um significado constante em todo o seu projeto.

```clojure
(def PI 3.14159)

```

### 3.2. Nomes Locais com `let`

A forma `let` é a maneira preferida e mais comum de criar nomes em Clojure. Diferente de `def`, `let` cria nomes com **escopo local**, o que significa que eles só existem dentro do bloco `let`.

De acordo com *Clojure for the Brave and True*, `let` oferece duas vantagens principais:

1. **Clareza:** Permite dar nomes a valores intermediários, tornando o código mais legível.
2. **Eficiência:** Permite que uma expressão seja avaliada apenas uma vez e seu resultado reutilizado várias vezes dentro do bloco.

```clojure
(def nome-global "Rosanthony")

(println nome-global)
; imprime: Rosanthony   (o valor de retorno de println é nil)

(let [nome-local "Bloodthunder"]
  (println nome-local)    ; imprime: Bloodthunder
  (println nome-global)) ; A variável global ainda é acessível aqui

;; A linha abaixo causaria um erro, pois 'nome-local' não existe fora do bloco 'let'.
;; (println nome-local)

```

!!! tip
    Veremos melhor os nomes locais quando criarmos nossas funções.

### 3.3. Keywords: Etiquetas Eficientes

Keywords (palavras-chave) são um tipo de dado único em Clojure. Elas começam com o caractere de dois-pontos (`:`), como em `:mochila` ou `:autor`. Pense nelas como etiquetas ou rótulos permanentes. A principal diferença para uma string é que uma string (ex: `"nome"`) é um dado que seu programa processa, enquanto uma keyword (ex: `:nome`) é como uma etiqueta que faz parte da estrutura do seu programa. Elas são extremamente eficientes e são a forma mais comum de usar chaves em mapas, uma das estruturas de dados mais importantes de Clojure.

Após aprender a nomear valores, o próximo passo lógico é aprender a nomear e criar blocos de código reutilizáveis: as funções.

## 4. A Essência do Código: Criando Suas Próprias Funções

As funções são os blocos de construção fundamentais de qualquer programa Clojure. É através delas que a lógica do seu programa toma forma.

### 4.1. A Anatomia de uma Função com `defn`

Usamos o macro `defn` para definir uma nova função. Sua estrutura básica é muito clara:

`(defn nome-da-funcao [parametros] (corpo-da-funcao))`

Vamos analisar um exemplo simples:

```clojure
(defn total-cost [item-cost number-of-items]
  (* item-cost number-of-items))

```

Neste exemplo, `total-cost` é o nome da função, `item-cost` e `number-of-items` são seus parâmetros. Um ponto crucial em Clojure é que **não existe a palavra-chave `return`**. A função automaticamente retorna o valor da última expressão avaliada em seu corpo.

### 4.2. Funções Puras vs. Funções com Efeitos Colaterais

Em programação funcional, é útil distinguir entre dois tipos de funções:

| Funções Puras | Funções com Efeitos Colaterais |
| --- | --- |
| **Definição:** São funções cujo resultado depende *apenas* de suas entradas e que não interagem com o mundo exterior (não têm efeitos colaterais). | **Definição:** Funções que interagem com o mundo exterior, como imprimir no console, escrever em um arquivo ou fazer uma requisição de rede. |
| **Benefício:** São altamente previsíveis, fáceis de testar e de raciocinar sobre seu comportamento. | **Quando usar:** São necessárias sempre que seu programa precisa interagir com o usuário ou outros sistemas. |
| **Exemplo:** A função `total-cost` que definimos acima é uma função pura. | **Exemplo:** A função `println`, que imprime texto no console. |

### 4.3. Múltiplas Aridades: Uma Função, Várias Formas

Clojure permite que uma única função tenha diferentes comportamentos dependendo do número de argumentos (ou "aridade") que recebe. Isso é conhecido como sobrecarga de aridade.

```clojure
(defn minha-funcao
  ;; Corpo para 0 argumentos
  ([]
   "Fui chamada sem argumentos!")

  ;; Corpo para 1 argumento
  ([x]
   (str "Recebi o argumento: " x))

  ;; Corpo para 2 argumentos
  ([x y]
   (str "Recebi dois argumentos: " x " e " y)))

(println (minha-funcao))          ; imprime: Fui chamada sem argumentos!
(println (minha-funcao "olá"))    ; imprime: Recebi o argumento: olá
(println (minha-funcao 1 2))      ; imprime: Recebi dois argumentos: 1 e 2

```

Com a capacidade de criar funções, precisamos agora de ferramentas para controlar o fluxo de execução dentro delas, tomando decisões baseadas em certas condições.

Podemos testar no onecompiler.com

![image.png](img-clojure-fundamentos/image%201.png)

## 5. Tomando Decisões no Código

Lógica condicional é a espinha dorsal de qualquer programa não trivial. Clojure oferece formas simples e poderosas para isso.

### 5.1. O Fundamental: `if`

O `if` em Clojure é uma expressão, o que significa que ele sempre retorna um valor. Sua estrutura é direta:

`(if teste-condicional expressao-se-verdadeiro expressao-se-falso)`

Se `teste-condicional` for verdadeiro, ele avalia e retorna `expressao-se-verdadeiro`. Caso contrário, avalia e retorna `expressao-se-falso`.

```clojure
(defn verificar-maioridade [idade]
  (if (>= idade 18)
    "Você é maior de idade."
    "Você é menor de idade."))

```

**Uso:**

```clojure
(verificar-maioridade 20)
;; => "Você é maior de idade."

(verificar-maioridade 15)
;; => "Você é menor de idade."

```

### 5.2. A Verdade em Clojure: `false` e `nil`

O conceito de "verdade" (ou *truthiness*) em Clojure é extremamente simples e consistente. Como diz *Getting Clojure*:

"Em um [...] contexto booleano, **apenas `false` e `nil` são tratados como falsos**. Todo o resto é tratado como verdadeiro."

Isso significa que números (incluindo `0`), strings (incluindo as vazias `""`), e coleções vazias (como `[]` ou `{}`) são todos considerados **verdadeiros** em um teste condicional.

```clojure
(defn verificar-verdade [valor]
  (if valor
    "Isto é tratado como VERDADEIRO."
    "Isto é tratado como FALSO."))

```

**Uso:**

```clojure
(verificar-verdade 0)
;; => "Isto é tratado como VERDADEIRO."

(verificar-verdade "")
;; => "Isto é tratado como VERDADEIRO."

(verificar-verdade nil)
;; => "Isto é tratado como FALSO."

(verificar-verdade false)
;; => "Isto é tratado como FALSO."

```


### 5.3. Predicados e a Convenção

Uma **função predicado** é simplesmente uma função que retorna um valor booleano (`true` ou `false`). Por convenção, em Clojure, os nomes dessas funções terminam com um ponto de interrogação (`?`).

Exemplos comuns incluem:

- `zero?`: Retorna `true` se o número for zero.
- `empty?`: Retorna `true` se uma coleção estiver vazia.
- `string?`: Retorna `true` se o valor for uma string.

```clojure
(defn analisar-colecao [col]
  (if (empty? col)
    "A coleção está vazia."
    "A coleção contém elementos."))

```

**Uso:**

```clojure
(analisar-colecao [])
;; => "A coleção está vazia."

(analisar-colecao [1 2 3])
;; => "A coleção contém elementos."

(string? "abc")
;; => true

(zero? 0)
;; => true

(zero? 10)
;; => false

```

Uma variação útil é o `if-let`: ele combina `let` e `if`, ligando um nome a um valor e executando o primeiro ramo apenas se esse valor não for `nil` nem `false`:

```clojure
(defn saudacao [nome]
  (if-let [n nome]
    (str "Olá, " n "!")
    "Olá, visitante!"))

```

**Uso:**

```clojure
(saudacao "Ana")
;; => "Olá, Ana!"

(saudacao nil)
;; => "Olá, visitante!"

```

### 5.4. `when`: Para Casos sem "senão"

Às vezes, você só quer executar uma ação se uma condição for verdadeira, sem se preocupar com o caso "senão". Para isso, Clojure oferece o `when`, uma forma mais concisa de `if` sem a cláusula `else` — e que aceita várias expressões no corpo.

**Exemplo 1 – Imprimir apenas se o número for positivo**

```clojure
(defn mostrar-se-positivo [n]
  (when (> n 0)
    (println n "é positivo.")))

```

**Uso:**

```clojure
(mostrar-se-positivo 10)
;; => Imprime: 10 é positivo.

(mostrar-se-positivo -5)
;; => Não imprime nada

```

**Exemplo 2 – Atualizar estado apenas se uma chave existir**

```clojure
(defn atualizar-nome [usuario]
  (when (:nome usuario)
    (println "Atualizando nome para:" (:nome usuario))
    ;; outras ações poderiam vir aqui
    true))

```

**Uso:**

```clojure
(atualizar-nome {:nome "Carlos"})
;; => Imprime: Atualizando nome para: Carlos

(atualizar-nome {:idade 30})
;; => Não imprime nada

```

Repare que o corpo do `when` tem várias expressões (um `println`, outras ações, o retorno `true`), algo que o `if` não permite sem um `do`.

---

**Exemplo 3 – Combinar com Predicados**

```clojure
(defn alertar-se-vazio [colecao]
  (when (empty? colecao)
    (println "Atenção: a coleção está vazia!")))

```

**Uso:**

```clojure
(alertar-se-vazio [])
;; => Imprime: Atenção: a coleção está vazia!

(alertar-se-vazio [1 2 3])
;; => Não imprime nada

```

Recapitulando

- `if` → usa-se quando há **caso "senão"**.
- `when` → para **executar algo apenas se a condição for verdadeira**, com possibilidade de múltiplas expressões.

Com todos esses blocos de construção em mãos — sintaxe, nomes, funções e condicionais — estamos prontos para descobrir o conceito mais poderoso e distintivo de Clojure e da família Lisp.

## 6. O Superpoder do Clojure: Homoiconicidade (Código como Dados)

Este é o conceito que, uma vez compreendido, revela a verdadeira magia por trás das linguagens Lisp.

### 6.1. O que significa "Código como Dados"?

Homoiconicidade (do grego *homo*, que significa "o mesmo", e *icon*, que significa "representação") é uma propriedade de linguagens de programação onde "o código é escrito nas próprias estruturas de dados da linguagem" (*Clojure in Action*).

Para um iniciante, isso se traduz em algo muito concreto: a mesma sintaxe que você usa para criar uma lista de dados é a sintaxe que você usa para escrever seu código. Uma lista de Clojure é um conjunto de itens entre parênteses, e uma chamada de função também é um conjunto de itens entre parênteses.

### 6.2. A Diferença entre Dados e Execução

Como Clojure sabe quando uma lista é apenas uma lista e quando é um código para executar? A resposta está no `quote` (ou seu atalho, a apóstrofo `'`).

```clojure
;; A apóstrofo (') "cita" a lista, tratando-a como dados.
;; O resultado é a própria lista.
'(+ 1 2)
; => (+ 1 2)

;; Sem a apóstrofo, a lista é executada como código.
;; O resultado é a soma.
(+ 1 2)
; => 3

```

O `quote` instrui Clojure a *não avaliar* a expressão seguinte, mas sim a tratá-la como uma estrutura de dados literal.

### 6.3. Por que isso é importante?

A homoiconicidade é o que torna a metaprogramação em Clojure tão natural e poderosa. Como o seu código é apenas uma estrutura de dados (uma lista), você pode escrever programas que criam, analisam e manipulam outros programas com a mesma facilidade com que manipula dados. Essa capacidade é a base dos **macros**, que permitem estender a sintaxe da linguagem de formas que são impossíveis em muitas outras linguagens.

Parabéns! Você completou uma introdução aos conceitos fundamentais de Clojure. Estes são os alicerces sobre os quais todo o resto é construído.

## 7. Conclusão: Sua Jornada Apenas Começou

Você deu os primeiros e mais importantes passos no mundo da programação funcional com Clojure. Mais do que uma coleção de funcionalidades, os conceitos que você aprendeu formam um sistema coeso onde cada parte fortalece as outras. A **sintaxe uniforme** de Lisp é o que torna a **homoiconicidade** (código como dados) uma realidade prática. Essa propriedade, por sua vez, permite que a linguagem seja construída em torno de **funções** como blocos de construção primários. A ênfase em **imutabilidade** e no **escopo local** com `let` garante que essas funções sejam previsíveis e seguras. Vamos revisar como esses pilares se sustentam mutuamente:

1. **Imutabilidade:** Os dados não mudam. Em vez disso, novas versões são criadas, levando a um código mais seguro e previsível.
2. **Sintaxe Uniforme:** A notação prefixada `(função ...)` aplica-se a tudo, desde operações matemáticas a chamadas de função, simplificando a gramática da linguagem.
3. **Funções são o Coração:** A principal atividade em Clojure é criar pequenas e reutilizáveis funções, preferencialmente puras, usando `defn`.
4. **Escopo Local é Rei:** Usar `let` para criar nomes locais é a prática padrão para manter o código limpo, organizado e livre de efeitos colaterais inesperados.
5. **Código é Dados:** A homoiconicidade é o "superpoder" de Clojure, permitindo uma metaprogramação poderosa e uma expressividade única.

Estes são os blocos de construção essenciais. Com essa base sólida, você está mais do que preparado(a) para explorar as estruturas de dados de Clojure, seu modelo de concorrência e o vasto ecossistema de bibliotecas à sua disposição. A jornada é desafiadora, mas incrivelmente recompensadora. Continue curioso(a) e continue codificando!

## 8. Prática Guiada

### 8.1. Exercício 1: Operações Simples com Símbolos

Vamos reforçar os conceitos de `def` e notação prefixada.

1. **Passo 1:** Defina um símbolo global `preco-base` com o valor `85` usando `def`.
2. **Passo 2:** Defina outro símbolo global `taxa-fixa` com o valor `15` usando `def`.
3. **Passo 3:** Use a notação prefixada para somar os dois símbolos e imprima o resultado com `println`.

### Código da Solução

```clojure
(def preco-base 85)
(def taxa-fixa 15)
(println (+ preco-base taxa-fixa))

```

### 8.2. Exercício 2: Função com Escopo Local

Agora, vamos praticar a definição de funções com escopo local usando `defn` e `let`.

1. **Passo 1:** Defina uma função chamada `calcula-total-com-desconto` que recebe um parâmetro `valor-bruto`.
2. **Passo 2:** Dentro da função, use `let` para definir um símbolo local `desconto-fixo` com o valor `10`.
3. **Passo 3:** O corpo do `let` deve calcular e retornar a subtração do `desconto-fixo` do `valor-bruto`.
4. **Passo 4:** Invoque a função com o valor `100` e imprima o resultado na tela usando `println`.

### Código da Solução

```clojure
(defn calcula-total-com-desconto [valor-bruto]
  (let [desconto-fixo 10]
    (- valor-bruto desconto-fixo)))

;; Invocando a função
(println (calcula-total-com-desconto 100))

```