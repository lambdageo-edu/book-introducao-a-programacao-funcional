# Coleções Avançadas e Pattern Matching

**Tópicos:** Enum avançado, comprehensions, pattern matching, controle de fluxo

## **Objetivos de Aprendizagem**

- Dominar operações avançadas com `Enum`
- Aplicar list comprehensions
- Implementar pattern matching em diferentes contextos
- Utilizar estruturas de controle de fluxo

---

## **2.0 Pattern Matching**

O **pattern matching** é um dos recursos centrais de Elixir.

Em vez de simplesmente atribuir valores, o operador `=` **verifica se um padrão se encaixa em uma estrutura de dados** — e, se possível, extrai valores automaticamente.

Essa técnica substitui boa parte do uso de condicionais e facilita o código declarativo e seguro.

### **Operador de Match (=)**

```elixir
# Match básico
x = 1
1 = x  # não é atribuição, é match!
2 = x
# ** (MatchError) no match of right hand side value: 1

```

## **2.1 Enum Avançado**

O módulo `Enum` é uma das ferramentas mais poderosas da linguagem Elixir. Ele fornece funções para **iterar, transformar, filtrar e reduzir coleções** de forma declarativa e funcional.

A maior parte das operações retorna uma nova lista (ou outro tipo de coleção) sem alterar a original — reforçando a **imutabilidade** característica da linguagem.

### **Operações de Transformação**

Essas funções permitem modificar ou combinar elementos de coleções.

```elixir
# Reduce (acumulador)
Enum.reduce([1, 2, 3, 4], 0, fn x, acc -> x + acc end)
# 10

Enum.reduce([1, 2, 3, 4], [], fn x, acc -> [x * 2 | acc] end)
# [8, 6, 4, 2]

# Flat map (achatar e mapear)
Enum.flat_map([[1, 2], [3, 4]], fn x -> x end)
# [1, 2, 3, 4]

Enum.flat_map([1, 2, 3], fn x -> [x, x * 2] end)
# [1, 2, 2, 4, 3, 6]

# Group by
Enum.group_by([1, 2, 3, 4, 5, 6], fn x -> rem(x, 2) end)
# %{0 => [2, 4, 6], 1 => [1, 3, 5]}

# Chunk
Enum.chunk_every([1, 2, 3, 4, 5, 6], 2)
# [[1, 2], [3, 4], [5, 6]]

```

---

### **Operações de Busca e Comparação**

Além das transformações, o módulo `Enum` também oferece funções para **buscar elementos, verificar condições e comparar valores** dentro de coleções.

```elixir
# Find
Enum.find([1, 2, 3, 4], fn x -> x > 2 end)
# 3

Enum.find([1, 2, 3, 4], fn x -> x > 10 end)
# nil

# All, Any
Enum.all?([1, 2, 3], fn x -> x > 0 end)
# true

Enum.any?([1, 2, 3], fn x -> x > 2 end)
# true

# Min, Max
Enum.min_max([3, 1, 4, 2])
# {1, 4}

```

---

## **2.2 List Comprehensions**

As **list comprehensions** são uma forma concisa e expressiva de gerar e transformar coleções em Elixir.

Inspiradas em linguagens funcionais como Haskell, elas permitem **iterar sobre listas, aplicar filtros e produzir novas listas** em uma única linha de código.

### **Sintaxe Básica**

A estrutura geral é:

```elixir
for <geradores>, <filtros>, do: <expressão>

```

```elixir
# For básico
for x <- [1, 2, 3, 4], do: x * x
# [1, 4, 9, 16]

# Com filtro
for x <- 1..10, rem(x, 2) == 0, do: x
# [2, 4, 6, 8, 10]

# Múltiplos geradores
for x <- [1, 2], y <- [3, 4], do: {x, y}
# [{1, 3}, {1, 4}, {2, 3}, {2, 4}]

```

---

### **Comprehensions Avançadas**

