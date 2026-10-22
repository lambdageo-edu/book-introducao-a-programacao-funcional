# Vetores e Mapas

## 1.0 Introdução: Os Pilares da Organização de Dados

Em Clojure, quase todos os dados com os quais você irá interagir estarão organizados em coleções. Dentre elas, duas estruturas se destacam como pilares fundamentais: os **vetores** e os **mapas**. Pense em vetores como uma estante de livros numerada e em mapas como uma agenda de contatos onde você busca por nome. Dominar a criação, o acesso e a manipulação dessas coleções é o primeiro grande passo para escrever código funcional, organizado e eficiente. Ao final deste guia, você não apenas saberá usar vetores e mapas—você terá o modelo mental para organizar dados de forma eficiente em Clojure, compreendendo o conceito central que torna a linguagem tão poderosa e previsível: a imutabilidade.

## 2.0 Vetores: Sua Lista de Dados Ordenada e Imutável

### 2.1 O que é um Vetor e Como Criá-lo?

Um vetor em Clojure é uma coleção ordenada de elementos, muito similar a um *array* ou lista em outras linguagens de programação. Sua principal característica é o acesso rápido a qualquer elemento através de um índice numérico (começando em zero).

Para criar um vetor, utilizamos colchetes `[]` e separamos os elementos por espaços. Podemos definir um símbolo para referenciar nosso vetor usando `def`.

```clojure
;; Define um vetor chamado 'estoque' com dois produtos.
(def estoque ["Mochila", "Camiseta"])

```

**Nota de Estilo:** A vírgula `,` em Clojure é tratada como um espaço em branco (`whitespace`). Embora opcional, usá-la entre os elementos de uma coleção na mesma linha é uma boa prática que melhora a legibilidade do código.

### 2.2 Acessando Elementos: Duas Abordagens Seguras

Em Clojure, vetores são coleções indexadas, e você pode acessar seus elementos de duas formas principais: **usando o vetor como função** ou usando a **função `get`**.

Antes de tudo, vamos definir um vetor com alguns itens de estoque:

```clojure
(def estoque ["Mochila" "Camiseta"])

```

### **1. Usando o vetor como função**

Essa é a forma **mais rápida e idiomática** de acessar elementos. Em Clojure, vetores se comportam como funções: você pode "chamá-los" com um índice.

Exemplo válido:

```clojure
(estoque 0)
;=> "Mochila"

```

Aqui estamos pedindo o item no índice 0. Como o índice existe, recebemos "Mochila".

Exemplo inválido:

```clojure
(estoque 2)
;=> IndexOutOfBoundsException
```

O índice 2 **não existe**, então isso **lança uma exceção** e pode interromper seu programa.

> Use esta abordagem quando você tem certeza de que o índice existe.
> 

---

### **2. Usando a função `get`**

A função `get` é uma forma **mais segura** de acessar elementos. Ela aceita:

- a coleção (no caso, o vetor),
- o índice desejado,
- e **opcionalmente**, um valor padrão.

Exemplo com índice válido:

```clojure
(get estoque 1)
;=> "Camiseta"

```

Exemplo com índice inválido:

```clojure
(get estoque 17)
;=> nil

```

Diferente da abordagem anterior, `get` não lança erro se o índice for inválido — apenas retorna `nil`.

Exemplo com valor padrão:

```clojure
(get estoque 17 "Produto esgotado")
;=> "Produto esgotado"

```

Se o índice não existir, o valor `"Produto esgotado"` será retornado.

> Use get quando quiser evitar exceções e/ou quando fizer sentido ter um valor padrão.
> 

---

### Conclusão

- Use **o vetor como função** para máxima performance — mas **cuidado com exceções**.
- Use **`get`** para acessos **seguros** e mais **flexíveis**, especialmente quando o índice pode não existir.

### 2.3 Lendo e Modificando Vetores

Para interagir com vetores, duas funções são absolutamente essenciais: `count` para saber seu tamanho e `conj` para adicionar elementos.

Contando Elementos com `count`

**O que faz?**

A função `count` retorna o número de elementos em um vetor (ou em qualquer coleção, como listas, mapas e sets).

