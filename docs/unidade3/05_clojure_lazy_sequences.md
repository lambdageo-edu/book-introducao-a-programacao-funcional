# Trabalhando com Coleções: De Lazy Sequences à Recursão Otimizada

Bem-vindo a mais um capítulo em nossa jornada com Clojure. Hoje, vamos nos aprofundar em três pilares essenciais para a manipulação de dados na linguagem. Primeiro, exploraremos como agrupar e resumir dados utilizando poderosas funções de agregação, transformando coleções brutas em insights valiosos. Em seguida, aprenderemos a ordenar e selecionar esses dados, uma etapa crucial para apresentar informações de forma clara e significativa. Por fim, desvendaremos o conceito de *lazy sequences* (sequências preguiçosas), um dos segredos por trás da eficiência e do excelente gerenciamento de memória de Clojure. Dominar essas três áreas permitirá que você construa pipelines de dados expressivos, legíveis e de alto desempenho.

## 1. Agrupamento e Agregação de Dados

### 1.1. A Importância Estratégica da Agregação

Agrupar dados é um passo fundamental em qualquer processo de análise de informações. Raramente os dados brutos, em sua forma original, nos contam a história completa. A agregação nos permite transformar grandes coleções de registros individuais em resumos concisos e significativos. Ao agrupar dados por atributos comuns — como categorias de produtos, departamentos ou datas — podemos calcular métricas como totais, médias e contagens, revelando padrões e insights que, de outra forma, permaneceriam ocultos na massa de informações.

Imagine que você recebeu dados de **várias frases**, e já fez uma análise inicial que retornou, para cada palavra, o número de vezes que ela aparece em uma parte do texto.

Esses dados vieram **em partes**, como pequenas listas de contagens:

```clojure
(def dados-por-frase '([["era" 1] ["uma" 1]]
                       [["vez" 1] ["um" 1] ["rei" 1]]
                       [["era" 1] ["um" 1]]))

```

**Objetivo:**

Agrupar todas as ocorrências da mesma palavra, para depois poder somar as contagens.

---

**Passo 1 – Juntar todas as sublistas em uma única sequência**

```clojure
(def dados-unificados (apply concat dados-por-frase))

```

Agora `dados-unificados` é:

```clojure
(["era" 1] ["uma" 1] ["vez" 1] ["um" 1] ["rei" 1] ["era" 1] ["um" 1])

```

---

**Passo 2 – Agrupar os pares pela palavra (a primeira posição de cada vetor)**

```clojure
(group-by first dados-unificados)

```

Resultado:

```clojure
{"era"  [["era" 1] ["era" 1]],
 "uma"  [["uma" 1]],
 "vez"  [["vez" 1]],
 "um"   [["um" 1] ["um" 1]],
 "rei"  [["rei" 1]]}

```

---

**Explicação:**

- Cada item da lista é um par `["palavra" 1]`.
- A função `group-by` agrupa esses pares, **usando a palavra como chave** (ou seja, o `first` de cada vetor).
- O resultado é um **mapa onde cada chave é uma palavra**, e o valor é uma **lista de todas as vezes que essa palavra apareceu** nos dados originais.

**O Que Fazer Depois?**

Você pode somar as contagens:

```clojure
(def agrupado (group-by first dados-unificados))

(def contagens
  (into {}
        (for [[palavra ocorrencias] agrupado]
          [palavra (reduce + (map second ocorrencias))])))

```

Resultado:

```clojure
{"era" 2, "uma" 1, "vez" 1, "um" 2, "rei" 1}

```

### 1.2. A Função `group-by`

A principal ferramenta de Clojure para agrupamento é a função `group-by`. Sua operação é elegantemente simples: ela recebe uma função (a função de agrupamento) e uma coleção. Em seguida, retorna um mapa onde as chaves são o resultado da aplicação da função a cada item da coleção, e os valores são listas contendo todos os itens que geraram aquela chave específica.

O exemplo de contagem de palavras da seção anterior, inspirado no paradigma map/reduce, já mostrou seu funcionamento: no Passo 2, `group-by` agrupou os pares `[palavra contagem]` usando a primeira posição de cada vetor como chave.

