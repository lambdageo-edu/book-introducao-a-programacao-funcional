# Módulos, Funções e Testes

Vamos mudar a forma como você pensa sobre organizar código. Se você vem de Java, C# ou Python, você está acostumado a pensar em **Classes** (que guardam dados e comportamentos). Em Elixir, nós separamos isso:

1. **Dados** são imutáveis e transparentes.
2. **Módulos** são apenas agrupadores de funções.
3. **Funções** transformam dados.

Vamos construir um sistema de **Avaliação Escolar** do zero.

## 1. O Ambiente de Trabalho (Mix)

Em engenharia de software, não criamos arquivos soltos. Precisamos de uma estrutura padrão para compilação, testes e dependências. Em Elixir, essa ferramenta é o **Mix**.

### Passo 1: Criando o Projeto

Abra seu terminal e digite o comando abaixo. Ele vai criar a estrutura de pastas padrão da indústria.

```bash
mix new escola_elixir
cd escola_elixir
```

Ao rodar o comando `mix new`, o Elixir não cria apenas pastas aleatórias. Ele estabelece uma convenção arquitetural usada por toda a comunidade. Entender isso agora vai te salvar horas no futuro.

Abra a pasta no VS Code. Você verá:

- **`lib/` (A Fábrica):**
    - Aqui vive 100% do seu código de produção.
    - **Diferença importante:** Em Elixir, compilamos os arquivos `.ex` (Elixir files).
    - O Mix compila tudo o que está aqui para bytecode da BEAM para garantir performance máxima.
    - *Regra de Ouro:* A estrutura de pastas dentro de `lib` deve espelhar o nome dos seus módulos. Se o módulo é `Escola.Financeiro.Boleto`, o arquivo deve ser `lib/escola/financeiro/boleto.ex`.
- **`test/` (O Controle de Qualidade):**
    - Aqui vivem seus scripts de teste.
    - Observe que a extensão é `.exs` (Elixir **Script**).
    - **Por que `.exs`?** Arquivos de teste não precisam ser compilados para bytecode otimizado, pois eles mudam com frequência e rodam apenas em desenvolvimento. O Elixir os interpreta em tempo de execução, o que é mais rápido para o ciclo de desenvolvimento (escreve -> testa -> reescreve).
- **`mix.exs` (O Gerente do Projeto):**
    - Este é o coração da configuração. É o equivalente ao `pom.xml` (Maven/Java) ou `package.json` (Node.js).
    - Ele tem duas responsabilidades principais:
        1. **Project:** Define versão (`0.1.0`), nome da app e configurações do compilador.
        2. **Deps:** Lista as bibliotecas externas (dependências) que seu projeto vai baixar e usar.
- *(Oculto/Gitignore)* **`_build` e `deps`**:
    - Você verá essas pastas aparecerem depois de compilar. Não mexa nelas. `deps` é onde as bibliotecas baixadas ficam, e `_build` é onde estão os arquivos compilados (binários). Nunca commite essas pastas no Git.

### 1.1. Aridade: A Identidade da Função (vs. Sobrecarga)

Se você vem de linguagens como Java, C# ou C++, provavelmente está acostumado com o conceito de **Sobrecarga de Métodos (Method Overloading)**. Nessas linguagens, você pode ter vários métodos com o mesmo nome, e o compilador decide qual usar analisando os tipos e a quantidade de argumentos. Para o sistema, eles são variações do mesmo comportamento.

Em Elixir (e na máquina virtual Erlang/BEAM), o conceito é diferente e mais estrito. Nós chamamos de **Aridade**.

A aridade é o número de argumentos que uma função aceita. Em Elixir, o nome da função + a aridade formam a **identidade única** daquela função.