**Por que usar?**

É a maneira **mais direta e segura** de verificar o tamanho de uma coleção.

Exemplos:

```clojure
(def estoque ["Mochila" "Camiseta" "Tênis"])

(count estoque)
;=> 3

```

O vetor `estoque` tem 3 elementos, então `count` retorna `3`.

Você também pode usar `count` em coleções vazias:

```clojure
(count [])
;=> 0

```

---

**Adicionando Elementos com `conj`**

**O que faz?**

A função `conj` (*conjoin*) adiciona um elemento a uma coleção. Com vetores, o elemento é **adicionado ao final**.

**Por que usar?**

Clojure é uma linguagem funcional e imutável — isso significa que `conj` **não altera o vetor original**. Em vez disso, retorna **uma nova coleção com o elemento adicionado**.

**Exemplos:**

```clojure
(def estoque ["Mochila" "Camiseta"])

(conj estoque "Tênis")
;=> ["Mochila" "Camiseta" "Tênis"]

```

O vetor original continua o mesmo:

```clojure
estoque
;=> ["Mochila" "Camiseta"]

```

Se quiser atualizar a variável com o novo valor, você pode fazer isso com `def` novamente (ou `defonce`, `let`, etc., dependendo do contexto):

```clojure
(def estoque (conj estoque "Tênis"))

```

Agora sim, `estoque` contém o novo item:

```clojure
estoque
;=> ["Mochila" "Camiseta" "Tênis"]
```

!!! tip "Rebinding, não mutação"
    Embora pareça que a "variável" `estoque` tenha mudado de valor, o que realmente aconteceu aqui foi um **rebinding** — ou seja, o símbolo `estoque` foi **reassociado** a um novo valor.

    Em Clojure, os valores são imutáveis: o vetor original (`["Mochila" "Camiseta"]`) continua existindo, intacto. O que mudou foi apenas a **referência**: agora o nome `estoque` aponta para um novo vetor (`["Mochila" "Camiseta" "Tênis"]`).

    Isso é diferente de linguagens imperativas onde variáveis são caixas mutáveis. Em Clojure, estamos apenas **atualizando o mapeamento entre um nome e um valor**, sem modificar os dados em si.

---

Recapitulando

- `count` → conta os elementos de um vetor.
- `conj` → adiciona um elemento **ao final do vetor**, retornando uma nova coleção.

Essas duas funções são **fundamentais** para trabalhar com vetores de forma idiomática, segura e funcional em Clojure.

## 3.0 Mapas (Hashmaps): Organizando Dados com Chaves e Valores

### 3.1 O que é um Mapa e Por Que Usar Keywords?

Um mapa (ou *hashmap*) é uma coleção que associa **chaves** únicas a **valores**. É a estrutura de dados ideal para representar entidades e suas propriedades, como um produto e seu estoque, ou um usuário e seus detalhes. Mapas são criados com chaves `{}`.

Embora seja possível usar strings como chaves, a boa prática fundamental em Clojure é usar **keywords** (palavras-chave). Uma keyword é precedida por dois-pontos, como `:mochila`.

**A Grande Ideia:** O principal benefício é que keywords, assim como os vetores, implementam a interface de função do Clojure (`IFn`). Isso significa que uma keyword pode ser usada como uma função para buscar seu próprio valor em um mapa. Essa não é apenas uma conveniência sintática; é um princípio de design fundamental onde estruturas de dados são também executáveis, tornando o código de acesso incrivelmente conciso, legível e consistente.

```clojure
;; Define um mapa de estoque usando keywords como chaves
(def estoque {:mochila 10 :camiseta 5})

```

**Exemplo 1 – Acessando dados com `get` e com keywords**

```clojure
(def estoque {:mochila 10 :camiseta 5})

;; Acessando valores com `get`
(get estoque :mochila)
;; => 10

;; Acessando diretamente com keyword como função
(:mochila estoque)
;; => 10

;; Acessando uma chave inexistente
(get estoque :sapato)
;; => nil

;; Com valor padrão, caso a chave não exista
(get estoque :sapato 0)
;; => 0

```

Acessando dados em mapas aninhados com get-in