### 1.3. Caso de Uso: Análise de Carrinhos de Compras

Vamos aplicar esses conceitos a um cenário mais robusto: analisar o faturamento de uma loja online a partir de uma coleção de carrinhos de compra. Cada carrinho contém múltiplos itens, e cada item tem informações sobre seu departamento e preço.

**Estrutura de Dados de Exemplo:**

```clojure
(def carts
  [{:settled? true
    :line-items [{:dept :toys :total 10}
                 {:dept :clothing :total 25}]}
   {:settled? false ; Carrinho abandonado, deve ser ignorado
    :line-items [{:dept :kitchen :total 50}]}
   {:settled? true
    :line-items [{:dept :toys :total 15}]}])

```

Nosso objetivo é calcular o faturamento total por departamento. Para isso, construiremos um pipeline de processamento passo a passo.

1. **Filtragem com `filter`**: Primeiro, selecionamos apenas os carrinhos que foram finalizados (`:settled? true`).
2. **Extração com `mapcat`**: Em seguida, extraímos todos os `:line-items` dos carrinhos finalizados e os colocamos em uma única sequência. `mapcat` é perfeito para isso, pois aplica uma função (`:line-items`) e concatena os resultados.
3. **Transformação com `map`**: Antes de agrupar, é uma boa prática simplificar os dados. Usaremos `map` com uma função auxiliar, `line-summary`, para transformar cada item de linha em um resumo contendo apenas os dados de que precisamos: o departamento (`:dept`) e o total (`:total`). Isso descarta informações irrelevantes e torna o próximo passo mais eficiente.
4. **Agrupamento com `group-by`**: Agora que temos uma sequência limpa e simplificada de itens, aplicamos `group-by` para agrupá-los por departamento (`:dept`).

A função auxiliar `line-summary` é simplesmente uma seleção de chaves:

```clojure
(defn line-summary [item]
  (select-keys item [:dept :total]))
```

Vamos combinar esses passos em um pipeline legível usando o macro `->>`:

```clojure
(->> carts
     (filter :settled?)
     (mapcat :line-items)
     (map line-summary)
     (group-by :dept))

```

O resultado neste ponto é um mapa onde cada chave é um departamento e o valor é uma lista de todos os resumos de itens daquele departamento:

```clojure
{:toys     [{:dept :toys, :total 10} {:dept :toys, :total 15}],
 :clothing [{:dept :clothing, :total 25}]}

```

Falta o último passo, **agregação com `update-vals` e `reduce`**. Por fim, precisamos somar os totais de cada departamento. A função `update-vals` (disponível desde o Clojure 1.11) aplica uma função a **cada valor** de um mapa, mantendo as chaves. Combinada com `reduce`, ela soma os totais de cada grupo:

```clojure
(def agrupado
  (->> carts
       (filter :settled?)
       (mapcat :line-items)
       (map line-summary)
       (group-by :dept)))

(update-vals agrupado (fn [resumos] (reduce + (map :total resumos))))
```

O mapa final nos dá um resumo valioso dos dados brutos, mostrando o faturamento total para cada departamento.

```clojure
{:toys 25, :clothing 25}

```

Juntando tudo, a função completa seria:

```clojure
(defn revenue-by-department [carts]
  (update-vals
    (->> carts
         (filter :settled?)
         (mapcat :line-items)
         (map line-summary)
         (group-by :dept))
    (fn [summaries] (reduce + (map :total summaries)))))

(revenue-by-department carts)
;; => {:toys 25, :clothing 25}
```

Repare que `update-vals` fica **fora** do `->>`: ela recebe o mapa como **primeiro** argumento, e o `->>` insere o resultado sempre na **última** posição. (Em versões anteriores ao Clojure 1.11, o mesmo papel era feito por `map-vals`, da biblioteca externa Medley.)

Com os dados devidamente agregados, o próximo passo lógico é organizá-los para facilitar a visualização e a tomada de decisões.

--------------------------------------------------------------------------------

## 2. Ordenação e Manipulação de Coleções

### 2.1. A Necessidade de Ordenar e Selecionar Dados

