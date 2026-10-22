# Metaprogramação

**Módulo 1: O "Código que escreve Código"**

Caro aluno, bem-vindo ao coração do Elixir.
Muitas linguagens tratam Metaprogramação como uma "magia negra" perigosa que deve ser evitada. No Elixir, ela é a base da própria linguagem.

## 1. Contextualização: Onde vivem as Macros?

Antes de escrevermos nossa primeira macro, você precisa entender um segredo: **Quase tudo o que você usa em Elixir é uma Macro.**

O núcleo da linguagem (o compilador) é extremamente pequeno. A maior parte das funcionalidades que você usa no dia a dia foi construída **sobre** esse núcleo, usando Metaprogramação.

Aqui estão os 4 exemplos clássicos de uso na indústria:

### 1.1. A Própria Linguagem (Módulo `Kernel`)

Você sabia que `if`, `unless`, `def`, `defmodule` e `alias` **não são** palavras reservadas do compilador?
Elas são macros escritas em Elixir!

- **Como funciona:** Quando você escreve `if`, o compilador troca isso por uma estrutura `case` (que é a primitiva real da linguagem).
- **Vantagem:** Isso torna o Elixir infinitamente extensível. Se você não gosta do `if` do Elixir, você pode criar o seu próprio `meu_if` e ele se comportará como um cidadão de primeira classe.

### 1.2. Ecto (Queries de Banco de Dados)

O Ecto permite escrever consultas SQL usando sintaxe Elixir.

```elixir
# Isso parece Elixir, mas vira SQL seguro
from u in User, where: u.age > 18
```

- **O Poder da Macro:** O Ecto usa macros para analisar seu código em **Tempo de Compilação**. Se você errar o nome de um campo ou tentar uma injeção de SQL, o Ecto detecta isso *antes* do seu código rodar, prevenindo bugs críticos em produção.

### 1.3. Phoenix (Roteamento Web)

O framework Phoenix usa macros para definir rotas de forma limpa.

```elixir
get "/users/:id", UserController, :show
```

- **O Poder da Macro:** Em tempo de compilação, o Phoenix transforma essa linha bonita em uma função gigante de Pattern Matching ultra-otimizada. Você escreve código legível; a macro gera código performático.

### 1.4. ExUnit (Testes Inteligentes)

Em outras linguagens, você tem `assertEquals(a, b)`. Em Elixir, temos apenas `assert a == b`.

- **O Poder da Macro:** Como `assert` é uma macro, ela recebe o código `a == b` (e não apenas o resultado `false`). Quando o teste falha, ela consegue dissecar o código e te dizer: *"Falhou porque o lado esquerdo era 5 e o direito era 10"*. Uma função normal não conseguiria fazer isso.

## 2. Teoria Fundamental: Função vs. Macro (A Linha do Tempo)

Para dominar a metaprogramação, você precisa ajustar sua visão sobre **quando** as coisas acontecem no seu software.

Na Engenharia de Software com Elixir, existem dois momentos distintos:

1. **Compile Time (Tempo de Compilação):** O momento em que o código fonte (`.ex`) é lido e transformado em binário (`.beam`).
2. **Runtime (Tempo de Execução):** O momento em que o usuário final está usando o sistema e o binário está rodando na CPU.

### 2.1. A Analogia do Cinema

Para visualizar a diferença, pense na produção de um filme:

- **A Macro é o Roteirista (Compile Time):**
O roteirista trabalha **antes** das câmeras ligarem. Ele recebe um esboço e pode reescrever a cena inteira, apagar personagens ou mudar o final.
    - *Entrada:* Texto (Ideias).
    - *Saída:* Roteiro Final.
    - *Poder:* Total. Pode impedir que uma cena sequer exista.
- **A Função é o Ator (Runtime):**
O ator entra em cena quando a câmera liga. Ele recebe o roteiro pronto e **tem que executá-lo**. Ele não pode decidir "não falar essa frase" se ela estiver no roteiro.
    - *Entrada:* Ação/Dados.
    - *Saída:* Performance.
    - *Limitação:* Segue estritamente o que foi definido antes.

---