Comprehensions podem incluir **pattern matching**, **múltiplos geradores** e a opção `:into`, que permite coletar resultados em outras estruturas, como mapas ou conjuntos.

```elixir
# Com pattern matching
users = [%{name: "Ana", age: 25}, %{name: "João", age: 30}]
for %{name: name, age: age} <- users, age > 26, do: name
# ["João"]

# Into (coletar em estrutura diferente)
for {k, v} <- %{a: 1, b: 2}, into: %{}, do: {k, v * 2}
# %{a: 2, b: 4}

# Nested comprehensions
matrix = [[1, 2], [3, 4], [5, 6]]
for row <- matrix, x <- row, do: x * 2
# [2, 4, 6, 8, 10, 12]

```

---

## **2.3 Pattern Matching com Estruturas**

O pattern matching pode ser usado para **desestruturar listas, tuplas e mapas**, extraindo apenas as partes relevantes.

```elixir
# Listas
[head | tail] = [1, 2, 3, 4]
# head: 1
# tail: [2, 3, 4]

# Tuplas
{:ok, result} = {:ok, "success"}
# result: "success"

# Mapas
%{name: name} = %{name: "Carlos", age: 28}
# name: "Carlos"

```

---

## **Pin Operator (`^`)**

O **pin operator (`^`)** serve para **“fixar” o valor atual de uma variável durante um pattern match**, evitando que ela seja sobrescrita.

Em Elixir, normalmente quando você faz um match:

```elixir
x = 1
x = 2
# x agora vale 2

```

O valor antigo de `x` é simplesmente substituído. Mas às vezes queremos **usar o valor atual de `x` para comparação**, sem reatribuir. É aí que entra o `^`.

---

### **Exemplo Básico**

```elixir
x = 1
[^x, y] = [1, 2]
# O primeiro elemento do lado direito precisa ser igual a x
# y agora vale 2

```

Se o valor não corresponder:

```elixir
[^x, y] = [2, 3]
# MatchError, porque 2 ≠ x (que é 1)

```

> O ^ “pina” o valor de x, dizendo: “não quero reatribuir x, quero que o valor seja igual ao que x já tem”.
> 

---

### **Casos de uso**

**1. Comparação de padrões sem sobrescrever variáveis**

```elixir
status = :ok
case {:ok, "dados"} do
  {^status, result} -> "Sucesso: #{result}"
  {:error, reason} -> "Erro: #{reason}"
end
# "Sucesso: dados"

```

> Aqui usamos ^status para garantir que só vamos corresponder quando o status atual for :ok.
> 

**2. Filtros em listas com pattern matching**

```elixir
x = 10
list = [5, 10, 15]

for ^x <- list, do: IO.puts("Encontrei 10!")
# Apenas imprime se o elemento for igual a x

```

**3. Funções com múltiplas cláusulas**

```elixir
x = :admin
role_check = fn
  ^x -> "Administrador reconhecido"
  _ -> "Outro usuário"
end

role_check.(:admin)  # "Administrador reconhecido"
role_check.(:user)   # "Outro usuário"

```

> Sem o ^, o primeiro parâmetro seria simplesmente atribuído a x, o que não é o comportamento desejado.
> 

---

**Resumo:**

- Sem `^`, a variável **pode ser sobrescrita** durante um match.
- Com `^`, o match **verifica o valor existente** e falha se não corresponder.
- Útil em **cases, funções com múltiplas cláusulas, filtros e comparações de padrões complexos**.

---

## **2.4 Funções Anônimas e Pattern Matching**

Em Elixir, funções anônimas (`fn ... end`) podem ter **várias cláusulas**, cada uma com seu **pattern matching** e **guard**.

Isso permite criar funções **flexíveis e expressivas**, que se comportam de forma diferente dependendo do formato ou tipo do argumento recebido.

