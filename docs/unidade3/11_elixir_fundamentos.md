# Fundamentos do Elixir

## 1. Introdução e Conexão de Paradigmas: Clojure vs Elixir

Seja bem-vindo(a) à introdução ao Elixir!

Após explorar o ecossistema Lisp com o **Clojure**, que roda na JVM e foca fortemente em processamento de dados imutáveis com S-expressions e metaprogramação, passamos para a segunda metade da Unidade 3, onde estudamos a linguagem **Elixir**.

Ambas as linguagens compartilham princípios funcionais cruciais, como imutabilidade e funções puras, mas diferem amplamente na sua plataforma e propósito de concorrência:

1. **VM do Erlang (BEAM) vs JVM**: A JVM do Clojure foi desenhada para alta performance de computação bruta em thread pools. A BEAM do Elixir foi desenhada para sistemas distribuídos altamente tolerantes a falhas e concorrentes com milhões de processos leves de baixo consumo de memória.
2. **Modelo de Atores vs Memória Compartilhada**: Clojure lida com concorrência usando referências de estado imutável com STM (Software Transactional Memory) ou Atoms sobre threads compartilhadas. Elixir utiliza o **Modelo de Atores**, onde processos são totalmente isolados e não compartilham nenhuma memória entre si, comunicando-se exclusivamente via troca de mensagens assíncronas.
3. **Sintaxe Amigável**: Elixir possui uma sintaxe influenciada por Ruby, muito mais familiar e acessível para desenvolvedores de linguagens imperativas do que as S-expressions do Clojure, mas mantendo a semântica funcional (dados imutáveis) sob o capô.
4. **O Operador Pipe (`|>`)**: Elixir introduz uma das sintaxes mais elegantes para pipeline de dados, o operador pipe, que permite encadear transformações de dados de forma extremamente legível (similar às threading macros do Clojure, mas integrado na raiz da sintaxe da linguagem).

Nesta seção, você aprenderá os fundamentos sintáticos e operacionais do Elixir, preparando-se para construir aplicativos web de tempo real com Phoenix LiveView.

---

## **Objetivos de Aprendizagem**

- Instalar e configurar o ambiente Elixir
- Dominar o uso do IEx (Interactive Elixir)
- Compreender tipos básicos e imutabilidade
- Aplicar o operador pipe (`|>`)
- Manipular coleções básicas com `Enum`

---

## **1.1 Instalação e Ambiente**

Existem diversas maneiras de instalar o Elixir; veja mais detalhes em <https://elixir-lang.org/install.html>.

---

## **1.2 Tipos Básicos**

### **Inteiros e Operações Aritméticas**

```elixir
42
0x1F  # hexadecimal -> 31
0b1010  # binário -> 10
1_000_000  # separador visual -> 1000000

1 + 2  # 3
10 / 2  # 5.0 (float)
div(10, 2)  # 5 (divisão inteira)
rem(10, 3)  # 1 (resto)

```

> Inteiros podem ser representados em diferentes bases e suportam operações básicas.
> 

---

### **Pontos Flutuantes**

```elixir
3.14
1.0e-10

```

> Usados para valores decimais ou muito pequenos/grandes, sempre em precisão dupla (64 bits).
> 

---

### **Booleanos e Valores de Verdade**

```elixir
true
false
nil  # também considerado "falsy"

!!1    # true
!!nil  # false

```

> Em Elixir, apenas false e nil são falsos. Todo o resto é verdadeiro.
> 

---

### **Átomos**

```elixir
:ok
:error
:admin
:"átomo com espaços"

```

> Átomos são constantes nomeadas, cujo nome é o próprio valor.
> 
> 
> Servem para:
> 

- Representar **estados** (`:ok`, `:error`)
- Rótulos simbólicos e flags
- Identificadores em estruturas de dados

```elixir
File.read("dados.txt")
# {:ok, "conteúdo"} ou {:error, :enoent}

```

---

### **Strings**

```elixir
"Hello World"
"interpolação #{1 + 1}"  # => "interpolação 2"
String.length("Olá")      # 3

```

> Strings são binários UTF-8, suportando acentos e emojis.
> 
> 
> Suportam **interpolação** e funções do módulo `String`.
> 

---

## **1.3 Imutabilidade**

```elixir
x = [1, 2, 3]
y = [0 | x]  # x não é modificado
x  # => [1, 2, 3]
y  # => [0, 1, 2, 3]

```

> Todos os valores em Elixir são imutáveis. Alterar dados cria novas estruturas, preservando as originais.
> 

---

## **1.4 Coleções Básicas**