Após a agregação, os dados estão resumidos, mas não necessariamente organizados de uma maneira útil. A ordenação é o passo que nos permite responder a perguntas como "Quais são os departamentos mais lucrativos?" ou "Quais os 10 produtos menos vendidos?". Selecionar subconjuntos de dados, como os "Top 5", é igualmente importante para focar a atenção nos resultados mais relevantes. Clojure oferece um rico conjunto de funções para realizar essas tarefas de forma concisa e eficiente.

### 2.2. Ordenação com `sort-by` e Inversão com `reverse`

A função `sort-by` é a ferramenta ideal para ordenar coleções com base em um critério específico. Ela recebe uma função-chave e uma coleção, e ordena a coleção em ordem *crescente* com base no valor retornado pela função-chave para cada elemento.

Consideremos uma coleção de livros, onde cada livro é um mapa com título e avaliação (`:rating`):

```clojure
(def books
  [{:title "Jaws", :rating 4.5}
   {:title "Emma", :rating 4.8}
   {:title "2001", :rating 4.2}])

;; Ordena os livros pela avaliação, do menor para o maior
(sort-by :rating books)

```

O resultado será uma sequência de livros ordenada de 4.2 a 4.8.

Para obter uma ordem *decrescente* (do maior para o menor), simplesmente combinamos `sort-by` com a função `reverse`, que inverte a ordem de uma sequência.

```clojure
;; Ordena os livros do mais bem avaliado para o pior
(reverse (sort-by :rating books))

```

### 2.3. Acesso Posicional e Limitação de Resultados

Clojure fornece funções simples para acessar elementos específicos de uma sequência ou limitar o número de resultados.

- **`first`**: Retorna o primeiro item de uma coleção.
- **`nth`**: Retorna o item em um índice específico (base zero).
- **`take`**: Retorna os primeiros `n` itens de uma coleção. É importante notar que `take` retorna uma *sequência preguiçosa* (lazy sequence), o que a torna extremamente eficiente.

Podemos combinar `take` com nosso exemplo de ordenação para obter os "Top 3" livros mais bem avaliados:

```clojure
(take 3 (reverse (sort-by :rating books)))

```

### 2.4. Combinando Operações com o Macro `->>`

Pipelines de dados como o que acabamos de construir podem se tornar difíceis de ler devido ao aninhamento de funções: `(take 3 (reverse (sort-by :rating books)))`. Para resolver isso, Clojure oferece o macro de encadeamento `->>` (conhecido como *thread-last*). Ele reescreve o código aninhado em uma sequência linear e legível, de cima para baixo. O macro insere o resultado de cada linha como o *último* argumento da função na linha seguinte.

Vamos refatorar nosso exemplo completo para obter os títulos dos três livros mais bem avaliados:

```clojure
(->> books
     (sort-by :rating)  ; (sort-by :rating books)
     reverse            ; (reverse <resultado-anterior>)
     (take 3)           ; (take 3 <resultado-anterior>)
     (map :title))      ; (map :title <resultado-anterior>)

```

Este código é funcionalmente idêntico ao aninhado, mas sua legibilidade é muito superior. Ele descreve claramente a sequência de transformações aplicadas aos dados.

Mas como Clojure consegue executar esses pipelines de forma tão eficiente, especialmente com coleções potencialmente enormes? A resposta está em um dos conceitos mais poderosos da linguagem: as *Lazy Sequences*.

## 3. A Eficiência das Lazy Sequences (Sequências Preguiçosas)

### 3.1. O que são Lazy Sequences?

A maioria das linguagens de programação opera de forma *gulosa* (*eager evaluation*). Quando você aplica uma série de transformações a uma coleção, cada etapa é executada completamente, gerando uma coleção intermediária em memória, antes de passar para a próxima. Se você mapear, filtrar e depois pegar os 10 primeiros itens de uma lista de um milhão de elementos, uma abordagem gulosa criaria duas listas intermediárias de um milhão de elementos cada.

