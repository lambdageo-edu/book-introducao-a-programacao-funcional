# Funções de alta ordem

## 1. Introdução: Funções como Superpoderes

Uma das ideias mais poderosas da programação funcional em Clojure é o tratamento de **funções como dados**. Isso significa que uma função não é apenas um bloco de código que executa uma tarefa; ela pode ser tratada como qualquer outro valor, como um número ou uma string.

Pense nisso da seguinte forma: assim como você pode passar o número `100` para uma função que calcula um total, em Clojure, você pode passar a *própria função* `soma` como um argumento. Dizemos que as funções são **"cidadãos de primeira classe"**, pois elas têm os mesmos direitos e privilégios que outros tipos de dados na linguagem.

Este documento explora três conceitos centrais que desbloqueiam esse superpoder e permitem escrever código mais limpo, flexível e expressivo:

1. **Funções de Alta Ordem (HOFs):** Funções que operam sobre outras funções.
2. **Funções Anônimas (Lambdas):** Funções "descartáveis" criadas no momento do uso. Não precisam ser associadas a um nome.
3. **Composição de Funções:** A arte de construir novas funções combinando as existentes.

## 2. Funções como Cidadãos de Primeira Classe: As HOFs

### 2.1. O que significa "Cidadão de Primeira Classe"?

Em Clojure, quando dizemos que as funções são "cidadãos de primeira classe", queremos dizer que elas podem ser:

- **Passadas como argumentos** para outras funções.
- **Retornadas como resultado** de outras funções.
- **Armazenadas em estruturas de dados** ou **atribuídas a um símbolo (variável)** com `def`, assim como qualquer outro valor.

Uma **Função de Alta Ordem (HOF - Higher-Order Function)** é simplesmente uma função que aproveita essa característica, seja recebendo outra função como argumento, seja retornando uma nova função como resultado.

Veja um exemplo prático. A função `valor-descontado` abaixo foi projetada para receber não apenas o valor a ser descontado, mas também a *própria regra* que decide se o desconto deve ser aplicado.

```clojure
(defn mais-caro-que-100?
  [valor-bruto]
  (> valor-bruto 100))

(defn valor-descontado
  "Retorna o valor com desconto de 10% se deve aplicar desconto."
  [aplica? valor-bruto] ; 'aplica?' é um parâmetro que espera uma função
  (if (aplica? valor-bruto)
    (* valor-bruto 0.9)
    valor-bruto))

;; Passando a função `mais-caro-que-100?` como argumento
(println (valor-descontado mais-caro-que-100? 1000)) ; imprime 900.0
(println (valor-descontado mais-caro-que-100? 100))  ; imprime 100

```

Neste caso, `valor-descontado` é uma HOF porque aceita a função `mais-caro-que-100?` como seu primeiro argumento, `aplica?`.

### 2.2. HOFs em Ação: `map`, `filter` e `reduce`

`map`, `filter` e `reduce` são a tríade clássica de HOFs para processamento de coleções em Clojure. Todas recebem uma função como primeiro argumento para operar sobre os elementos de uma coleção.

Para os exemplos a seguir, considere a coleção `precos` e uma função `preco-final`, de um único argumento, que aplica o desconto com base em uma regra fixa (diferente de `valor-descontado`, ela não recebe a regra como parâmetro), além da função predicado `aplica-desconto?`:

```clojure
(def precos [30 700 1000])

(defn aplica-desconto?
  [valor-bruto]
  (> valor-bruto 100))

(defn preco-final
  "Retorna o valor com desconto de 10% se o valor bruto for estritamente maior que 100."
  [valor-bruto]
  (if (aplica-desconto? valor-bruto)
    (* valor-bruto 0.9)
    valor-bruto))

```

`map`: aplica uma função a cada item da coleção

A função `map` aplica `preco-final` a **cada elemento** da coleção `precos`.

```clojure
(map preco-final precos)
;=> (30 630.0 900.0)

```

O que aconteceu aqui:

- O preço `30` permanece igual (não recebe desconto).
- Os preços `700` e `1000` são descontados em 10%.

---

`filter`: seleciona apenas os elementos que satisfazem uma condição

A função `filter` usa o predicado `aplica-desconto?` para **filtrar** apenas os preços elegíveis para desconto.

```clojure
(filter aplica-desconto? precos)
;=> (700 1000)

```

Resultado: Apenas os valores **estritamente maiores que 100** são mantidos.

---

`reduce`: reduz a coleção a um único valor

A função `reduce` combina os valores da coleção em **um único resultado**. Exemplo: somar todos os preços.

```clojure
(reduce + precos)
;=> 1730

```

Como funciona internamente:

1. Soma os dois primeiros: `(+ 30 700)` → `730`
2. Soma o resultado com o próximo: `(+ 730 1000)` → `1730`

O uso de HOFs é incrivelmente poderoso, mas definir uma função nomeada com `defn` para cada pequena operação pode ser verboso. É aqui que as funções anônimas entram em cena.

## 3. Funções "na Hora": O Poder das Lambdas (Funções Anônimas)

### 3.1. A Necessidade de Funções Descartáveis

Muitas vezes, a lógica que queremos passar para uma HOF é tão simples e usada em um único lugar que não vale a pena dar-lhe um nome formal com `defn`. Para esses casos, Clojure oferece as **funções anônimas**, também conhecidas como **lambdas**.

A forma `fn` permite criar uma função "na hora", sem associá-la a um símbolo.

```clojure
;; Em vez de definir uma função `mais-caro-que-100?` separadamente,
;; criamos a mesma lógica diretamente na chamada com `fn`.

(println (valor-descontado (fn [valor-bruto] (> valor-bruto 100)) 1000))

```