### **Listas (`[ ]`)**

```elixir
list = [1, 2, 3, 4]
[head | tail] = list
# head: 1, tail: [2, 3, 4]
[0 | list]  # [0, 1, 2, 3, 4]

```

> Sequências ordenadas, boas para iterações e transformações.
> 
> 
> Acesso rápido no início, lento no final.
> 

---

### **Tuplas (`{ }`)**

```elixir
person = {"João", 25, :engineer}
elem(person, 0)  # "João"
put_elem(person, 1, 26)  # {"João", 26, :engineer}

```

> Estrutura fixa de tamanho conhecido.
> 
> 
> Ideal para **retornar múltiplos valores** ou **representar estados**.
> 

---

### **Mapas (`%{ }`)**

```elixir
user = %{name: "Maria", age: 30}
user.name       # "Maria"
user[:name]     # "Maria"
Map.get(user, :age)  # 30

```

> Coleção de pares chave-valor, ótima para representar entidades.
> 
> 
> Atualizações geram **novos mapas**.
> 

---

### **Keyword Lists (`[chave: valor]`)**

```elixir
config = [host: "localhost", port: 4000]
config[:host]  # "localhost"

```

> Lista de tuplas com chaves átomos, preservando ordem.
> 
> 
> Muito usada para **configurações e opções de funções**.
> 

!!! tip "Mapas vs Keyword Lists"
    Em Elixir, **mapas** e **keyword lists** são coleções de pares chave-valor, mas têm diferenças importantes:

    | Característica | Mapas (`%{}`) | Keyword Lists (`[key: value]`) |
    | --- | --- | --- |
    | **Sintaxe** | `%{chave => valor}` | `[chave: valor]` |
    | **Chaves** | Qualquer tipo | Sempre átomos (`:chave`) |
    | **Ordem** | Não garantida | Mantida (útil quando a ordem importa) |
    | **Chaves duplicadas** | Não permitidas | Permitidas |
    | **Acesso** | `map[:chave]` ou `map.chave` | `kw[:chave]` |
    | **Uso típico** | Dados estruturados complexos | Opções de funções, configurações simples |

    **Resumo:**

    - Use **mapas** para representar dados estruturados (como registros de pessoas, produtos, etc.).
    - Use **keyword lists** para passar **opções ou parâmetros de funções**, especialmente quando a ordem importa.

    Exemplo:

    ```elixir
    # Mapa
    user = %{name: "Ana", age: 30}
    user[:name]
    # "Ana"

    # Keyword List
    opts = [host: "localhost", port: 4000]
    opts[:host]
    # "localhost"
    ```

    Apesar de parecerem similares, a escolha depende do contexto e da necessidade de ordem ou chaves duplicadas.

---

## **1.5 Operador Pipe (`|>`)**

```elixir
"  hello world  "
|> String.trim()
|> String.upcase()
# "HELLO WORLD"

[1, 2, 3, 4, 5]
|> Enum.filter(&(&1 > 2))
|> Enum.map(&(&1 * 2))
|> Enum.sum()
# 24

```

> Encadeia funções de forma sequencial e legível.
> 

---

## **1.6 Introdução ao Enum**

```elixir
Enum.count([1, 2, 3])         # 3
Enum.empty?([])               # true
Enum.member?([1, 2, 3], 2)   # true

Enum.map([1, 2, 3], fn x -> x * 2 end)       # [2,4,6]
Enum.filter([1, 2, 3, 4], fn x -> rem(x,2)==0 end)  # [2,4]

```

> Enum oferece funções de ordem superior para manipular coleções de forma funcional.
> 

---

## **Exercícios em Sala**

1. Criar uma lista com números de 1 a 10 e, usando pipe, **filtrar apenas os pares** e depois **dobrar cada um** (resultado esperado: `[4, 8, 12, 16, 20]`). Por que não faria sentido dobrar primeiro e filtrar depois?
2. Criar um mapa representando uma pessoa, **adicionar nova chave** usando `Map.put`.

---

## **Tarefa para Casa**

1. Ler as lições “Basics” do Elixir School em português.
2. Fazer exercícios do Exercism: "Hello World", "Two Fer", "Leap".
3. Praticar no IEx: criar **10 pipelines diferentes** usando `Enum`.

---

## **Material de Apoio**

- [Elixir School - Básico](https://elixirschool.com/pt/lessons/basics/basics)
- [HexDocs - Getting Started](https://hexdocs.pm/elixir/introduction.html)
- [Exercism - Trilha Elixir](https://exercism.org/tracks/elixir)

---