```clojure
(def usuario {:nome "João"
              :idade 30
              :endereco {:cidade "São Paulo"
                         :cep "01000-000"}})

;; Acessando o CEP
(get-in usuario [:endereco :cep])
;; => "01000-000"
```

!!! tip
    veremos esses mapas com mais detalhes em outra aula

verificando se uma chave existe

```clojure
(contains? estoque :mochila)
;; => true

(contains? estoque :sapato)
;; => false
```

iterando sobre mapas

```clojure
(doseq [[item quantidade] estoque]
  (println "Item:" item "-> Quantidade:" quantidade))

;; Saída:
;; Item: :mochila -> Quantidade: 10
;; Item: :camiseta -> Quantidade: 5
```

### 3.2 Explorando o Conteúdo de um Mapa

Para inspecionar o conteúdo de um mapa, três funções são essenciais:

- `count`: Assim como nos vetores, retorna o número de pares chave-valor no mapa.
- `keys`: Retorna uma sequência com todas as chaves do mapa. É útil para quando você precisa saber quais "rótulos" de dados estão disponíveis.
- `vals`: Retorna uma sequência com todos os valores do mapa. É útil para quando você só se interessa pelos dados em si, e não por suas chaves de associação.

Exemplos

```clojure
(def produto {:nome "Camiseta"
              :preco 49.90
              :estoque 20})
```

`count` – Quantidade de pares chave-valor

```clojure
(count produto)
;; => 3
```

**Explicação:**

O mapa `produto` tem 3 entradas: `:nome`, `:preco` e `:estoque`.

---

`keys` – Todas as chaves do mapa

```clojure
(keys produto)
;; => (:nome :preco :estoque)

```

---

`vals` – Todos os valores do mapa

```clojure
(vals produto)
;; => ("Camiseta" 49.90 20)
```

**Explicação:**

Essa é a sequência dos valores armazenados nas chaves, na mesma ordem que as chaves retornadas por `keys`.

Exercício

```clojure
(def aluno {:nome "Joana" :idade 22 :curso "Engenharia"})

;; Perguntas:
;; 1. Quantos dados temos sobre a aluna?
;; 2. Quais são os "rótulos" dessas informações?
;; 3. Quais são os valores?
;; 4. Imprima cada chave com seu valor correspondente.
```

### 3.3 Recapitulando

Assim como em vetores, há várias formas de acessar valores em um mapa em Clojure. Cada abordagem tem seu uso ideal. A seguir, exploramos as três formas principais.

---

**1. Usando Keywords como Funções (Forma Idiomática)**

Essa é a maneira mais comum, legível e preferida em Clojure. Você usa a própria **keyword** como se fosse uma função, passando o mapa como argumento.

```clojure
(def estoque {:mochila 10 :camiseta 5})

(:mochila estoque)
;; => 10

(:cadeira estoque)
;; => nil

```

Essa forma é **segura** e **nunca lança erro** caso a chave não exista — apenas retorna `nil`.

---

**2. Usando a Função `get` (Forma Segura com Valor Padrão)**

A função `get` também retorna o valor associado a uma chave, com a vantagem de permitir fornecer um valor **padrão** caso a chave não exista.

```clojure
(get estoque :camiseta)
;; => 5

(get estoque :cadeira)
;; => nil

(get estoque :cadeira 0)
;; => 0

```

Essa é a melhor opção quando você precisa de um valor padrão explícito ao acessar mapas.

---

**3. Usando o Mapa como Função (Forma Menos Comum)**

Em Clojure, **um mapa pode agir como uma função**, recebendo uma chave e retornando seu valor associado.

```clojure
(estoque :mochila)
;; => 10

```

Embora seja válida, essa abordagem é **menos usada** por um motivo importante: se o mapa for `nil`, você terá uma **NullPointerException**. Já `get` e `:keyword` são mais seguros nesse sentido.

---

**Resumindo ..**

- **Use `:keyword`** como função para acesso simples e idiomático.
- **Use `get`** quando quiser segurança e valor padrão.
- **Evite usar o mapa como função** diretamente, a menos que tenha certeza de que ele não é `nil`.

## 4.0 Manipulação Imutável: A Arte de Transformar Coleções