Essa função anônima `(fn [valor-bruto] (> valor-bruto 100))` existe apenas como argumento para `valor-descontado` e é descartada em seguida.

### 3.2. A Sintaxe Abreviada: `#()`

Para tornar as lambdas ainda mais concisas, Clojure oferece uma sintaxe abreviada `#()`, ideal para funções de uma única expressão. Dentro dela, os parâmetros são referenciados implicitamente:

- `%` refere-se ao primeiro (e único) argumento.
- `%1`, `%2`, etc., referem-se ao primeiro, segundo e demais argumentos, caso haja mais de um.

A lógica `(fn [valor-bruto] (> valor-bruto 100))` pode ser reescrita de forma muito mais compacta:

```clojure
#(> % 100)

```

A tabela a seguir compara as três maneiras de definir a mesma lógica de verificação de preço:

| Forma | Exemplo de Código | Principal Vantagem |
| --- | --- | --- |
| `defn` | `(defn mais-caro-que-100? [valor-bruto] (> valor-bruto 100))` | Reutilizável, nome claro, fácil de testar. |
| `fn` | `(fn [valor-bruto] (> valor-bruto 100))` | Descartável, não polui o namespace com nomes. |
| `#()` | `#(> % 100)` | Extremamente concisa para lógicas simples. |

Versões com funções anônimas

Para operações simples, podemos usar **funções anônimas** com a sintaxe `#()`:

```clojure
(map #(if (> % 100) (* % 0.9) %) precos)
;=> (30 630.0 900.0)

(filter #(> % 100) precos)
;=> (700 1000)

(reduce + precos)
;=> 1730

```

### Compondo tudo com threading (`->>`)

Você pode encadear operações de forma elegante usando o operador `->>`:

```clojure
(->> precos
     (filter aplica-desconto?)
     (map preco-final)
     (reduce +))
;=> 1530.0

```

Neste exemplo:

- Filtramos os preços com desconto
- Aplicamos o desconto
- Somamos o total com desconto

### Resumo rápido

| Função | O que faz | Exemplo com `precos` |
| --- | --- | --- |
| `map` | Transforma cada item | `(map preco-final precos)` |
| `filter` | Filtra itens com base em condição | `(filter aplica-desconto? precos)` |
| `reduce` | Reduz coleção a um único valor | `(reduce + precos)` |

A flexibilidade de criar funções rapidamente com lambdas nos leva ao próximo passo: combiná-las para criar comportamentos mais complexos.

## 4. Construindo Lógica com Blocos: A Composição de Funções

**Composição de funções** é o ato de combinar múltiplas funções para criar uma nova, que representa a aplicação sequencial delas. Em vez de aninhar chamadas de função, como `(not (gratuito? item))`, podemos usar uma HOF para construir uma função que faça isso por nós.

A principal ferramenta para composição em Clojure é a HOF `comp`.

### 4.1. Exemplo Prático: Criando a Função Oposta com `comp`

Imagine que temos uma função chamada `gratuito?` que verifica se um item **não tem custo** — ou seja, retorna `true` se o item for gratuito, e `false` caso contrário.

Agora, queremos criar a função oposta, chamada `pago?`, que retorna `true` para itens pagos e `false` para itens gratuitos.

Em vez de escrever toda a lógica novamente, podemos usar a função de alta ordem `comp` para **compor** a função `gratuito?` com a função `not` (que inverte um valor booleano).

Como fazer isso?

```clojure
(def pago? (comp not gratuito?))
```

---

O que está acontecendo aqui?

Vamos entender passo a passo essa única linha:

1. **`comp` recebe duas funções**:
    - A função da esquerda: `not`
    - A função da direita: `gratuito?`
2. **`comp` cria uma nova função** — uma função anônima que, quando chamada,
    - Primeiro chama a função da direita (`gratuito?`) com o argumento recebido.
    - Depois passa o resultado dessa chamada para a função da esquerda (`not`).
3. **`def` associa essa nova função ao nome `pago?`**, para que possamos reutilizá-la no nosso código.

---

Visualizando o fluxo

Quando chamamos:

```clojure
(pago? item)
```

A execução acontece assim:

```clojure
(not (gratuito? item))
```

Ou seja, `gratuito?` verifica se o item é gratuito, e `not` inverte o resultado, dizendo se o item é pago.

---

Por que isso é legal?

- **`comp` é uma função de alta ordem:**
    
    Ela **recebe funções como entrada** e **retorna uma nova função** — exatamente a definição de HOF.
    
- **Reutilização e clareza:**
    
    Em vez de repetir lógica, você reutiliza funções existentes e as combina para criar comportamento novo.
    
- **Código mais limpo e declarativo:**
    
    Você descreve *o que* quer fazer (compor funções) em vez de *como* fazer passo a passo.
    

---

Exemplo completo para testar

```clojure
(defn gratuito? [item]
  (= (:preco item) 0))

(def pago? (comp not gratuito?))

(pago? {:nome "Caneta" :preco 0})
;=> false

(pago? {:nome "Livro" :preco 25})
;=> true

```

## 5. Conclusão: A Tríade da Programação Funcional Expressiva

As Funções de Alta Ordem, as Lambdas e a Composição de Funções formam uma tríade que define a expressividade do código funcional em Clojure. Elas trabalham em perfeita sinergia:

- **HOFs** (`map`, `filter`, `reduce`) fornecem os padrões de abstração para operar sobre dados.
- **Lambdas** (`fn`, `#()`) oferecem a conveniência para usar essas abstrações de forma concisa e localizada.
- **Composição** (`comp`) permite construir novas abstrações e lógicas a partir das existentes, como se fossem blocos de montar.

Dominar esses três conceitos é um passo fundamental para evoluir de apenas escrever código que funciona para escrever código Clojure que é verdadeiramente limpo, reutilizável e elegante.