| **Conceito** | **Linguagem OO (Java/C#)** | **Elixir** |
| --- | --- | --- |
| **Definição** | `calcular(int a)` e `calcular(int a, int b)` | `calcular/1` e `calcular/2` |
| **Identidade** | São o mesmo método, apenas “sobrecarregado”. | São duas funções completamente distintas. |
| **Resolução** | O compilador infere pelos tipos dos argumentos. | O runtime busca a função exata pelo **nome** e **número de argumentos** (arity). |

!!! tip "Por que isso importa ?"
    Ao ler uma documentação ou um erro no terminal, você verá algo como `undefined function calcular/1`. Isso diz especificamente: "Eu até conheço a `calcular/2`, mas você chamou uma versão de 1 argumento que não existe". Tratar funções de aridades diferentes como entidades diferentes nos dá precisão absoluta no rastreamento de execução.

## 2. Organização e Namespaces (O Módulo)

Antes de escrevermos o código, precisamos entender **onde** ele vai morar.

### 2.1. Teoria: Módulos como Namespaces

Se você vem do Java ou C#, está acostumado com **Packages** ou **Namespaces** (ex: `br.com.escola.financeiro.Boleto`). Isso serve para garantir que o seu "Boleto" não se confunda com o "Boleto" de outra biblioteca que você instalou.

Em Elixir, usamos a **Notação de Ponto** para criar essa hierarquia.
O módulo não é apenas o nome da funcionalidade, é o **endereço** dela no sistema.

- `EscolaElixir` -> É o nosso namespace raiz (O nome do app).
- `EscolaElixir.Boletim` -> É um submódulo específico.

### 2.2. A Regra de Ouro: Arquivos vs. Módulos

Para manter a sanidade mental em grandes projetos, o Elixir adota uma convenção estrita de mapeamento entre o **Sistema de Arquivos** e o **Nome do Código**.

| **Contexto** | **Convenção** | **Exemplo** |
| --- | --- | --- |
| **No Código (Módulo)** | CamelCase (PascalCase) | `EscolaElixir.Boletim` |
| **No Arquivo (Pasta)** | snake_case | `lib/escola_elixir/boletim.ex` |

!!! tip "Por que minúsculo no arquivo?"
    Sistemas operacionais (Windows vs Linux) tratam letras maiúsculas/minúsculas de formas diferentes. Usar sempre `snake_case` nos arquivos evita bugs bizarros de "Arquivo não encontrado" ao fazer deploy em servidores Linux.

### Passo 2: Criando o Módulo de Boletim

Agora vamos aplicar a teoria. Observe que vamos traduzir os pontos (`.`) do nome do módulo para barras (`/`) no sistema de pastas.

1. Crie a pasta: `lib/escola_elixir` (caso não exista).
2. Crie o arquivo: `lib/escola_elixir/boletim.ex`.

Digite o código abaixo:

```elixir
# O nome do módulo reflete a hierarquia: App -> Contexto
defmodule EscolaElixir.Boletim do
  @moduledoc """
  Módulo responsável por cálculos de notas.
  """

  # def define uma função pública (visível para qualquer outro módulo)
  def calcular_media(nota1, nota2, nota3) do
    (nota1 + nota2 + nota3) / 3
  end
end
```

### Vamos testar?

Como alteramos a estrutura de arquivos, precisamos garantir que o Mix encontre tudo.

1. No terminal, dentro da pasta do projeto, inicie o ambiente interativo:
    
    `iex -S mix`
    
    *(O `-S mix` diz: "Carregue o IEx, mas antes compile e traga todo o meu projeto Mix junto".)*
    
2. Chame a função pelo seu **endereço completo** (Namespace):
    
    ```elixir
    iex> EscolaElixir.Boletim.calcular_media(10, 5, 6)
    7.0
    ```
    

No vscode:

![image.png](img-elixir-modulos/image.png)

**Se funcionou, parabéns!** Você acabou de criar sua primeira estrutura modular respeitando os padrões de engenharia da comunidade Elixir.

## 3. Pattern Matching e Guards: O Superpoder do Elixir

Aqui chegamos ao divisor de águas. Se em Java ou Python o fluxo de controle é imperativo (*"Se x for isso, faça aquilo"*), em Elixir e Erlang ele é declarativo (*"Para este x, a resposta é essa"*).

### 3.1. Teoria: O Diferencial da BEAM

Na maioria das linguagens, você entra na função e depois decide o que fazer com vários `if/else`.
Em Elixir, **a decisão acontece antes da função começar.**

Imagine que sua função tem várias "portas de entrada".

1. O dado chega.
2. O Elixir tenta encaixar o dado na **Porta 1**. Encaixou? Entra e executa.
3. Não encaixou? Tenta a **Porta 2**.
4. E assim por diante.

Isso elimina a complexidade ciclomática (ninhos de `if`) e torna o código extremamente legível.

### 3.2. O Papel dos Guards (`when`)

O Pattern Matching sozinho é ótimo para igualdade exata (ex: "A nota é 10?"). Mas e se precisarmos de comparações, como "A nota é maior que 7"?

O Pattern Matching **não sabe fazer contas**. Ele só compara formas.
Para cobrir essa lacuna, usamos **Guards** (a palavra-chave `when`).

- **Pattern Matching:** Verifica a *forma* e *valores exatos*.
- **Guards:** Refinam a entrada com testes lógicos (`>`, `<`, `is_integer`, etc).

É como um segurança na porta da balada:

1. **Pattern:** "Você tem ingresso?" (Verifica se o dado existe).
2. **Guard:** "Você é maior de 18 anos?" (Valida a condição).

---

### Passo 3: Implementando as Regras de Aprovação

Vamos ver isso na prática. Edite o arquivo `lib/escola_elixir/boletim.ex`.
Apague ou modifique a lógica antiga para usar **Cláusulas de Função**.

```elixir
  # --- Lógica de Situação ---

  # PORTA 1: Match Exato (Pattern Matching Puro)
  # O Elixir verifica: "O argumento é exatamente o float 10.0?"
  # Se for, entra aqui e ignora o resto.
  def verificar_situacao(10.0), do: "Aprovado com Louvor"

  # PORTA 2: Match com Guard (Refinamento)
  # O pattern "media" aceita qualquer coisa, mas o GUARD barra se não for >= 7.
  def verificar_situacao(media) when media >= 7.0 do
    "Aprovado"
  end

  # PORTA 3: Outro Guard
  # Note que não precisamos testar se é < 7, pois se chegou aqui,
  # é porque falhou na porta anterior!
  def verificar_situacao(media) when media >= 5.0 do
    "Recuperação"
  end

  # PORTA 4: O "Catch-all" (Pega-tudo)
  # Se nada acima funcionou, cai aqui.
  # O underscore (_media) avisa ao compilador: "Sei que tem um dado, mas não vou usá-lo".
  def verificar_situacao(_media), do: "Reprovado"
```

### A Regra de Ouro: A Ordem Importa

Em linguagens com `Overloading` (Sobrecarga), o compilador escolhe o método mais específico. **Em Elixir, não.** O Elixir é sequencial (top-down).

Se você colocar a regra do "Reprovado" (que aceita qualquer coisa) no topo do arquivo, **nenhum aluno passará de ano**, pois o Elixir vai casar tudo na primeira regra e parar de procurar.

### Vamos testar?

No terminal (IEx), você não precisa sair e entrar de novo. Use o comando `recompile()`:

1. Recompile o projeto:
    
    `iex> recompile()`
    
2. Teste as diferentes “portas”:
    
    ```elixir
    iex> EscolaElixir.Boletim.verificar_situacao(10.0)
    "Aprovado com Louvor"
    
    iex> EscolaElixir.Boletim.verificar_situacao(7.5)
    "Aprovado"
    
    iex> EscolaElixir.Boletim.verificar_situacao(6.0)
    "Recuperação"
    
    iex> EscolaElixir.Boletim.verificar_situacao(3.0)
    "Reprovado"
    ```
    

Veja como o código fica limpo? Sem chaves aninhadas `{{}}`, sem `else if`, apenas uma lista de regras de negócio claras.

## 4. Encapsulamento e Funções Privadas (`defp`)

Na engenharia de software, existe um princípio vital: **Exponha apenas o necessário.**
Imagine um restaurante: o cliente escolhe o prato no cardápio (API Pública), mas ele não entra na cozinha para cortar a cebola (Implementação Privada).

Em Elixir, usamos:

- `def`: Para o que deve ser acessível de fora (O Cardápio).
- `defp`: Para funções auxiliares internas (A Cozinha).

### Passo 4: Escondendo a Lógica

Vamos refatorar nosso código. Queremos que o usuário chame apenas `gerar_relatorio`, e o módulo se vire para calcular a média e verificar a regra.

Edite `lib/escola_elixir/boletim.ex`. Mude a visibilidade das funções de lógica para `defp` e crie a função pública:

```elixir
defmodule EscolaElixir.Boletim do
  # --- API PÚBLICA (A única coisa que o mundo externo vê) ---

  def gerar_relatorio(n1, n2, n3) do
    # Abordagem Clássica (Estilo Imperativo/Procedural)
    # Salvamos o resultado em variáveis passo a passo
    lista_notas = [n1, n2, n3]
    media = calcular_media_interna(lista_notas)
    situacao = verificar_situacao(media)

    situacao # Retorna a última linha
  end

  # --- FUNÇÕES PRIVADAS (O motor interno) ---

  # Mudei de 'def' para 'defp'. Tente chamar isso no IEx e veja o erro!
  defp calcular_media_interna(lista_notas) do
    Enum.sum(lista_notas) / length(lista_notas)
  end

  # Estas funções também viraram privadas
  defp verificar_situacao(10.0), do: "Aprovado com Louvor"
  defp verificar_situacao(media) when media >= 7.0, do: "Aprovado"
  defp verificar_situacao(media) when media >= 5.0, do: "Recuperação"
  defp verificar_situacao(_), do: "Reprovado"
end
```

### Vamos testar a Segurança?

Essa é a hora de provar que o `defp` funciona.

1. Recompile: `recompile()`
2. Tente chamar a função pública (deve funcionar):
    
    ```elixir
    iex> EscolaElixir.Boletim.gerar_relatorio(8, 8, 8)
    "Aprovado"
    ```
    
3. Agora, tente chamar a função privada (deve falhar):
    
    ```elixir
    iex> EscolaElixir.Boletim.verificar_situacao(5.0)
    ** (UndefinedFunctionError) function EscolaElixir.Boletim.verificar_situacao/1 is undefined or private
    ```
    

Viu? O erro diz **"undefined or private"**. Você encapsulou a lógica com sucesso.

## 5. O Operador Pipe (`|>`) e a Linha de Produção

Agora olhe para a função `gerar_relatorio` que escrevemos acima. Ela tem muitas variáveis temporárias (`lista_notas`, `media`, `situacao`). Em sistemas grandes, isso fica difícil de ler.

Em Programação Funcional, pensamos em **Transformação de Dados**. O dado entra bruto e passa por uma "Esteira de Fábrica", sofrendo modificações em cada etapa.

Para isso, o Elixir tem o **Operador Pipe** (`|>`).

### 5.1. Teoria: Como o Pipe Funciona?

O operador `|>` pega o resultado da expressão à esquerda e o injeta como o **primeiro argumento** da função à direita.

- **Sem Pipe (Leitura de dentro para fora):**`funcao_C(funcao_B(funcao_A(dado)))`*(Difícil de ler, parece matemática complexa)*
- **Com Pipe (Leitura natural):**`dado |> funcao_A() |> funcao_B() |> funcao_C()`*(Lê-se: Pegue o dado, passe na A, depois na B, depois na C)*

### Passo 5: Refatorando para Pipeline

Vamos limpar nossa função pública. Edite o `gerar_relatorio` no `lib/escola_elixir/boletim.ex`:

```elixir
  def gerar_relatorio(n1, n2, n3) do
    # O dado flui visualmente de cima para baixo
    [n1, n2, n3]
    |> calcular_media_interna()
    |> verificar_situacao()
  end
```

!!! tip "Por que isso é melhor ?"
    1. **Clareza Cognitiva:** Você vê a ordem exata das operações.
    2. **Imutabilidade:** Não criamos variáveis desnecessárias que poderiam ser alteradas por engano.
    3. **Debug:** É muito fácil comentar uma linha do pipe para testar o fluxo.

    **Teste novamente no IEx (`recompile()`)**. O comportamento externo é idêntico, mas o código interno agora é "Elixir Idiomático".

## 6. Documentação: Escrevendo, Lendo e Publicando 📖

Em Engenharia de Software com Elixir, a documentação tem três propósitos:

1. **Explicação:** Para humanos lerem.
2. **Teste:** Para o computador validar (`Doctests`).
3. **Visualização:** Para gerar sites estáticos navegáveis.

### Passo 6.1: Escrevendo a Documentação

Primeiro, garantimos que o código fonte tenha as diretivas `@moduledoc` e `@doc`. (Como você já fez acima, vamos manter o foco, mas observe o código abaixo).

Edite `lib/escola_elixir/boletim.ex`:

```elixir
defmodule EscolaElixir.Boletim do
  @moduledoc """
  Módulo responsável pela lógica de avaliação de alunos.

  Esta é a **API pública** que o restante do sistema deve usar.
  """

  @doc """
  Gera uma string com a situação final do aluno baseada em 3 notas.

  ## Exemplo de Uso (Doctest)

      iex> EscolaElixir.Boletim.gerar_relatorio(10, 10, 10)
      "Aprovado com Louvor"

      iex> EscolaElixir.Boletim.gerar_relatorio(5, 5, 5)
      "Recuperação"

  O resultado é o status final após o processamento da pipeline.
  """
  def gerar_relatorio(n1, n2, n3) do
    # ... código anterior (pipeline) continua aqui ...
    [n1, n2, n3]
    |> calcular_media_interna()
    |> verificar_situacao()
  end

  # ... funções privadas continuam ...
end
```

### Passo 6.2: Lendo no Terminal (O Modo "Hacker")

Enquanto você está programando, você não quer sair do terminal para ler um PDF ou site. O Elixir tem o helper **`h`** (help) embutido.

1. Vá ao seu terminal e rode o projeto:
    
    `iex -S mix`
    
2. Para ler a documentação do **Módulo** (`@moduledoc`), digite:
    
    `iex> h EscolaElixir.Boletim`
    
    *O terminal vai imprimir o texto que você escreveu no `@moduledoc`, formatado e colorido.*
    
3. Para ler a documentação da **Função** (`@doc`), digite:
    
    `iex> h EscolaElixir.Boletim.gerar_relatorio`
    
    *O terminal mostrará a explicação, os argumentos e os exemplos.*
    

> Dica de Produtividade: Isso funciona para qualquer módulo da linguagem. Tente digitar h Enum ou h Enum.map para ver como a documentação oficial do Elixir é feita.
> 

### Passo 6.3: Gerando o Site HTML (O Modo Profissional)

Para entregar o projeto ou compartilhar com outros times, geramos uma página HTML estática (igual à documentação oficial do Elixir). Para isso, precisamos instalar uma ferramenta chamada **ExDoc**.

**1. Adicionando a Dependência**
Abra o arquivo `mix.exs` na raiz do projeto. Procure a função `deps` (lá no final do arquivo) e adicione a linha do `ex_doc`:

```elixir
  defp deps do
    [
      # Adicione esta linha dentro da lista:
      {:ex_doc, "~> 0.31", only: :dev, runtime: false}
    ]
  end
```

**2. Instalando e Gerando**
Volte ao terminal (saia do IEx com CTRL+C, CTRL+C) e execute:

1. Baixar a biblioteca nova:
    
    `mix deps.get`
    
2. Compilar e gerar a documentação:
    
    `mix docs`
    

**3. Visualizando o Resultado**
O comando acima criou uma pasta nova chamada `doc/` dentro do seu projeto.

- Abra a pasta `doc` no seu gerenciador de arquivos.
- Dê um duplo clique no arquivo **`index.html`**.

**O que você verá:**
Uma página web moderna, com barra lateral de navegação, busca instantânea e modo escuro, contendo exatamente o texto e os exemplos que você escreveu no código.

![image.png](img-elixir-modulos/image%201.png)

---

### 6.4. O Segredo do `iex>` (Doctests)

Pare um minuto e olhe para o código que você acabou de escrever dentro do `@doc`.

Você deve ter notado que usamos o prefixo `iex>` antes das chamadas de função.
À primeira vista, isso parece apenas um exemplo visual, uma "colinha" para quem for ler a documentação saber como usar sua função.

```elixir
  ## Exemplo de Uso
      iex> EscolaElixir.Boletim.gerar_relatorio(10, 10, 10)
      "Aprovado com Louvor"
```

**Mas aqui está a genialidade do Elixir:**
Isso não é apenas texto. Isso é código executável.
O compilador olha para essa linha e entende:

1. **Ação:** "Vou rodar `EscolaElixir.Boletim.gerar_relatorio(10, 10, 10)`".
2. **Expectativa:** "O resultado **TEM** que ser `"Aprovado com Louvor"`".

Se o código retornar qualquer outra coisa, o teste falha. Isso garante que a sua documentação nunca minta. Chamamos isso de **Doctest**.

---

### 6.5. Ativando os Testes (Onde colocar o `doctest`?)

Se você rodar `mix test` agora, vai perceber que o Elixir **ainda não está testando** o seu Módulo Boletim. Ele provavelmente mostrará apenas 1 teste (o padrão do projeto).

Por quê? Porque precisamos avisar ao framework de testes (ExUnit) para ler a documentação daquele módulo específico.

**A Regra da Engenharia:**
Para cada arquivo na pasta `lib/`, criamos um arquivo correspondente na pasta `test/`.

### Passo a Passo:

1. Crie um novo arquivo: `test/escola_elixir/boletim_test.exs`*(Note que a pasta espelha a estrutura da `lib` e o arquivo termina em `_test.exs`)*.
2. Digite o código abaixo. É aqui que ligamos a chave dos Doctests:

```elixir
defmodule EscolaElixir.BoletimTest do
  use ExUnit.Case

  # A MÁGICA ACONTECE AQUI:
  # Esta linha diz: "Vá no módulo Boletim, leia a documentação
  # e execute todas as linhas que começam com iex>"
  doctest EscolaElixir.Boletim
end
```

1. Agora, vá ao terminal e rode:
    
    `mix test`
    

**O Resultado:**
Agora você deve ver algo como `2 doctests` (ou mais) passando.
Isso prova que o texto que você escreveu para humanos também serviu para o computador testar seu código.

---

## 7. Testes de Unidade (O "JUnit" do Elixir) 🧪

Agora você pode perguntar: *"Professor, se eu já tenho o Doctest, para que preciso de mais testes?"*

O **Doctest** serve para documentar o "caminho feliz" (o uso básico). Mas você não quer poluir sua documentação com 50 linhas de exemplos testando erros, valores nulos ou casos complexos.

Para isso, usamos os **Testes de Unidade** padrão. É o equivalente ao que você faria com **JUnit** (Java) ou **PyTest** (Python). Eles ficam no mesmo arquivo de teste, mas dentro de blocos `test`.

### Passo 7.1: Criando Cenários de Teste

Vamos expandir o arquivo `test/escola_elixir/boletim_test.exs` que acabamos de criar. Vamos adicionar testes para garantir que as regras de limite (notas 5 e 7) funcionam exatamente como esperado.

Edite o arquivo:

```elixir
defmodule EscolaElixir.BoletimTest do
  use ExUnit.Case

  # 1. Valida a documentação
  doctest EscolaElixir.Boletim

  # 2. Testes de Unidade (Cenários Específicos)

  test "deve reprovar aluno quando a média for muito baixa" do
    # Arrange & Act (Preparar e Agir)
    resultado = EscolaElixir.Boletim.gerar_relatorio(2, 3, 2)

    # Assert (Verificar) - O equivalente ao assertEquals do Java
    assert resultado == "Reprovado"
  end

  test "deve garantir recuperação no limite exato da nota 5.0" do
    # Queremos ter certeza que 5.0 não reprova e nem aprova, vai para recuperação
    resultado = EscolaElixir.Boletim.gerar_relatorio(5, 5, 5)
    assert resultado == "Recuperação"
  end
end
```

### Rodando a Bateria Completa

Volte ao terminal:

`mix test`

Agora o Mix vai executar tudo junto:

1. Vai ler os exemplos da documentação (`doctest`).
2. Vai rodar os blocos de teste (`test`).

Se tudo ficar verde, você atingiu um nível profissional de qualidade de código: documentado e testado em camadas.

## 8. Reutilização de Código: `use`, `import` e `require` 🧩

Você deve ter notado que no nosso arquivo de teste escrevemos:
`use ExUnit.Case`

Se você vem de Java, pensou: *"Ah, isso é um `extends TestCase`!"*.
**Cuidado.** Em Elixir não existe herança de classes. O que aconteceu ali não foi uma relação de Pai/Filho.

Vamos desmistificar as três formas de interagir com outros módulos.

### 8.1. O `use` (A Injeção de Código)

O `use` é a ferramenta mais agressiva de acoplamento. Ele é uma **Macro**.
Quando você diz `use ExUnit.Case`, você está dizendo ao compilador:

> "Vá até o módulo ExUnit.Case, copie o código que está configurado lá e injete dentro do meu arquivo agora."
> 

É por isso que as palavras mágicas `test` e `assert` aparecem no seu arquivo de teste sem você tê-las definido. Elas foram injetadas.

### Passo 8.1: Criando nosso próprio `use`

Vamos supor que todos os sistemas da escola precisem ter uma função de copyright padrão. Em vez de copiar e colar em todo arquivo, criamos um injetor.

1. Crie o arquivo `lib/escola_elixir/sistema.ex`:

```elixir
defmodule EscolaElixir.Sistema do
  # A mágica acontece aqui. __using__ é o que o 'use' procura para executar.
  defmacro __using__(_opts) do
    quote do
      # Tudo que estiver aqui dentro será "transplantado" para quem usar este módulo
      def copyright do
        "Sistema Escolar v1.0 - Todos os direitos reservados"
      end
    end
  end
end
```

1. Agora, injete isso no `lib/escola_elixir/boletim.ex`. Adicione a linha no topo:

```elixir
defmodule EscolaElixir.Boletim do
  use EscolaElixir.Sistema  # <--- A Injeção acontece aqui!
  # ... resto do código ...
end
```

1. **Teste no IEx (`recompile()`):**
    
    ```elixir
    iex> EscolaElixir.Boletim.copyright()
    "Sistema Escolar v1.0 - Todos os direitos reservados"
    ```
    
    Veja: A função nasceu dentro do Boletim, mesmo sem termos escrito ela lá explicitamente.
    

### 8.2. O `import` (Trazendo para Perto com Segurança)

Diferente do `use` (que injeta código), o `import` apenas permite que você chame as funções de outro módulo sem digitar o nome completo dele.

Mas cuidado: importar um módulo inteiro pode trazer centenas de funções e causar conflito com as suas. Por isso, usamos filtros.

### A Analogia para quem vem de fora

Para entender o `import` do Elixir, compare com o que você já conhece:

| **Linguagem** | **Sintaxe** | **O equivalente em Elixir** |
| --- | --- | --- |
| **Java** | `import static java.lang.Integer.*;` | `import Integer` *(Importa tudo – perigoso)* |
| **Java** | `import static java.lang.Integer.parseInt;` | `import Integer, only: [parse: 1]` *(Importa só um – recomendado)* |
| **Python** | `from math import *` | `import :math` |
| **Python** | `from math import sqrt` | `import :math, only: [sqrt: 1]` |

> Nota Técnica: Em Elixir, quando filtramos, precisamos informar a Aridade (número de argumentos) da função. Não basta dizer o nome.
> 

### Usando `only` (Lista Branca) e `except` (Lista Negra)

Vamos ver na prática com o módulo `List` do Elixir, que tem muitas funções.

**1. `only:` (Traga APENAS estes)**
Esta é a forma recomendada. Você declara explicitamente o que seu código vai usar.

```elixir
defmodule MinhaLista do
  # Quero usar a função 'last/1' e 'duplicate/2' do módulo List
  # sem digitar "List." toda vez.
  import List, only: [last: 1, duplicate: 2]

  def duplicar_ultimo(lista) do
    ultimo_item = last(lista)       # Chamada direta (veio do import)
    duplicate(ultimo_item, 2)       # Chamada direta
  end

  def pegar_primeiro(lista) do
    first(lista) # ERRO! A função 'first' não foi importada na lista 'only'.
  end
end
```

**2. `except:` (Traga tudo, EXCETO estes)**
Menos comum, mas útil quando você quer importar um módulo quase todo, mas ele tem uma função com o mesmo nome de uma função sua.

```elixir
defmodule MeuCalculo do
  # O módulo Integer tem a macro is_even e is_odd.
  # Vamos supor que eu já tenha uma função chamada is_odd aqui.
  import Integer, except: [is_odd: 1]

  def verificar(n) do
    is_even(n) # Funciona (veio do Integer)
  end

  def is_odd(n) do
    "Esta é a MINHA função, não a do Integer"
  end
end
```

**3. `only: :functions` ou `only: :macros`**
Às vezes queremos importar apenas as funções e deixar as macros de fora (ou vice-versa).
Exemplo: `import Integer, only: :macros` (Traz `is_odd`, `is_even`, mas não traz funções normais).

---

### Resumo da Etiqueta do Import

Como engenheiro de software, siga esta regra de ouro:

1. **Prefira `alias`:** `List.last(lista)` é explícito e fácil de ler.
2. **Se usar `import`:** Use sempre `only: [...]`. Isso diz para o próximo programador exatamente de onde aquela função misteriosa veio.
3. **Evite `import` limpo:** Fazer `import List` polui seu escopo e deixa seu código frágil a mudanças futuras no módulo importado.

### 8.3. O `require` (Macros e Compilação)

Este é o que mais confunde quem vem de fora.
Em Elixir, existem funções normais e existem **Macros** (código que escreve código).

Se você quiser usar uma Macro definida em outro módulo, você é **obrigado** a avisar o compilador antes, usando `require`. Isso garante que o módulo da macro seja compilado antes do módulo que a usa.

Exemplo clássico: O módulo `Integer` tem uma macro chamada `is_odd` (é ímpar).

```elixir
defmodule TesteImpar do
  require Integer # Sem isso, o código abaixo falha!

  def impar?(numero) do
    Integer.is_odd(numero)
  end
end
```

Se você esquecer o `require`, o Elixir vai reclamar: *"Você está tentando invocar a macro Integer.is_odd/1 mas não a requisitou"*.

---

### 8.4 Organização de Código: O Poder do `alias`

À medida que nosso sistema cresce, os nomes dos módulos tendem a ficar longos e aninhados para respeitar a hierarquia de pastas.

Exemplo: EscolaElixir.Financeiro.Pagamentos.BoletoBancario.

Ninguém quer digitar isso toda vez que for chamar uma função.

Para isso existe o alias (Apelido).

### O que ele faz?

Ele cria um "atalho" para o nome do módulo dentro do arquivo atual.

Importante: Ele NÃO traz as funções para o escopo (como o import). Ele apenas encurta o nome do módulo.

### A Analogia (Java/Python)

| **Linguagem** | **Sintaxe Original** | **O equivalente em Elixir** |
| --- | --- | --- |
| **Java** | `import java.util.List;` *(agora posso escrever `List` em vez de `java.util.List`)* | `alias MeuApp.Util.Lista` *(agora posso escrever `Lista`)* |
| **Python** | `import meuapp.util.lista as l` | `alias MeuApp.Util.Lista, as: L` |
| **C#** | `using Lista = MeuApp.Util.Lista;` | `alias MeuApp.Util.Lista, as: Lista` |

### Passo 8.4: Simplificando Nomes

Vamos supor que, no nosso teste, estejamos cansados de digitar `EscolaElixir.Boletim` toda hora.

Edite o arquivo `test/escola_elixir/boletim_test.exs`:

```elixir
defmodule EscolaElixir.BoletimTest do
  use ExUnit.Case

  # 1. Cria o apelido.
  # A partir desta linha, "Boletim" significa "EscolaElixir.Boletim"
  alias EscolaElixir.Boletim

  test "deve aprovar com alias" do
    # Veja como ficou mais limpo:
    resultado = Boletim.gerar_relatorio(7, 7, 7)
    assert resultado == "Aprovado"
  end
end
```

### Formas de usar o Alias

Existem três maneiras comuns de escrever um alias:

1. O Alias Implícito (Padrão da Indústria)

Quando você omite o as:, o Elixir assume que o apelido será a última parte do nome.

```elixir
alias EscolaElixir.Boletim
# O apelido automático é: Boletim
```

2. O Alias Explícito (Renomeando)

Útil quando você tem dois módulos com o mesmo nome final (ex: Escola.User e Twitter.User).

```elixir
alias EscolaElixir.Boletim, as: Notas
# Agora chamo: Notas.gerar_relatorio(...)
```

3. O Multi-Alias (Agrupando)

Quando você quer trazer vários módulos do mesmo "pai".

```elixir
alias EscolaElixir.{Boletim, Sistema, Financeiro}
# Equivalente a escrever 3 linhas de alias separadas.
```

### Dica de Engenharia: Alias vs Import

Muitos iniciantes perguntam: *"Por que não usar `import` logo de cara e economizar ainda mais digitação?"*

- **Com `alias`:** `Boletim.gerar_relatorio()`
    - Eu sei exatamente que essa função pertence ao módulo Boletim. O código é explícito.
- **Com `import`:** `gerar_relatorio()`
    - De onde veio isso? Do meu módulo? Do módulo importado? De uma macro?
    - Em sistemas com 50 arquivos, o `import` cria confusão mental.

**Regra:** Use `alias` sempre que puder. Deixe o `import` apenas para bibliotecas matemáticas ou de testes.

### 8.5. Resumo da Ópera (Comparativo)

Para organizar sua cabeça de engenheiro, aqui está a hierarquia de acoplamento:

| **Comando** | **O que faz?** | **Analogia Java/C#** | **Frequência de uso** |
| --- | --- | --- | --- |
| **alias** | Apelida o módulo. Ex.: `alias Escola.Boletim` permite usar `Boletim.gerar...` | `import package.Class` | **Muito Alta (90% dos casos)** |
| **import** | Traz funções para o escopo atual **sem prefixo**. | `import static package.Class.*` | **Baixa (use com moderação)** |
| **require** | Garante compilação para usar **macros**. | Não tem equivalente direto. | **Média (só para macros)** |
| **use** | Injeta código e pode alterar o comportamento do módulo. | ~Herança (mas via *injeção*/mixins) | **Média (frameworks e libs)** |

**Próximo Passo:** Agora que entendemos como criar código (Módulos/Funções), garantir qualidade (Testes/Docs) e reutilizar estruturas (`use`/`import`), estamos prontos para o **Grand Finale**: O Desafio Prático "Motor de Compras".

## Desafio Final: O Motor de E-commerce (GeekStore)

**O Cenário:**
Você foi contratado para criar o módulo de "Checkout" da *GeekStore*. O sistema recebe uma lista de produtos e precisa cuspir a nota fiscal final.
Você deve entregar código limpo, testado e documentado.

---

### Passo 1: O Setup Profissional (Mix)

Não vamos fazer arquivos soltos. Crie um projeto novo.
No terminal:

```bash
mix new geek_store
cd geek_store
```

---

### Passo 2: A Arquitetura do Módulo

Crie o arquivo `lib/geek_store/caixa.ex`.
Vamos começar definindo o módulo e usando **boas práticas de importação**.

**Requisito de Engenharia:**
Queremos arredondar os preços finais para 2 casas decimais. O módulo `Float` tem a função `round/2`.

- Em vez de escrever `Float.round(valor, 2)` toda hora, use um **import seguro** (`only:`) para trazer apenas essa função.

```elixir
defmodule GeekStore.Caixa do
  # --- 1. DOCUMENTAÇÃO DO MÓDULO ---
  @moduledoc """
  Responsável pelo processamento de compras e cálculo de descontos da GeekStore.
  """

  # --- 2. IMPORTAÇÃO SEGURA ---
  # Trazemos apenas a função round/2. Se usássemos "import Float", traríamos lixo desnecessário.
  import Float, only: [round: 2]

  # --- 3. API PÚBLICA (DOCUMENTADA COM DOCTESTS) ---
  @doc """
  Recebe uma lista de itens (mapas) e retorna o recibo final.

  ## Regras de Negócio
  - Compras acima de R$ 500.00: 10% de desconto.
  - Compras acima de R$ 200.00: 5% de desconto.
  - Abaixo disso: Sem desconto.

  ## Exemplos (Doctests)

      iex> lista = [%{preco: 100.0}, %{preco: 150.0}]
      iex> GeekStore.Caixa.fechar_conta(lista)
      %{total: 250.0, desconto: 12.5, a_pagar: 237.5}

  """
  def fechar_conta(itens) do
    # SEU CÓDIGO AQUI: Use o Pipeline (|>)
    # Fluxo sugerido:
    # 1. itens |> somar_total()
    # 2.       |> aplicar_desconto()
    # 3.       |> formatar_recibo()
  end

  # --- 4. FUNÇÕES PRIVADAS (defp) ---
  # Implemente a lógica suja aqui embaixo.
  # Lembre-se: Use Pattern Matching e Guards para decidir o desconto!
end
```

---

### Passo 3: Implementando a Lógica (Sua Vez!)

Tente implementar antes de ver a resposta. Lembre-se:

1. **Imutabilidade:** Não tente criar uma variável `soma = 0` e ir somando. Use `Enum.sum` ou `Enum.map`.
2. **Guards:** Use `when valor >= 500` nas funções privadas de desconto.
3. **Pipeline:** Conecte tudo na função pública.

??? note "Gabarito Comentado (A Solução do Professor)"
    
    Aqui está como um engenheiro Elixir resolveria, aplicando todos os conceitos da aula.
    
    **Arquivo:** `lib/geek_store/caixa.ex`
    
    ```elixir
    defmodule GeekStore.Caixa do
      @moduledoc """
      Responsável pelo processamento de compras e cálculo de descontos.
      """
    
      # Importamos apenas o que precisamos para não poluir o namespace
      import Float, only: [round: 2]
    
      @doc """
      Gera a nota fiscal final com descontos aplicados.
    
      ## Exemplos
    
          iex> itens = [%{preco: 100.0}, %{preco: 100.0}] # Total 200
          iex> GeekStore.Caixa.fechar_conta(itens)
          %{total: 200.0, desconto: 10.0, a_pagar: 190.0}
    
      """
      def fechar_conta(itens) do
        # O pipeline conta a história do dado
        itens
        |> calcular_total_bruto()
        |> calcular_valores_finais()
      end
    
      # --- ÁREA RESTRITA (Privada) ---
    
      defp calcular_total_bruto(itens) do
        # Transformamos a lista de Mapas em uma lista de números e somamos
        itens
        |> Enum.map(fn item -> item.preco end)
        |> Enum.sum()
      end
    
      # Aqui aplicamos as regras de negócio.
      # Recebe o total bruto e retorna o mapa final.
      defp calcular_valores_finais(total) do
        valor_desconto = obter_desconto(total)
        a_pagar = total - valor_desconto
    
        %{
          total: round(total, 2),        # Usando a função importada
          desconto: round(valor_desconto, 2),
          a_pagar: round(a_pagar, 2)
        }
      end
    
      # Pattern Matching + Guards (A ordem importa!)
      defp obter_desconto(total) when total >= 500, do: total * 0.10
      defp obter_desconto(total) when total >= 200, do: total * 0.05
      defp obter_desconto(_total), do: 0.0
    end
    ```
    
    ---
    
    ### Passo 4: O Teste Unitário (Aplicando `alias` e `use`)
    
    Agora vamos criar o arquivo de teste para garantir a robustez.
    Aqui você vai usar:
    
    1. `use ExUnit.Case`: Injeção do framework.
    2. `doctest`: Validar a documentação.
    3. `alias`: Para não ter que digitar `GeekStore.Caixa` toda vez.
    
    Crie o arquivo: `test/geek_store/caixa_test.exs`
    
    ```elixir
    defmodule GeekStore.CaixaTest do
      # 1. INJEÇÃO: Traz 'test', 'assert', 'describe' para este módulo
      use ExUnit.Case
    
      # 2. DOCTEST: Valida os exemplos "iex>" escritos no arquivo .ex
      doctest GeekStore.Caixa
    
      # 3. ORGANIZAÇÃO: Cria o apelido. Agora "Caixa" = "GeekStore.Caixa"
      alias GeekStore.Caixa
    
      test "compra simples sem desconto" do
        itens = [%{preco: 50.0}, %{preco: 40.0}]
        resultado = Caixa.fechar_conta(itens) # Usando o Alias!
    
        assert resultado.a_pagar == 90.0
        assert resultado.desconto == 0.0
      end
    
      test "compra VIP com 10% de desconto" do
        itens = [%{preco: 1000.0}]
        resultado = Caixa.fechar_conta(itens)
    
        # Verifica lógica: 1000 - 100 = 900
        assert resultado.a_pagar == 900.0
      end
    end
    ```
    
    ---
    
    ### A Validação Final
    
    Vá ao terminal e execute o comando supremo:
    
    `mix test`
    
    **O que deve acontecer:**
    
    1. O Mix compila seu código (`lib`).
    2. O Mix lê seu arquivo de teste (`test`).
    3. O `doctest` roda os exemplos da documentação.
    4. O `test` roda os cenários específicos.
    
    Se você vir as letras verdes, **Parabéns!**
    
    ### Recapitulando o que você construiu:
    
    - [x]  **Projeto Mix:** Estrutura profissional.
    - [x]  **Namespace:** `GeekStore.Caixa`.
    - [x]  **Import Seguro:** Usou `only:` para trazer `round/2`.
    - [x]  **Pipeline:** Lógica clara com `|>`.
    - [x]  **Guards:** Substituiu `if/else` por lógica declarativa.
    - [x]  **Alias:** Simplificou o teste.
    - [x]  **Doctest:** Documentação que não mente.
    
    Você acabou de escrever código Elixir com nível de produção.