### 4.1 Modificando Mapas com `assoc` e `dissoc`

Em Clojure, os mapas são **imutáveis**. Isso significa que não alteramos um mapa original, mas sim criamos **uma nova versão modificada** usando funções como `assoc` e `dissoc`.

`assoc` – Adiciona ou atualiza uma chave

A função `assoc` é usada para **inserir** um novo par chave-valor ou **atualizar** uma chave existente.

Adicionando uma nova chave

```clojure
(def estoque {:mochila 10 :camiseta 5})

(def novo-estoque (assoc estoque :cadeira 3))

novo-estoque
;; => {:mochila 10, :camiseta 5, :cadeira 3}

```

O mapa original `estoque` **não foi alterado**. `novo-estoque` é uma nova versão com a chave `:cadeira` adicionada.

Atualizando uma chave existente

```clojure
(def estoque-atualizado (assoc estoque :mochila 1))

estoque-atualizado
;; => {:mochila 1, :camiseta 5}

```

Aqui, `:mochila` foi atualizada de `10` para `1`.

---

`dissoc` – Remove uma chave

A função `dissoc` é usada para **remover** uma chave (e seu valor) de um mapa.

```clojure
(def estoque-sem-mochila (dissoc estoque :mochila))

estoque-sem-mochila
;; => {:camiseta 5}

```

Assim como `assoc`, `dissoc` não altera o mapa original, mas retorna um **novo mapa** sem a chave especificada.

---

Observações Didáticas

- Essas funções são fundamentais para **trabalhar com dados imutáveis**, que é a base da programação funcional.
- Você pode usar `assoc` e `dissoc` em sequência para construir lógicas de atualização mais complexas.
- Ambas funcionam com mapas simples ou aninhados (em combinação com `assoc-in`, `update-in`, etc.).
    
### 4.2 A Função Universal: Atualizando Valores com `update`

A função `update` permite modificar um valor **existente** aplicando-lhe uma função, seja num **vetor** (por índice) ou num **mapa** (por chave).

Essa abordagem evita código imperativo como:

```clojure
;; Imperativo (errado em Clojure)
(let [atual (get mapa :x)
      novo (inc atual)]
  (assoc mapa :x novo))

```

Com `update`, tudo isso vira uma linha só.

---

1. Usando `update` com Mapas

A função recebe:

```clojure
(update mapa :chave funcao)

```

Exemplo – Atualizar estoque de produtos

```clojure
(def estoque {:mochila 10 :camiseta 5})

;; Aumentando em 1 a quantidade de mochilas
(update estoque :mochila inc)
;; => {:mochila 11, :camiseta 5}

;; Dobrando a quantidade de camisetas
(update estoque :camiseta #(* 2 %))
;; => {:mochila 10, :camiseta 10}

```

!!! note "O que é `#(* 2 %)`?"
    É uma **função anônima** escrita de forma abreviada: `#(* 2 %)` equivale a `(fn [x] (* 2 x))`, e o `%` representa o argumento recebido. Veremos funções anônimas em detalhe no próximo capítulo, sobre funções de alta ordem.

> Dica: O % representa o valor atual da chave. É como dizer: "pegue esse valor, faça isso com ele e retorne".
> 

---

2. Usando `update` com Vetores

A lógica é a mesma, mas o primeiro argumento agora é um **índice numérico**.

Exemplo – Aplicar desconto em um produto

```clojure
(def precos [100.0 200.0 150.0])

;; Aplicar 10% de desconto no segundo item (índice 1)
(update precos 1 #(Math/round (* 0.9 %)))
;; => [100.0 180 150.0]
;; Atenção: Math/round converte apenas o elemento atualizado para long;
;; os demais elementos permanecem Double, pois update só toca no índice indicado.

```

---

3. `update` com argumentos adicionais

Você pode passar **argumentos extras** para a função que está sendo aplicada:

```clojure
(update estoque :mochila + 5)
;; => {:mochila 15, :camiseta 5}

```

> Aqui, estamos somando 5 ao valor atual da chave :mochila.
> 

---

Recapitulando