Clojure, por outro lado, favorece a *avaliação preguiçosa* (*lazy evaluation*). Muitas de suas funções de sequência, como `map` e `filter`, não calculam os elementos imediatamente. Em vez disso, elas retornam um objeto de sequência "preguiçoso" que sabe como calcular os elementos *sob demanda*. Um elemento da sequência só é efetivamente calculado quando alguma outra parte do programa realmente precisa dele. Isso resulta em enormes benefícios de desempenho e gerenciamento de memória, pois evita a criação de coleções intermediárias desnecessárias. É importante notar que, enquanto funções de transformação como `map` e `filter` são preguiçosas, funções que precisam consumir a sequência inteira para produzir um único valor, como `reduce`, são inerentemente gulosas (eager).

### 3.2. Lidando com o Infinito

Uma das consequências mais poderosas da avaliação preguiçosa é a capacidade de representar e manipular coleções infinitas. Como os elementos são calculados apenas quando necessários, podemos definir uma sequência que, teoricamente, nunca termina.

Funções como `range` sem argumentos ou `repeat` criam sequências infinitas:

```clojure
(range)    ;; Gera (0 1 2 3 4 5 ...)
(repeat 1) ;; Gera (1 1 1 1 1 1 ...)

```

Tentar realizar uma operação gulosa em uma sequência infinita resultaria em um loop infinito e esgotamento de memória. No entanto, em Clojure, é perfeitamente seguro combinar uma sequência infinita com uma função como `take`:

```clojure
(take 5 (range)) ;; Retorna (0 1 2 3 4)

```

Isso funciona porque `take` só precisa dos primeiros 5 itens. Ele solicita um item de cada vez da sequência `range`, e assim que obtém o quinto item, ele para. O restante da sequência infinita nunca é calculado.

### 3.3. O Benefício Prático da Preguiça

Vamos ilustrar o benefício de desempenho e eficiência com um cenário prático: **identificar o primeiro número primo** em uma sequência muito grande de inteiros.

```clojure
;; Função auxiliar para testar se um número é primo
(defn primo? [n]
  (and (> n 1)
       (not-any? #(zero? (mod n %))
                 (range 2 (inc (long (Math/sqrt n)))))))

;; Função que simula uma transformação de dados cara (por exemplo, uma consulta)
(defn detalhes-numero [n]
  (Thread/sleep 1)  ; simula operação lenta, como uma consulta de BD
  {:valor n :par? (even? n)})

;; Pipeline preguiçoso em Clojure
(first
  (filter (comp primo? :valor)
          (map detalhes-numero (range 1e6))))

```

### O que acontece, passo a passo:

1. **`first`** pede **um único item** do pipeline.
2. **`filter`** solicita um elemento de **`map`**.
3. **`map`** pega o primeiro número de `(range 1e6)` (ou seja, `0`), aplica `detalhes-numero` e entrega o resultado a `filter`.
4. **`filter`** testa o campo `:valor` com `primo?`.
    - Se **não** for primo, descarta e pede o próximo.
    - Se **for primo**, entrega o mapa para `first`.
5. **`first`**, tendo recebido o que precisava, **encerra o pipeline** imediatamente.

---

### Por que isso é eficiente?

Em Clojure, **tanto `map` quanto `filter` são *lazy sequences*** — ou seja, eles **só processam os dados quando necessário**.

O `first` pede elementos **um a um**, e a computação **para assim que encontra o primeiro primo**, sem precisar:

- Mapear toda a coleção de 1 milhão de itens;
- Nem filtrar a lista inteira.

Em uma linguagem ou abordagem **gulosa (eager)**, o programa:

- Executaria `map` para todos os elementos,
- Criaria uma coleção completa de `detalhes`,
- Depois aplicaria `filter` em tudo,
- E só então pegaria o primeiro resultado — desperdiçando memória e tempo.

---

### Resultado

Ao executar o código, o retorno será algo como:

```clojure
{:valor 2, :par? true}

```

E você notará que a execução é praticamente instantânea, porque o pipeline parou logo após encontrar o **primeiro número primo (2)** — os demais milhões de elementos **nem foram processados**.