### 2.2. A Prova Real: Por que Funções falham em Controle de Fluxo?

Vamos provar tecnicamente por que você **não** consegue criar estruturas como `if` ou `unless` usando funções normais.

O Elixir (assim como Java, C# e Python) usa **Avaliação Estrita (Eager Evaluation)** para funções. Isso significa que **todos os argumentos são calculados e executados ANTES de entrar na função.**

### O Experimento do "Apagar Banco"

Imagine que queremos criar nosso próprio `meu_if`. Vamos tentar fazer isso com uma função (`def`).

```elixir
defmodule TentativaRuim do
  # Uma função normal que recebe uma condição e dois blocos
  def meu_if(condicao, bloco_verdadeiro, bloco_falso) do
    if condicao do
      bloco_verdadeiro
    else
      bloco_falso
    end
  end
end
```

Agora, vamos simular um cenário perigoso no terminal (`iex`):
Queremos executar a ação de apagar o banco **apenas se** a condição for `true`.

```elixir
condicao = false

# Chamando a função:
TentativaRuim.meu_if(condicao, IO.puts("🔥 APAGANDO O BANCO..."), IO.puts("Ufa, salvo."))
```

**Resultado no Terminal:**

```text
🔥 APAGANDO O BANCO...
Ufa, salvo.
```

**O Desastre:**
A mensagem "Apagando o banco" apareceu, mesmo a condição sendo `false`!

**A Explicação de Engenharia:**

1. Antes de chamar `meu_if`, o Elixir precisa resolver os argumentos.
2. Ele executa `IO.puts("🔥...")` imediatamente.
3. Ele executa `IO.puts("Ufa...")` imediatamente.
4. Só depois ele passa os *resultados* (que são `ok`) para dentro da função.
5. O estrago já foi feito.

---

### 2.3. A Solução com Macros (Lazy Evaluation)

A Macro resolve isso porque ela roda no **Tempo de Compilação**.
Ela não recebe o *resultado* da execução ("Apagando..."). Ela recebe o **código em si** (a representação textual).

A Macro olha para o código e diz: *"Eu vejo que você escreveu um comando para apagar o banco. Não vou executá-lo agora: vou **reescrevê-lo** dentro de um `if` (ou `case`), de modo que ele só rode se a condição for verdadeira."*

Repare que a macro, em geral, **não sabe** o valor da condição — `condicao` só existe em tempo de execução. O que ela faz é gerar um código em que os ramos ficam **protegidos** por uma decisão tomada em Runtime: quando o programa roda, só o ramo escolhido é avaliado, e o comando perigoso nunca é executado se a condição for falsa.

| **Característica** | **Função (def)** | **Macro (defmacro)** |
| --- | --- | --- |
| **Quando roda?** | **Runtime** (Toda vez que é chamada). | **Compile Time** (Uma vez só, ao gerar o binário). |
| **O que recebe?** | **Dados** (O resultado de `1+1` é `2`). | **Código** (A estrutura `1+1`). |
| **Avaliação** | **Estrita** (Argumentos rodam antes). | **Preguiçosa** (Argumentos não rodam, são analisados). |
| **Poder** | Transformar dados. | Transformar a realidade da linguagem. |

---

## 3. A Árvore de Sintaxe (AST)

Para entender como criar macros, precisamos abrir a "Caixa Preta" do compilador.

Você, como engenheiro, está acostumado a escrever código como **Texto** (arquivos `.c`, `.java`, `.ex`). Mas o computador não entende texto; ele entende instruções de máquina. Como chegamos de um ponto ao outro?

### 3.1. A Linha de Montagem de um Compilador

Em qualquer linguagem (C, Java, Python), o compilador segue um pipeline padrão para transformar seu texto em binário. A AST nasce exatamente no meio desse processo.

1. **Análise Léxica (Tokenização):**
O compilador lê o arquivo de texto e agrupa caracteres em palavras conhecidas (tokens).
    - *Entrada:* `if a + b`
    - *Saída:* `[KEYWORD_IF, VAR_A, OP_PLUS, VAR_B]`
2. **Análise Sintática (Parsing) → Onde nasce a AST:**
Aqui acontece a mágica. O compilador organiza esses tokens soltos em uma estrutura hierárquica lógica, uma árvore que representa a **intenção** do código.
    - Chamamos de **"Abstrata"** (Abstract Syntax Tree) porque ela descarta detalhes visuais irrelevantes, como espaços em branco, comentários ou parênteses excessivos. Ela guarda apenas a pura estrutura lógica.
3. **Geração de Código:**
O compilador percorre essa árvore e cospe o código final (Bytecode ou Assembly).

### 3.2. O Diferencial do Elixir (O "Hack" do Sistema)

Na maioria das linguagens (como Java ou C#), essa **AST fica trancada dentro do compilador**. Você escreve o texto (passo 1) e recebe o binário (passo 3). Você nunca vê o passo 2.

**O segredo das Macros em Elixir:**
O Elixir pausa o processo exatamente entre o passo 2 e o 3. Ele te entrega a AST na mão e diz:

> "Aqui está a árvore que representa o seu código. Se você quiser mudar os galhos de lugar, adicionar folhas ou podar a árvore antes de eu gerar o binário final, fique à vontade."
> 

É por isso que precisamos deixar de ver o código como texto e passar a vê-lo como **Estrutura de Dados**.

---

### 3.3. A Homoiconicidade: Elixir vs. Clojure

Linguagens que permitem acessar e modificar sua própria AST facilmente são chamadas de **Homoicônicas**.

Se você já estudou **Lisp** ou **Clojure**, deve lembrar que lá o código é cheio de parênteses. Isso acontece porque, nessas linguagens, você escreve a AST manualmente usando Listas.

- **Em Clojure:** A AST é uma **Lista**.
`(+ 1 2)` → O primeiro item é a função, o resto são argumentos.

- **Em Elixir:** A AST é uma **Tupla**.
O Elixir oferece uma sintaxe mais limpa (sem tantos parênteses), mas por baixo dos panos, ele converte tudo para tuplas de 3 elementos.

**Comparativo Visual:**

| **Linguagem** | **Código Fonte (Texto)** | **Estrutura Interna (AST)** | **Tipo de Dado** |
| --- | --- | --- | --- |
| **Clojure** | `(+ 1 2)` | `(list + 1 2)` | Lista Encadeada |
| **Elixir** | `1 + 2` | `{:+, [...], [1, 2]}` | Tupla |

Em ambas, **Código é Dado**. Saber escrever macros é apenas saber manipular Listas (no Clojure) ou Tuplas (no Elixir).

### 3.4. Anatomia da AST em Elixir

Em Elixir, quase todo código vira uma tupla com este formato exato:

```elixir
{nome_da_operacao, metadados, argumentos}
```

1. **Nome (Atom):** Qual função ou macro está sendo chamada (ex: `:if`, `:sum`, `:def`).
2. **Metadados (List):** Informações para o compilador (número da linha, contexto, imports). Geralmente ignoramos isso em macros simples.
3. **Argumentos (List):** Os dados ou sub-árvores que a função recebeu.

---

### 3.5. Laboratório: Raio-X com `quote`

Vamos usar a macro `quote` para atuar como o Parser do compilador e ver essa estrutura.

Abra o `iex`:

**Experimento A: Matemática Simples**

```elixir
# O Elixir lê "1 + 2" e transforma nisto:
quote do: 1 + 2

# Saída: {:+, [context: Elixir, import: Kernel], [1, 2]}
```

- Operação: `:+`
- Argumentos: `[1, 2]`

Experimento B: Funções aninhadas

Veja como a árvore cresce para baixo.

```elixir
quote do: soma(1, div(10, 2))

# Saída:
# {
#   :soma, 
#   [...], 
#   [1, {:div, [...], [10, 2]}]  <-- Olha a sub-árvore aqui!
# }
```

Perceba que o segundo argumento não é o número `5`. É outra tupla (nó da árvore) representando a divisão.

Experimento C: Variáveis

Até variáveis são representadas por tuplas na AST.

```elixir
quote do: idade
# Saída: {:idade, [], Elixir}
```

---

### 3.6. A Dinâmica: Injetando Valores com `unquote`

Agora que você viu que `quote` transforma **tudo** em uma estrutura de dados estática, surge um problema de engenharia.

Imagine que você quer gerar um código de soma, mas um dos números vem de uma variável externa.

### O Problema: O Isolamento do `quote`

Vamos ao laboratório (IEx):

```elixir
numero_externo = 50

# Tentativa ingênua:
ast = quote do
  1 + numero_externo
end

# Saída: {:+, [...], [1, {:numero_externo, [], Elixir}]}
```

**Analise a AST:** O Elixir não leu o valor `50`. Ele criou uma AST que diz: *"Some 1 com uma variável chamada `numero_externo`"*.
Se você tentar rodar esse código em um lugar onde a variável `numero_externo` não existe, vai dar erro de `undefined variable`.

### A Solução: A "Vacina" `unquote`

Para consertar isso, precisamos dizer ao compilador:

> "Ei, pausa o congelamento (quote) rapidinho! Avalie essa variável numero_externo agora, pegue o valor dela (50) e injete dentro da árvore."
> 

Isso é o `unquote`.

**A Analogia Perfeita:**
Pense em interpolação de Strings.

- `quote` é como as aspas `""`.
- `unquote` é como a interpolação `#{}`.

```elixir
# String
nome = "João"
frase = "Olá #{nome}"  # Resultado: "Olá João"

# AST (Macro)
valor = 50
ast = quote do
  1 + unquote(valor)
end
# Resultado da AST: {:+, [...], [1, 50]}
```

**Olhe a diferença:** Agora o segundo argumento da soma é o **número 50**, e não uma referência a uma variável.

---

### Resumo da Ferramenta

Para escrever macros, você precisa dominar esse "Vem e Vai" mental:

1. **`quote do ... end`**: Você entra no modo "Roteirista". Tudo aqui vira dado (AST) e nada é executado.
2. **`unquote(...)`**: Você volta momentaneamente para o modo "Execução". O que estiver aqui dentro é calculado na hora e o resultado é colado dentro da AST.

**Próximo Passo:**
Agora que sabemos gerar a árvore (`quote`) e injetar valores nela (`unquote`), estamos prontos para encapsular isso dentro de um `defmacro` e criar nossa primeira extensão da linguagem: o comando `unless`.

Agora que dominamos a **teoria** (AST) e as **ferramentas** (`quote`/`unquote`), vamos para a **Engenharia**.

Vamos construir uma funcionalidade que o Elixir tem, mas vamos fingir que não tem: o comando `unless` (a menos que).
Lógica: `unless(x)` é semanticamente idêntico a `if(!x)`.

---

## 4. Prática de Engenharia: Criando a Macro `unless`

Neste laboratório, você vai sentir o poder de estender a linguagem. Você não vai criar uma função; você vai criar uma **nova palavra-chave**.

### Passo 1: O Ambiente

Crie um projeto novo para isolarmos nossos experimentos:

```bash
mix new meta_lab
cd meta_lab
iex -S mix
```

### Passo 2: A Implementação (`defmacro`)

Crie o arquivo `lib/estruturas_controle.ex`.

Aqui, usaremos o `defmacro` em vez de `def`.
O compilador sabe que, ao encontrar um `defmacro`, ele deve executar esse código **durante a compilação** e substituir a chamada pelo resultado (a AST) que ele retornar.

```elixir
defmodule EstruturasControle do
  # A macro recebe a AST da condição e a AST do bloco 'do'
  defmacro unless(condicao, do: bloco) do
    # O quote abre o modo "Roteirista": vamos gerar código.
    quote do
      # Aqui dentro, escrevemos o código FINAL que queremos gerar.
      # Transformamos o 'unless' em um 'if' com a lógica invertida (!).
      
      # IMPORTANTE: Usamos unquote() para injetar o que o usuário digitou.
      # Se não usássemos unquote, o Elixir procuraria uma variável chamada 
      # 'condicao' no escopo, e não a expressão que o usuário passou (ex: 1 > 2).
      if !unquote(condicao) do
        unquote(bloco)
      end
    end
  end
end
```

### Passo 3: O Teste de Fogo (Runtime)

Abra o terminal do projeto: `iex -S mix`.

Para usar macros de outro módulo, somos obrigados a usar `require`. Isso garante que o módulo da macro seja compilado **antes** do módulo que a usa.

```elixir
# 1. Carregue o módulo
require EstruturasControle

# 2. Teste a lógica (1 == 2 é Falso, então o unless deve executar)
EstruturasControle.unless 1 == 2 do
  IO.puts "Funcionou! A matemática ainda é lógica."
end
# Saída: "Funcionou! A matemática ainda é lógica."

# 3. Teste o inverso (1 == 1 é Verdadeiro, o unless deve ignorar)
EstruturasControle.unless 1 == 1 do
  IO.puts "Isso não deve aparecer."
end
# Saída: nil
```

---

## 5. Depuração: O Raio-X da Expansão

Como engenheiro, você não deve confiar em mágica. Você precisa ver o código gerado.
O Elixir possui a função `Macro.expand_once/2`, que simula um único passo do trabalho do compilador e te mostra o resultado imediato da expansão da nossa macro.

Ainda no IEx, digite:

```elixir
# 1. Vamos criar a AST da nossa chamada, sem executar
ast = quote do
  EstruturasControle.unless(true, do: IO.puts("Oi"))
end

# 2. Agora, pedimos para o Elixir expandir essa macro UMA vez
codigo_gerado = Macro.expand_once(ast, __ENV__)

# 3. Vamos converter a AST de volta para String para lermos
Macro.to_string(codigo_gerado) |> IO.puts
```

**Resultado no Terminal:**

```elixir
if !true do
  IO.puts("Oi")
end
```

**Conclusão Visual:**
Veja que o `unless` desapareceu completamente!
O que sobrou foi um `if` nativo com a negação `!`.

!!! warning "`expand_once` vs. `expand`"
    Se você usar `Macro.expand/2` em vez de `Macro.expand_once/2`, o Elixir não para no `if`: como `if` **também é uma macro** definida no `Kernel`, `Macro.expand/2` continua expandindo recursivamente até não haver mais nenhuma macro para expandir, chegando à forma primitiva real:
    ```elixir
    case !true do
      x when Kernel.in(x, [false, nil]) -> nil
      _ -> IO.puts("Oi")
    end
    ```
    Use `expand_once` quando quiser ver "um passo de cada vez" (bom para entender sua própria macro); use `expand` quando quiser ver a forma final que o compilador realmente executa.

---

**Professor:** Com isso, fechamos o ciclo básico de Metaprogramação:

1. Recebemos código (AST).
2. Manipulamos (envolvemos num `if !`).
3. Retornamos código novo.

**Próximo Passo:**

Agora que você domina a AST, o `quote` e o `unquote`, vamos aplicar isso no caso de uso mais nobre da Metaprogramação em Elixir: **Criar uma DSL (Domain Specific Language)**.

Vamos reconstruir, do zero, uma versão simplificada do **ExUnit**, o framework de testes do Elixir.

**O Objetivo de Engenharia:**
Queremos permitir que outro programador escreva testes usando esta sintaxe limpa, que não parece Elixir padrão:

```elixir
testar "soma basica" do
  assert 5 == 5
end
```

---

## 6. Laboratório Final: O Framework "MiniTest"

Neste laboratório, você vai entender como o Elixir consegue olhar para o código `assert 1 == 2` e dizer: *"Falhou. O lado esquerdo era 1 e o direito era 2"*. (Spoiler: Pattern Matching na AST).

### Passo 1: Criando o Motor do Framework

Crie um novo projeto (se ainda não estiver em um): `mix new mini_framework`.
Crie o arquivo `lib/mini_test.ex`.

Vamos construir em 3 partes. Leia os comentários com atenção de cirurgião.

```elixir
defmodule MiniTest do
  # PARTE 1: A Injeção (__using__)
  # Quando alguém der "use MiniTest", este código roda.
  defmacro __using__(_opts) do
    quote do
      # Injetamos o import automaticamente para o usuário não precisar fazer.
      import MiniTest
    end
  end

  # PARTE 2: Definindo o Teste
  # Recebe uma descrição (string) e o bloco de código.
  defmacro testar(descricao, do: bloco) do
    # Engenharia: Precisamos transformar a string "soma x" em um nome de função :test_soma_x
    nome_limpo = String.replace(descricao, " ", "_") |> String.downcase()
    nome_funcao = String.to_atom("test_#{nome_limpo}")

    quote do
      # Geramos uma função pública dinamicamente com esse nome!
      def unquote(nome_funcao)() do
        IO.puts("🧪 Executando: #{unquote(descricao)}")
        unquote(bloco)
      end
    end
  end

  # PARTE 3: O Assert Inteligente (A Mágica da AST)
  # Aqui usamos Pattern Matching para "desmontar" a comparação ==
  defmacro assert({:==, _contexto, [esquerda, direita]}) do
    quote do
      # 1. Calculamos os valores reais
      valor_esq = unquote(esquerda)
      valor_dir = unquote(direita)

      # 2. Fazemos a verificação
      if valor_esq == valor_dir do
        IO.write(".") # Sucesso (ponto verde)
      else
        # 3. Falha rica em detalhes!
        IO.puts("\n❌ FALHA no teste!")
        IO.puts("   Esquerda: #{valor_esq}")
        IO.puts("   Direita:  #{valor_dir}")
        # Só conseguimos mostrar isso porque temos acesso à AST separada!
      end
    end
  end
end
```

---

### Passo 2: O Consumidor (Escrevendo os Testes)

Agora vamos atuar como o usuário final do seu framework.
Crie o arquivo `lib/meus_testes.ex`.

```elixir
defmodule MeusTestes do
  # 1. Injeta nosso framework (roda o __using__)
  use MiniTest

  # 2. Nossa DSL em ação
  testar "matematica correta" do
    assert 1 + 1 == 2
    assert 10 * 2 == 20
  end

  testar "matematica errada" do
    # Este teste vai falhar propositalmente
    assert 5 == 10
  end
end
```

---

### Passo 3: Execução e Análise (IEx)

Abra o terminal: `iex -S mix`.

Lembre-se: As macros rodaram quando você compilou. Agora, o módulo `MeusTestes` possui duas funções públicas (`test_matematica_correta` e `test_matematica_errada`) que foram geradas invisivelmente.

**1. Rodando o teste que passa:**

```elixir
MeusTestes.test_matematica_correta()
```

**Saída:**

```elixir
🧪 Executando: matematica correta
..
```

**2. Rodando o teste que falha:**

```elixir
MeusTestes.test_matematica_errada()
```

**Saída:**

```elixir
🧪 Executando: matematica errada

❌ FALHA no teste!
   Esquerda: 5
   Direita:  10
```

---

### A Análise do Engenheiro (O "Pulo do Gato")

Pare e pense no que aconteceu no teste de falha.

Se `assert` fosse uma **Função** normal em Java ou Python:

```elixir
# Python
assert(5 == 10) # Recebe False
```

A função receberia apenas `False`. Ela não saberia que os números eram 5 e 10. Ela só poderia dizer: "Falhou".

Como `assert` é uma **Macro** em Elixir:

1. Ela recebeu a AST: `{:==, [], [5, 10]}`.
2. Ela pôde separar o `5` (esquerda) do `10` (direita).
3. Ela gerou um código `if` que imprime esses valores separadamente em caso de erro.

Isso é **Introspecção de Código**. É por isso que o Elixir não precisa de bibliotecas de "Assertion" complexas. A própria linguagem entende o código que você escreveu.

---

### Conclusão do Curso de Metaprogramação

Você acabou de construir a base do `ExUnit`, um dos frameworks de teste mais elogiados do mundo.

**O que você leva daqui:**

1. **AST:** Código é dado (Tupla).
2. **Quote/Unquote:** As ferramentas para manipular essa tupla.
3. **Defmacro:** O momento (Compilação) onde a mágica ocorre.
4. **Responsabilidade:** Você viu o poder. Use-o para criar ferramentas expressivas (DSLs), mas evite usá-lo para esconder lógica simples.

Parabéns! Você desbloqueou o nível mais profundo da Engenharia de Software em Elixir.