- `update` é a forma idiomática de **ler → transformar → reescrever**, tudo em uma única operação.
- Funciona tanto com **mapas** (por chave) quanto com **vetores** (por índice).
- É **imutável**: o valor original não é modificado.

## 5.0 Quando o Estado Precisa Mudar: `atom`

Tudo o que vimos até aqui é imutável — e é assim que a maior parte de um programa Clojure deve ser. Mas um programa real às vezes precisa de **uma** coisa que muda ao longo do tempo: o estoque atual de uma loja, a lista de tarefas de um aplicativo. Para isso, Clojure oferece **referências**, e a mais simples delas é o `atom`: uma caixa que guarda um valor imutável e pode ter seu conteúdo **trocado** de forma segura, mesmo com várias threads.

```clojure
(def contador (atom 0))       ; cria a caixa com o valor inicial 0

@contador                     ; lê o valor atual (o mesmo que (deref contador))
;=> 0

(swap! contador inc)          ; troca o valor aplicando uma função: (inc 0)
;=> 1

(swap! contador + 10)         ; argumentos extras, como no update: (+ 1 10)
;=> 11

(reset! contador 0)           ; troca por um valor fixo
;=> 0
```

Repare que nada aqui é mutação de dados: o número `0` não "virou" `1`. O que muda é **qual valor imutável** está dentro da caixa. `swap!` recebe uma função pura (`inc`, `+`) e garante que a troca acontece de forma atômica — se duas threads chamarem `swap!` ao mesmo tempo, nenhuma atualização se perde. O ponto de exclamação no nome é a convenção de Clojure para "esta função tem efeito colateral".

Clojure também tem referências coordenadas (`ref`, alteradas dentro de transações com `dosync` — a STM, *Software Transactional Memory*), usadas quando várias referências precisam mudar juntas. Neste curso, o `atom` basta: é ele que guarda as tarefas em memória no [trabalho de Clojure](10_clojure_tutorial_todo.md).

## 6.0 Hora da Prática!

Siga os passos abaixo em seu ambiente Clojure para aplicar o que aprendeu.

1. **Crie um Vetor:** Crie um vetor chamado `carrinho-de-compras` que represente itens em um carrinho, usando keywords.
2. **Consulte o Vetor:** Verifique quantos itens há no carrinho.
3. **Adicione ao Vetor:** Use `conj` para adicionar o item `:mochila` ao carrinho e armazene o resultado em um novo símbolo `carrinho-atualizado`. Lembre-se que o vetor original não será alterado!
4. **Crie um Mapa:** Crie um mapa chamado `precos-produtos` que associe cada um dos três itens a um preço.
5. **Adicione/Atualize o Mapa:** Use `assoc` para adicionar um novo item, `:chaveiro`, com o preço `10`.
6. **Transforme um Valor:** Use `update` para aplicar um aumento de 10% no preço da `:camiseta`. A expressão `#(* % 1.1)` é uma função anônima que multiplica seu argumento (`%`) por 1.1.
7. **Desafio:** Crie um novo mapa `carrinho-final` a partir de `precos-produtos` que contenha *apenas* os itens do seu `carrinho-atualizado` (do passo 3). Pesquise a função `select-keys` para uma solução elegante.

## 7.0 Conclusão: Seus Novos Superpoderes em Clojure

Parabéns! Você acaba de dominar as estruturas de dados mais importantes de Clojure. Ao final deste guia, você solidificou três conceitos essenciais:

1. A diferença fundamental entre **vetores** (listas ordenadas por índice) e **mapas** (associações de chave-valor).
2. O princípio da **imutabilidade**, onde transformações como `conj` e `assoc` sempre criam novas coleções, deixando as originais intactas e seu código mais seguro.
3. O uso das funções essenciais (`get`, `count`, `conj`, `keys`, `vals`, `assoc`, `dissoc`, `update`) como seu kit de ferramentas básico para manipulação de dados.

Com esta base sólida, você está perfeitamente preparado para explorar o próximo nível. As ferramentas que você aprendeu aqui são os blocos de construção para usar funções de processamento de coleções ainda mais poderosas, como `map`, `filter` e `reduce`, que levarão sua habilidade de programar em Clojure a novas alturas.