!!! note "Preguiçoso, mas em blocos de 32"
    Na prática, `range` e `map` produzem *chunked sequences*: por eficiência, Clojure calcula os elementos em blocos de **32**. Se você contar as chamadas de `detalhes-numero` neste exemplo, verá que ela roda 32 vezes (para `0` a `31`), e não apenas 3. Ainda é muito menos que um milhão — mas é por isso que não se deve confiar na preguiça para controlar **efeitos colaterais** (como o `Thread/sleep`, uma consulta ou um `println`) dentro de `map`.

---

### Moral da história

Esse exemplo mostra de forma **testável e mensurável** o poder da **avaliação preguiçosa (lazy evaluation)** em Clojure:

- As funções `map`, `filter`, `take`, `drop`, `first` e outras **trabalham sob demanda**.
- O pipeline processa **somente o necessário**, item por item.
- Isso evita **uso excessivo de memória** e **cálculo desnecessário**, mesmo com coleções potencialmente infinitas.

### 3.4. Contraste: Funções Lazy vs. Eager

Embora a preguiça seja o padrão para a maioria das transformações de dados em Clojure, existem situações, especialmente aquelas que envolvem efeitos colaterais (como imprimir na tela ou escrever em um arquivo), onde a avaliação completa e imediata é necessária.

| Lazy (Preguiçoso) | Eager (Guloso) |
| --- | --- |
| **Funções:** `map`, `filter`, `range`, `take`, `repeat`, `cycle` | **Funções:** `doseq`, `doall`, `dorun`, `reduce` |
| **Comportamento:** Retornam imediatamente uma representação da sequência. O cálculo de cada elemento é adiado até que seja realmente solicitado. Ideal para pipelines de transformação de dados. | **Comportamento:** Forçam a avaliação completa da sequência que recebem. São usadas intencionalmente quando o objetivo principal são os efeitos colaterais de cada passo, e não o valor de retorno final. |

Em resumo, a preguiça é o comportamento padrão e desejável para transformações de dados, garantindo eficiência. A avidez (`eager`) é uma ferramenta usada intencionalmente para controlar a execução de efeitos colaterais.

## 4. Recursão e Otimização de Cauda em Clojure

Na programação funcional, a **recursão** é a abordagem natural para lidar com repetição. Essa preferência está ligada ao pilar da **imutabilidade**: como as estruturas de dados não podem ser alteradas, laços imperativos como `for` e `while`, que dependem da mutação de variáveis, tornam-se pouco adequados.

A recursão, por outro lado, é perfeitamente compatível com dados imutáveis, pois opera gerando *novos valores* a cada chamada, até atingir um **caso base** — a condição que interrompe a repetição.

### 4.1 O Problema Comum: O Custo da Pilha

Funções recursivas simples podem rapidamente consumir toda a **pilha de chamadas** (*call stack*). Cada nova chamada empilha um novo *frame* de execução. Quando o número de chamadas é grande, ocorre o erro clássico `StackOverflowError`.

Exemplo:

```clojure
;; ATENÇÃO: este código causará erro com números grandes.
(defn count-down [n]
  (if-not (zero? n)
    (do
      (if (= 0 (rem n 100))
        (println "count-down:" n))
      (count-down (dec n)))))

;; (count-down 100000) ;; -> java.lang.StackOverflowError

```

Cada chamada empilha uma nova execução de `count-down`. Como a pilha é limitada, a recursão profunda esgota a memória disponível.

---

### 4.2 A Solução de Clojure: `recur`

Em Haskell e em Elixir, uma chamada recursiva em **posição de cauda** (a última coisa que a função faz) é otimizada automaticamente: o compilador reaproveita o *frame* atual em vez de empilhar um novo — a chamada **otimização de chamada de cauda** (*Tail Call Optimization*, TCO). A JVM, onde Clojure roda, **não** faz essa otimização. Por isso, Clojure oferece uma forma especial explícita, **`recur`**, que produz o mesmo efeito.

Mas o `recur` não é uma chamada de função — ele é uma **instrução de salto controlado**: diz ao compilador para **reiniciar a execução** da função (ou de um `loop`) **com novos valores** para os parâmetros, **sem empilhar uma nova chamada**.

Para que o `recur` funcione:

- Ele deve estar em **posição de cauda**, ou seja, ser a **última expressão** da função;
- Deve receber exatamente o **mesmo número de argumentos** que a função (ou o `loop`) original.

---

### 4.3 Transformando uma Função Recursiva em `recur` (passo a passo)

Vamos entender **como transformar** uma função recursiva comum em uma versão com `recur`.

### Passo 1 — Função recursiva tradicional

```clojure
(defn soma [n]
  (if (zero? n)
    0
    (+ n (soma (dec n)))))

```

Essa versão funciona, mas cria uma nova chamada a cada passo. `(soma 100000)` vai causar `StackOverflowError`.

### Passo 2 — Adicionar um acumulador e colocar a recursão em cauda

No Passo 1, depois que `(soma (dec n))` retorna, ainda falta somar `n` — por isso cada chamada precisa ficar esperando na pilha. Introduzimos um argumento extra (`acc`) para guardar o resultado parcial; assim, a chamada recursiva passa a ser **a última operação** da função:

```clojure
(defn soma [n acc]
  (if (zero? n)
    acc
    (soma (dec n) (+ acc n))))  ;; agora está em posição de cauda

```

Em Haskell ou Elixir, isso já bastaria. Mas, na JVM, `(soma 100000 0)` **ainda** causa `StackOverflowError`: sem TCO, cada chamada continua empilhando um *frame*, mesmo estando em cauda.

### Passo 3 — Substituir a chamada por `recur`

Por fim, trocamos a chamada explícita pela forma especial:

```clojure
(defn soma [n acc]
  (if (zero? n)
    acc
    (recur (dec n) (+ acc n))))

```

Agora a função não empilha novas chamadas — o `recur` apenas **reinicia o corpo da função** com os novos valores.

Chamando `(soma 100000 0)` → executa perfeitamente, sem erro e sem consumir pilha adicional.

---

### 4.4 Como o `recur` “sabe” o que repetir

O `recur` **não chama a função pelo nome**.

Ele é uma **forma especial do compilador** que sabe para qual ponto voltar:

- Dentro de uma `defn`, ele volta para o **início da própria função**;
- Dentro de um `loop`, ele volta para o **início do loop mais próximo**.

Isso é resolvido **em tempo de compilação** — o compilador associa cada `recur` ao bloco que ele controla, garantindo segurança e eficiência.

---

### 4.5 Iterações Locais com `loop/recur`

Além de `recur` dentro de funções, Clojure oferece a combinação `loop/recur` para criar **laços locais e eficientes**, sem precisar definir uma função nomeada.

```clojure
(loop [result [] x 5]
  (if (zero? x)
    result
    (recur (conj result x) (dec x))))
;; => [5 4 3 2 1]

```

O `loop` define as variáveis locais (`result`, `x`), e o `recur` reinicia o laço com novos valores.

É uma forma funcional, segura e performática de representar iteração — sem mutabilidade e sem consumo de pilha.

---

### 4.6 Conclusão

O `recur` permite que Clojure use **recursão como forma natural de iteração**, conciliando elegância funcional e eficiência prática.

Transformar uma função recursiva comum em uma versão com `recur` segue um processo sistemático:

1. Introduzir um **acumulador** se necessário;
2. Garantir que a recursão esteja **em posição de cauda**;
3. Substituir a chamada recursiva por `recur`.

Assim, obtemos o mesmo comportamento de um loop imperativo, mas preservando os princípios da **imutabilidade** e da **programação funcional pura**.

---

## Conclusão

Neste capítulo, exploramos um conjunto de ferramentas que formam a espinha dorsal da manipulação de dados em Clojure. Vimos como a agregação com `group-by` nos permite resumir dados complexos. Aprendemos a usar o rico conjunto de funções de sequência, como `sort-by`, `take` e `reverse`, para organizar e focar em informações relevantes. E, crucialmente, entendemos como a avaliação preguiçosa torna esses pipelines de dados não apenas possíveis, mas extremamente eficientes, mesmo com coleções massivas ou infinitas. O domínio desses conceitos — agregação, manipulação de sequências e preguiça — é o que permite a construção de código idiomático, expressivo, legível e de alto desempenho em Clojure.