```elixir
# Definindo uma função anônima com múltiplas cláusulas
greet = fn
  %{name: name} ->
    # Se o argumento for um mapa contendo a chave :name
    "Olá, #{name}!"

  name when is_binary(name) ->
    # Se o argumento for uma string (binário UTF-8)
    "Olá, #{name}!"

  _ ->
    # Qualquer outro tipo de argumento
    "Olá, pessoa desconhecida!"
end

# Testando a função
greet.(%{name: "Ana"})
# "Olá, Ana!"       -> corresponde à primeira cláusula (mapa)

greet.("João")
# "Olá, João!"      -> corresponde à segunda cláusula (string)

greet.(123)
# "Olá, pessoa desconhecida!" -> corresponde à terceira cláusula (catch-all)

```

### **O que está acontecendo**

1. **Pattern Matching:**
    - Cada cláusula da função tenta “encaixar” o argumento recebido.
    - A primeira que corresponder será executada.
2. **Guards (`when is_binary(name)`):**
    - Permite adicionar **condições extras** além do pattern matching.
    - Aqui garantimos que a segunda cláusula só será usada se o argumento for uma **string**.
3. **Cláusula coringa (`_`):**
    - Captura qualquer valor que não tenha correspondido às cláusulas anteriores.

!!! tip "Chamando uma função anônima"
    Em Elixir, funções anônimas são armazenadas em variáveis. Para chamá-las, usamos o **ponto (`.`)**:

    ```elixir
    greet = fn name -> "Olá, #{name}!" end
    greet.("João")  # Chamada correta
    ```

    Sem o ponto, o Elixir procura por uma função nomeada no módulo, e não funcionará.

    **Resumo:**

    - **Função anônima em variável:** `variavel.(args)`
    - **Função nomeada:** `Modulo.func(args)` ou `func(args)` dentro do mesmo módulo.

---

## **2.5 Estruturas de Controle**

Embora Elixir seja uma linguagem funcional, ela oferece **estruturas de controle de fluxo** para lidar com múltiplos cenários de decisão.

As mais comuns são `case`, `cond` e `with`.

### **Case**

Permite combinar pattern matching com múltiplos ramos de execução.

```elixir
result = {:ok, "dados"}

case result do
  {:ok, data} -> "Sucesso: #{data}"
  {:error, reason} -> "Erro: #{reason}"
  _ -> "Resultado inesperado"
end
# "Sucesso: dados"

```

---

### **Cond**

Equivalente a uma sequência de “if... else if...”, ideal quando se quer **testar múltiplas condições** sem depender de pattern matching.

```elixir
age = 25

cond do
  age < 18 -> "menor de idade"
  age < 65 -> "adulto"
  true -> "idoso"
end
# "adulto"

```

---

### **With**

O `with` encadeia várias operações que podem **falhar ou retornar tuplas**, tornando o código mais legível do que múltiplos `case` aninhados.

```elixir
user_input = %{name: "Ana", age: "25"}

with {:ok, name} <- Map.fetch(user_input, :name),
     {age, ""} <- Integer.parse(user_input.age),
     true <- age >= 18 do
  %{name: name, age: age, adult: true}
else
  error -> {:error, error}
end

# %{name: "Ana", age: 25, adult: true}

```

---

## **Exercícios em Sala**

### 1. **Comprehension Complexa**

```elixir
# Dados: lista de vendas [%{produto: "A", valor: 100}, ...]
# Criar comprehension que filtre vendas > 50 e calcule imposto de 10%

```

---

### 2. **Pattern Matching em Função**

```elixir
# Criar função que processa diferentes tipos de resposta HTTP:
# {:ok, data}, {:error, code}, {:redirect, url}

```

---

## **Tarefa para Casa**

1. Fazer os exercícios do Exercism:
    - **"List Ops"**
    - **"Nucleotide Count"**
    - **"Word Count"**
2. Implementar uma função que use **pattern matching** para processar uma **árvore binária**
3. Criar **5 exemplos diferentes** de **list comprehensions**

---