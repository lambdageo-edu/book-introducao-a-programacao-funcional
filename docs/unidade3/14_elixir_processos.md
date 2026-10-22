# Processos

## Módulo 1: Fundamentos de Computação Concorrente na BEAM

**Objetivo:** Compreender o modelo de execução da Máquina Virtual Erlang (BEAM) e contrastá-lo com os modelos tradicionais de Threads de SO e Green Threads (Go).

**1. O Problema da Concorrência Tradicional**

Na engenharia de software clássica (Java, C++, C#), a unidade de concorrência é a **Thread do Sistema Operacional (OS Thread)**.
• **Peso:** Cada thread consome uma quantidade significativa de memória (geralmente alguns MBs apenas para a stack).
• **Gerenciamento:** O Kernel do SO é responsável por agendar essas threads (Context Switching). Trocar de contexto no nível do kernel é uma operação custosa (latência).
• **Memória Compartilhada:** Threads do mesmo processo compartilham o mesmo espaço de memória (Heap).
    ◦ *O Perigo:* Se a Thread A e a Thread B tentam escrever na variável `x` ao mesmo tempo, ocorre uma **Condição de Corrida (Race Condition)**.
    ◦ *A Solução (e o problema dela):* Usamos Mutexes, Semáforos e Locks. Isso introduz complexidade e riscos de **Deadlocks**.

**2. A Solução Elixir: O Modelo de Atores e a BEAM**

O Elixir roda sobre a BEAM. A BEAM não utiliza threads do SO diretamente para executar sua lógica. Ela utiliza **Processos Leves (Lightweight Processes)**.

**2.1. O Processo da BEAM**

• **Peso:** Extremamente leve (inicia com cerca de ~300 words ou ~2.5KB). Você pode rodar milhões deles em um laptop.
• **Isolamento de Memória (Share Nothing):** Cada processo tem sua própria Heap e Stack. O Processo A **não consegue** acessar a memória do Processo B.
• **Comunicação:** A única forma de interação é através de **Troca de Mensagens** (Message Passing). Os dados são copiados de um processo para outro (com exceção de binários grandes, que usam contagem de referência).

**2.2. O Scheduler da BEAM (Escalonamento)**

Isso é crucial para engenheiros: A BEAM roda, geralmente, uma Thread de SO por Núcleo de CPU.
Dentro dessas threads, a BEAM roda seus próprios escalonadores (Schedulers).
• **Escalonamento Preemptivo:** A BEAM atribui a cada processo um número de "reduções" (aproximadamente 2000 chamadas de função). Quando as reduções acabam, a BEAM pausa o processo forçadamente e dá a vez para o próximo.
    ◦ *Vantagem:* Um processo com loop infinito ou cálculo pesado **não trava o sistema**. Em Node.js (single-thread event loop), um cálculo pesado trava tudo. No Elixir, não.

**3. Comparativo Técnico: Elixir vs. Go (Goroutines) vs. Java (Threads)**

Esta é uma dúvida comum de mercado e academia.

| **Característica** | **Java / C# (OS Threads)** | **Go (Goroutines)** | **Elixir (BEAM Processes)** |
| --- | --- | --- | --- |
| **Gerenciamento** | Kernel do SO | Runtime da linguagem (M:N) | Runtime da linguagem (M:N) |
| **Uso de Memória** | Alto (MBs) | Baixo (KBs) | **Muito Baixo (KBs)** |
| **Memória** | Compartilhada (precisa de mutex) | Compartilhada (mas encoraja *channels*) | **Isolada — *Share Nothing*** |
| **Tolerância a Falhas** | Se uma thread crashar, o processo pode cair. | Se houver *panic* não tratado, o programa cai. | **Se um processo crashar, só ele morre. O sistema continua.** |
| **Latência** | Previsível (limitada pelo SO) | Baixa | **Soft Real-Time (latência muito previsível)** |

---

### **Nota para o Engenheiro**

Goroutines (Go) e Processos Elixir (BEAM) são ambos *green threads*, mas com filosofias opostas:

- **Go:** paralelismo com **memória compartilhada**, embora incentive *channels*.
- **Elixir/BEAM:** **isolamento total**, focado em **tolerância a falhas** e sistemas auto-curáveis.

**4. Conceitos Fundamentais**

Precisamos alinhar o vocabulário para as próximas etapas.

**4.1. Concorrência vs. Paralelismo**

• **Concorrência:** É sobre a **estrutura** do programa. É a capacidade de lidar com muitas coisas ao mesmo tempo. (Ex: Um servidor web aceitando 10 mil conexões).
• **Paralelismo:** É sobre a **execução** física. É fazer muitas coisas no mesmo instante de tempo. Requer múltiplos núcleos de CPU.
    ◦ *Em Elixir:* Você escreve código concorrente (muitos processos). A BEAM se encarrega de paralelizar isso automaticamente em todos os núcleos disponíveis.

**4.2. Síncrono vs. Assíncrono**

Em sistemas distribuídos (e no Elixir):
• **Síncrono (Call):** Envio a mensagem e **bloqueio** minha execução até receber a resposta (Request/Response).
• **Assíncrono (Cast):** Envio a mensagem e sigo minha vida ("Fire and forget"). Não sei se a mensagem chegou ou se foi processada na hora.

**4.3. Transparência de Localização (Sistemas Distribuídos)**

Como processos se comunicam apenas por mensagens enviadas para um endereço (PID - Process ID), não importa onde esse processo está.
• Enviar mensagem para o PID `<0.100.0>` (na mesma máquina).
• Enviar mensagem para o PID `<50.100.0>` (em um servidor no Japão).
**O código é exatamente o mesmo.** Isso torna o Elixir uma linguagem naturalmente orientada a sistemas distribuídos (Clusters).

## Módulo 2: Laboratório de Anatomia de Processos

**Objetivo:** Manipular as primitivas de concorrência da BEAM (`spawn`, `send`, `receive`) e entender o ciclo de vida de um processo.

### 1. Preparação do Ambiente

Abra seu terminal. Vamos criar um projeto chamado `lab_concorrencia`.
Isso carrega o ambiente do Mix, que será útil para compilar os módulos que criaremos mais à frente.

```bash
mix new lab_concorrencia
cd lab_concorrencia
iex -S mix
```

*O comando `iex -S mix` abre o shell interativo carregando o contexto do seu projeto.*

---

### 2. Identidade: O PID (Process Identifier)

Em sistemas distribuídos, para enviar uma mensagem, precisamos de um endereço. No Elixir, esse endereço é o **PID**.

No IEx, digite:

```elixir
self()
```

**Saída esperada:** `#PID<0.152.0>` (Os números podem variar).

**Análise do Professor:**

- Este é o endereço do **próprio terminal IEx**. Sim, o shell onde você digita é um Processo Elixir rodando sobre a BEAM.
- O formato `<A.B.C>` geralmente indica:
    - `A`: Nó (Node) onde o processo está rodando (0 = máquina local).
    - `B`: ID sequencial do processo.
    - `C`: Serial (usado quando o B "dá a volta" no contador).

---

### 3. Criação: `spawn` (Vida e Morte)

A função `spawn/1` aceita uma função anônima, cria um processo novo para executá-la em uma stack de memória isolada e retorna o PID dele.

Digite no IEx:

```elixir
pid_calculadora = spawn(fn -> 1 + 1 end)
# Saída: #PID<0.155.0>
```

Agora, verifique se esse processo ainda existe:

```elixir
Process.alive?(pid_calculadora)
# Saída: false
```

**O Conceito de Ciclo de Vida:**
Por que ele morreu?
Diferente de uma Thread Java que pode ficar "idle" (ociosa) gastando recursos, um Processo Elixir é estritamente funcional.

1. Ele nasce.
2. Ele executa a função.
3. Se não há mais instruções, ele termina (exit normal).
4. O Garbage Collector limpa a memória dele imediatamente.

---

### 4. Comunicação Assíncrona: `send` e a Mailbox

Todo processo nasce com uma **Caixa de Correio (Mailbox)** interna.
O envio de mensagens em Elixir é **Assíncrono (Non-blocking)**.

Vamos usar o seu próprio processo do IEx (`self()`) para testar.

1. Armazene seu endereço:
    
    ```elixir
    meu_pid = self()
    ```
    
2. Envie uma mensagem para si mesmo:
    
    ```elixir
    send(meu_pid, {:ola, "Mundo"})
    # Saída: {:ola, "Mundo"}
    ```
    
    *Observe:* A função retorna a mensagem enviada, mas nada foi impresso na tela como "Recebido". Onde está a mensagem?
    
3. Inspecione a memória do processo (Buffer):
Use a função de debug `flush()`:
    
    ```elixir
    flush()
    # Saída:
    # {:ola, "Mundo"}
    # :ok
    ```
    

**Análise do Engenheiro:**
A mensagem ficou armazenada na Heap do processo. O `send` apenas deposita a mensagem e retorna o controle imediatamente para a CPU. O remetente não sabe se a mensagem foi lida. Isso é o fundamento do desacoplamento em sistemas distribuídos.

---

### 5. Sincronização: `receive` (O Bloqueio)

Para ler a mensagem, o processo precisa parar o que está fazendo e verificar a caixa de correio. Isso é feito com `receive`.
**Atenção:** O `receive` é uma operação **bloqueante**. Se a caixa estiver vazia, o processo dorme (não gasta CPU) até chegar algo.

Vamos criar um processo que sabe esperar por uma mensagem.

Digite no IEx:

```elixir
# Definimos a função que o processo vai rodar
funcao_ouvinte = fn ->
  receive do
    {:ping, remetente_pid} ->
      IO.puts "🏓 Recebi PING. Enviando PONG..."
      send(remetente_pid, :pong)
  end
end

# Criamos o processo
pid_ping = spawn(funcao_ouvinte)
```

Verifique se ele está vivo:

```elixir
Process.alive?(pid_ping)
# Saída: true
```

*Ele está vivo, parado na linha do `receive`, aguardando.*

Agora, vamos interagir:

```elixir
send(pid_ping, {:ping, self()})
# Saída no terminal: 🏓 Recebi PING. Enviando PONG...
# Retorno: {:ping, #PID<...>}
```

Verifique se recebemos a resposta (:pong):

```elixir
flush()
# Saída: :pong
```

Verifique se o processo ouvinte ainda vive:

```elixir
Process.alive?(pid_ping)
# Saída: false
```

*Ele processou o `ping`, acabou o bloco `receive`, saiu da função e morreu.*

---

### 6. A Persistência: Recursão de Cauda (O Servidor)

Para criar um sistema (como um Banco ou Webserver), o processo não pode morrer após uma mensagem. Ele deve voltar ao estado de espera.
Em linguagens imperativas, usaríamos `while(true)`. Em Elixir, usamos **Recursão**.

Como estamos dentro do projeto `lab_concorrencia`, vamos criar um arquivo real para isso.

1. Abra seu editor (VS Code).
2. Crie o arquivo `lib/processo_persistente.ex`.

```elixir
defmodule ProcessoPersistente do
  def aguardar_mensagem do
    receive do
      {:mensagem, texto} ->
        IO.puts "📝 Log: #{texto}"
        # AQUI ESTÁ O SEGREDO:
        # Chamamos a função novamente. O processo volta para o topo e para no 'receive'.
        aguardar_mensagem()

      {:parar} ->
        IO.puts "🛑 Encerrando processo..."
        # Não chamamos a recursão. O processo chegará ao fim e morrerá.
    end
  end
end
```

Recompile o projeto para ler o arquivo novo:

```elixir
recompile()
```

ou caso tenha saido:

```bash
iex -S mix
```

Teste a persistência:

```elixir
# 1. Spawn
pid = spawn(ProcessoPersistente, :aguardar_mensagem, [])
```

---

- Entendendo o `spawn/3`
    
    Quando usamos `spawn/3`, estamos usando o padrão **MFA** (Module, Function, Arguments), que é onipresente na máquina virtual do Elixir.
    
    O terceiro parâmetro (`[]`) é a **Lista de Argumentos Iniciais** que serão passados para a função quando ela começar.
    
    ### 1. Por que está vazio `[]` esse exemplo?
    
    Olhe para a definição da função que criamos:
    
    ```elixir
    def aguardar_mensagem do  # <--- Veja: Não recebe nenhum parâmetro aqui
      receive do
        # ...
      end
    end
    ```
    
    Como a função `aguardar_mensagem/0` não pede nenhum dado para começar, nós passamos uma lista vazia `[]` no `spawn`.
    
    ### 2. E se a função precisasse de dados?
    
    Imagine que queremos criar um processo que já nasce sabendo o nome do usuário.
    
    **A função seria:**
    
    ```elixir
    def cumprimentar(nome, idade) do
      IO.puts "Olá, eu sou o #{nome} e tenho #{idade} anos."
    end
    ```
    
    **O Spawn seria:**
    
    ```elixir
    # A lista DEVE ter 2 itens, na mesma ordem dos parâmetros da função
    spawn(Modulo, :cumprimentar, ["Maria", 30])
    ```
    

Agora vamos enviar algumas mensagens:

```elixir
# 2. Enviar múltiplas mensagens
send(pid, {:mensagem, "Primeiro log"})
send(pid, {:mensagem, "Segundo log"})
send(pid, {:mensagem, "Terceiro log"})

# 3. Verificar vida
Process.alive?(pid)
# true (Ele processou tudo e voltou a dormir no receive)

# 4. Parar
send(pid, {:parar})
Process.alive?(pid)
# false
```

**Professor:** Finalizamos o laboratório de primitivas.
O aluno aprendeu:

1. Como criar um PID (`spawn`).
2. Que o envio é "Fire-and-forget" (`send`).
3. Que o recebimento trava o processo (`receive`).
4. Que servidores são apenas loops recursivos infinitos.

## Módulo 3: Construindo um Servidor de Conta

**Objetivo:** Implementar um processo com estado persistente (Stateful). Faremos isso em duas etapas:

1. **A Versão "Raiz" (Manual):** Para entender onde o estado mora (recursão).
2. **A Versão "Profissional" (GenServer):** Para ganhar robustez e padronização.

---

### 1. Etapa A: O Servidor Manual (Entendendo o Loop)

Antes de usarmos ferramentas prontas, vamos criar uma conta bancária usando apenas primitivas: `spawn`, `receive` e recursão. Isso vai provar para você que **não existe mágica**, apenas engenharia.

#### Passo 1: O Loop de Estado

Crie o arquivo `lib/conta_manual.ex`.

Neste código, observe que o **saldo** não é uma variável global. Ele é um **argumento** que é passado de uma volta do loop para a próxima.

```elixir
defmodule ContaManual do
  # Função para iniciar o processo
  def iniciar(saldo_inicial) do
    spawn(__MODULE__, :loop_do_servidor, [saldo_inicial])
  end

  # O CORAÇÃO DO PROCESSO
  # Esta função nunca termina. Ela fica girando eternamente.
  def loop_do_servidor(saldo_atual) do
    # 1. O processo para e espera uma mensagem chegar na caixa de correio
    receive do
      {:depositar, valor} ->
        novo_saldo = saldo_atual + valor
        IO.puts "💰 [Manual] Depositado #{valor}. Novo saldo: #{novo_saldo}"
        
        # A MÁGICA: Reinicia o loop, passando o NOVO saldo
        loop_do_servidor(novo_saldo)

      {:ver_saldo, pid_do_cliente} ->
        # Para responder, precisamos enviar uma mensagem de volta manualmente
        send(pid_do_cliente, {:resposta_saldo, saldo_atual})
        
        # Reinicia o loop mantendo o MESMO saldo
        loop_do_servidor(saldo_atual)
    end
  end
end
```

#### Testando a Versão Manual (IEx)

Abra o terminal e veja a dificuldade de comunicação:

```elixir
# 1. Iniciar
pid = ContaManual.iniciar(0)

# 2. Depositar (Fácil, só enviar)
send(pid, {:depositar, 500})
# "💰 [Manual] Depositado 500. Novo saldo: 500"

# 3. Ver Saldo (Difícil! Preciso enviar meu PID e esperar resposta)
send(pid, {:ver_saldo, self()})

# Agora preciso caçar a resposta na minha caixa de correio
flush()
# {:resposta_saldo, 500}
```

**Análise do Professor:**
Funciona? Sim. Mas é frágil.

- E se o processo morrer? O cliente fica esperando pra sempre.
- E se eu quiser esperar a resposta (Síncrono)? Teria que escrever um `receive` no cliente.
- É muito código repetitivo ("Boilerplate").

---

### 2. Etapa B: O Padrão Ouro (`GenServer`)

Na Etapa A, construímos um servidor "na unha". Funciona? Sim. É seguro para produção? **Não.**
Como engenheiros, analisamos os riscos daquela abordagem manual:

1. **Boilerplate:** Você teve que escrever a recursão (`loop_do_servidor`) manualmente. Se esquecer uma linha, o servidor morre.
2. **Timeouts:** Se você mandar uma mensagem e o servidor não responder, seu cliente fica travado para sempre.
3. **Padronização:** Se cada programador escrever seu próprio loop `receive`, o código do projeto vira uma bagunça imprevisível.
Para resolver isso, a OTP (Open Telecom Platform) nos dá o **GenServer** (Generic Server). Ele abstrai a recursão, o tratamento de erros e o ciclo de vida do processo.

**2.1. Teoria: A Fronteira do Processo (Client vs Server)**

A maior confusão de quem começa com GenServer é entender **onde** o código está rodando. O módulo do GenServer é dividido em duas partes que rodam em processos diferentes.
Imagine um **Restaurante**:

1. **Client API (O Garçom):** É a interface pública. Você chama essas funções. Elas rodam no **seu** processo (no terminal ou na requisição HTTP). O trabalho delas é apenas anotar o pedido e gritar para a cozinha.
2. **Server Callbacks (A Cozinha):** É a implementação interna. Essas funções rodam dentro do **processo do GenServer**. Elas recebem o pedido, cozinham (processam a lógica/estado) e devolvem o prato.

**2.2. Os Protocolos de Comunicação (`Call` vs `Cast`)**

Em sistemas distribuídos, a forma como enviamos mensagens define a confiabilidade do sistema. O GenServer padroniza isso em dois verbos:

| Tipo | O Verbo | Analogia do Mundo Real | Comportamento Técnico |
| --- | --- | --- | --- |
| Síncrono | **Call** | Ligar para a Pizzaria | Você fica esperando a resposta. Se não atender em 5s (timeout), gera erro. **Garante consistência.** |
| Assíncrono | **Cast** | Mandar um e-mail | Você envia e não espera retorno. Pode ser lido agora ou mais tarde. **Garante performance (“fire and forget”).** |

**3. Laboratório Prático: O Banco Profissional**

Agora vamos refazer nossa conta bancária, mas usando a estrutura que você verá em empresas reais como Discord ou WhatsApp.
Desafio de Engenharia:
Implementar o módulo ContaBancaria onde:
• O **Estado** (Saldo) é gerenciado automaticamente pelo GenServer.
• **Depósitos** usam `cast` (pois o banco aceita seu dinheiro na hora, sem bloquear).
• **Saques** e **Consultas** usam `call` (pois você precisa da resposta imediata).

#### Passo 2: A Implementação Profissional

Vamos criar o módulo `ContaBancaria` (a versão final) dentro do projeto.

Crie o arquivo `lib/conta_bancaria.ex`.
*Note como `loop(novo_saldo)` vira `{:noreply, novo_saldo}`.*

```elixir
defmodule ContaBancaria do
  # 1. Injeta o comportamento de GenServer (o loop invisível)
  use GenServer

  # ===================================================================
  # API DO CLIENTE (Roda no processo do usuário)
  # ===================================================================

  @doc "Inicia a conta. O argumento é o saldo inicial (default 0)."
  def abrir_conta(saldo_inicial \\ 0) do
    # __MODULE__ é um atalho para 'ContaBancaria'.
    GenServer.start_link(__MODULE__, saldo_inicial)
  end

  def ver_saldo(pid) do
    # Call = Síncrono. O GenServer cuida de enviar, esperar e retornar o valor.
    GenServer.call(pid, :mostrar_saldo)
  end

  def depositar(pid, valor) do
    # Cast = Assíncrono. Só avisa.
    GenServer.cast(pid, {:depositar, valor})
  end

  def sacar(pid, valor) do
    # Call = Síncrono. Preciso saber se deu certo.
    GenServer.call(pid, {:sacar, valor})
  end

  # ===================================================================
  # CALLBACKS DO SERVIDOR (A Lógica Interna)
  # ===================================================================

  # 2. init/1: Define o estado inicial da primeira volta do loop.
  @impl true
  def init(saldo_inicial) do
    {:ok, saldo_inicial}
  end

  # 3. handle_call: Responde ao cliente E decide o estado futuro.
  @impl true
  def handle_call(:mostrar_saldo, _from, saldo_atual) do
    # {:reply, O_QUE_RESPONDE, O_ESTADO_FUTURO}
    {:reply, saldo_atual, saldo_atual}
  end

  @impl true
  def handle_call({:sacar, valor}, _from, saldo_atual) do
    if saldo_atual >= valor do
      novo_saldo = saldo_atual - valor
      {:reply, {:ok, novo_saldo}, novo_saldo}
    else
      # Responde erro e MANTÉM o estado antigo
      {:reply, {:error, "Saldo Insuficiente"}, saldo_atual}
    end
  end

  # 4. handle_cast: Não responde nada, apenas atualiza o estado futuro.
  @impl true
  def handle_cast({:depositar, valor}, saldo_atual) do
    novo_saldo = saldo_atual + valor
    IO.puts("🏦 [GenServer] Depósito de R$ #{valor} recebido.")
    
    # {:noreply, novo_saldo} é a versão chique de "loop_do_servidor(novo_saldo)"
    {:noreply, novo_saldo}
  end
end
```

#### Nota de Engenharia: O que é esse `@impl true`?

Você deve ter notado que, antes das funções `handle_call` e `handle_cast`, colocamos o atributo `@impl true`.

Se você vem do **Java** ou **C#**, isso é exatamente equivalente ao **`@Override`**.

#### Para que serve?

O `use GenServer` define um **Contrato** (chamado de *Behaviour* em Elixir). Ele diz: *"Quem usar este módulo DEVE ou PODE implementar funções chamadas `init`, `handle_call`, `handle_cast`, etc."*

O `@impl true` serve dois propósitos:

1. **Legibilidade (Para Humanos):** Ele avisa ao programador que está lendo: *"Ei, essa função `handle_cast` não foi inventada por mim. Ela faz parte das regras do GenServer."*
2. **Segurança (Para o Compilador):** Este é o mais importante. Ele age como um corretor ortográfico.

**Exemplo do "Salva-Vidas":**
Imagine que você digitou errado o nome da função (escreveu `handle_cat` em vez de `handle_cast`).

- **SEM `@impl`:** O Elixir vai achar que você criou uma função nova chamada `handle_cat` propositalmente. O código compila, mas seu servidor não funciona (as mensagens são ignoradas). Você perde horas debugando.
- **COM `@impl`:** O compilador emite um **aviso** (*warning*) apontando o erro na hora da compilação:
    
    ```text
    warning: got "@impl true" for function handle_cat/2 but no behaviour specifies such callback.
    ```
    
    Não é um erro que impede a compilação, mas é um aviso impossível de ignorar — e ferramentas como `mix compile --warnings-as-errors` (comum em CI) o transformam em erro.

**Regra de Ouro:** Sempre use `@impl true` nos callbacks (`init`, `handle_call`, `handle_cast`, `terminate`). É o cinto de segurança do seu código.

#### Testando a Versão GenServer

Volte ao terminal (`recompile()`). Veja como a experiência de uso (Developer Experience) é superior.

```elixir
# 1. Abrir conta
{:ok, pid} = ContaBancaria.abrir_conta(1000)

# 2. Testar Assincronismo (Cast)
ContaBancaria.depositar(pid, 500)
# :ok (Retorno imediato)
# [GenServer] Depósito de R$ 500 recebido. (Log assíncrono)

# 3. Testar Sincronismo (Call) - Sem precisar de flush()!
ContaBancaria.ver_saldo(pid)
# 1500

# 4. Testar Lógica de Negócio
ContaBancaria.sacar(pid, 2000)
# {:error, "Saldo Insuficiente"}
```

---

### 3. Reflexão de Engenharia

Por que migramos do manual para o GenServer?

1. **Abstração do Loop:** Não precisamos escrever a recursão manualmente. O retorno `{:noreply, novo_estado}` cuida disso.
2. **Abstração da Comunicação:** Não precisamos lidar com PIDs de resposta no `handle_call`. O GenServer sabe quem chamou e devolve a resposta.
3. **Tratamento de Erros:** O GenServer **não** engole exceções: se um callback falhar, o processo morre. Mas, como ele é um processo isolado, a falha não contamina os demais — e, como veremos no próximo módulo, um **Supervisor** pode reiniciá-lo automaticamente. Essa é a filosofia *"let it crash"*.

---

**Professor:** Agora temos um sistema funcional e limpo.
Mas, como engenheiros, devemos ser pessimistas. **E se houver um bug no código?** E se dividirmos por zero? O processo vai morrer. O dinheiro vai sumir.

## Módulo 4: Supervisores e Tolerância a Falhas

**Objetivo:** Transformar nosso processo isolado em uma **Árvore de Supervisão** (Supervision Tree), criando um sistema que se cura sozinho (Self-healing).

### 1. Teoria: O Supervisor

Um Supervisor é um processo especial. A única função dele é vigiar outros processos (chamados de *children* ou filhos).

Se um filho morre (crash), o Supervisor percebe o sinal de saída (`EXIT`) e aplica uma estratégia de reinicialização.

#### Estratégias Comuns:

- **:one_for_one (Um por Um):** Se o processo A morrer, reinicie apenas o A. (Mais comum).
- **:one_for_all (Um por Todos):** Se o processo A morrer, mate o B e o C, e reinicie todos. (Usado quando os processos dependem estritamente um do outro).

---

### 2. Laboratório Prático: Preparando o Crash

Para testar a resiliência, precisamos de uma forma de "quebrar" nossa conta bancária propositalmente.

#### Passo 1: Adicionando o Botão de Autodestruição

Edite o arquivo `lib/conta_bancaria.ex`. Vamos adicionar uma função que causa um erro fatal (divisão por zero ou `raise`).

```elixir
defmodule ContaBancaria do
  use GenServer

  # ... (Mantenha o código anterior de start_link, init, etc) ...

  # Adicione esta função na API do Cliente
  def causar_bug(pid) do
    GenServer.cast(pid, :bug_fatal)
  end

  # Adicione este callback lá no final
  @impl true
  def handle_cast(:bug_fatal, _saldo) do
    IO.puts("💣 BOOM! Ocorreu um erro inesperado...")
    # raise gera uma exceção que mata o processo na hora
    raise "Erro Fatal Simulada pelo Desenvolvedor"
  end
  
  # ... (Mantenha os outros handle_cast/handle_call) ...
end
```

---

#### Passo 2: Transformando o Projeto em uma Aplicação

Quando criamos o projeto com `mix new lab_concorrencia`, ele veio "pelado". Precisamos dizer ao Mix que este projeto tem um ponto de entrada (uma Application Callback).

1. Crie o arquivo `lib/lab_concorrencia/application.ex`:

```elixir
defmodule LabConcorrencia.Application do
  # Transforma este módulo em uma Application (ponto de partida do sistema)
  use Application

  @impl true
  def start(_type, _args) do
    # Lista de filhos que queremos que iniciem junto com o App
    children = [
      # Aqui dizemos: "Supervisor, inicie a ContaBancaria com saldo 1000"
      {ContaBancaria, 1000}
    ]

    # Opções:
    # strategy: :one_for_one -> Se a conta morrer, reinicia ela.
    # name: Nome do Supervisor para referência.
    opts = [strategy: :one_for_one, name: LabConcorrencia.Supervisor]

    IO.puts("🛡️ Supervisor Iniciado. Vigiando processos...")
    Supervisor.start_link(children, opts)
  end
end
```

1. **IMPORTANTE:** Precisamos avisar o Mix para carregar esse arquivo.
Abra o arquivo `mix.exs` na raiz, procure a função `application` e altere para:

```elixir
  def application do
    [
      extra_applications: [:logger],
      # Adicione esta linha: 'mod' define o módulo de entrada da aplicação
      mod: {LabConcorrencia.Application, []} 
    ]
  end
```

---

#### Passo 3: O Teste de Imortalidade (e a Correção do Contrato)

Agora que configuramos nosso Supervisor no `Application.ex`, a teoria diz que ele deve iniciar a conta automaticamente. Vamos testar?

**1. O Erro Esperado (A Falha de Contrato)**
Reinicie o terminal (`iex -S mix`).
Ao fazer isso, você provavelmente será recebido por uma mensagem de erro assustadora e o IEx fechará ou mostrará um crash log gigante.

```bash
* (Mix) Could not start application lab_concorrencia: ... ** (EXIT) 
an exception was raised: ** (UndefinedFunctionError) 
function ContaBancaria.start_link/1 is undefined or private
```

**O Diagnóstico de Engenharia:**
Por que falhou?
Quando dissemos ao Supervisor para cuidar do filho `{ContaBancaria, 1000}`, o Supervisor assumiu o comportamento padrão da OTP: ele tentou procurar e executar uma função chamada **obrigatoriamente** de `start_link`.

Mas espere! No passo anterior, nós demos o nome criativo de `abrir_conta`.

- **Você:** "Quero abrir uma conta."
- **Supervisor:** "Só sei executar `start_link`. Não encontrei, então vou crashar o sistema."

**2. A Solução (Obedecendo a Convenção)**
Vamos renomear nossa função de inicialização para seguir o padrão da indústria.

Edite o arquivo `lib/conta_bancaria.ex`:

```elixir
defmodule ContaBancaria do
  use GenServer

  # --- API DO CLIENTE ---

  # ANTES: def abrir_conta(saldo_inicial \\ 0) do
  # DEPOIS: Renomeamos para start_link para o Supervisor encontrar
  def start_link(saldo_inicial \\ 0) do
    # Aproveitamos para adicionar name: __MODULE__ (já prepara para o futuro)
    GenServer.start_link(__MODULE__, saldo_inicial, name: __MODULE__)
  end

  # ... mantenha o resto igual ...
end
```

**3. O Teste de Imortalidade (Agora funciona!)**
Agora sim. Reinicie o terminal (`iex -S mix`).
Você deve ver a mensagem: `"🛡️ Supervisor Iniciado..."`.

Agora vamos provar que o sistema se cura sozinho.

**A. Encontrando a Vítima:**
Como o Supervisor iniciou a conta, precisamos descobrir qual PID ele gerou.

```elixir
# O Supervisor nos diz quem são seus filhos.
[{_, pid_da_conta, _, _}] = Supervisor.which_children(LabConcorrencia.Supervisor)

# Verifique se pegou certo:
IO.inspect(pid_da_conta)
# Ex: #PID<0.155.0>
```

**B. Verificação de Saúde:**
Garanta que a conta está funcionando:

```elixir
ContaBancaria.ver_saldo(pid_da_conta)
# 1000 (O valor inicial configurado no Application.ex)
```

**C. O GRANDE MOMENTO: A Sabotagem:**
Vamos invocar a função `causar_bug` que criamos. Prepare-se para o erro vermelho (que agora é sinal de sucesso).

```elixir
ContaBancaria.causar_bug(pid_da_conta)
```

**O que aconteceu?**

1. O processo morreu (Crash).
2. O Supervisor percebeu o sinal de `EXIT`.
3. Imediatamente, ele iniciou um **novo processo** para substituir o morto.

**D. A Prova da Ressurreição:**
Tente falar com o PID antigo:

```elixir
Process.alive?(pid_da_conta)
# false (Morto e enterrado)
```

Agora, pergunte ao Supervisor quem é o filho dele **agora**:

```elixir
Supervisor.which_children(LabConcorrencia.Supervisor)
# Saída: [{ContaBancaria, #PID<0.160.0>, ...}]
```

**Olhe o PID!** É um número novo (ex: 160).
O Supervisor limpou a bagunça e subiu uma nova instância. Seu banco digital continua no ar, pronto para receber novos clientes.

---

### 3. Reflexão de Arquitetura (O "Pulo do Gato")

Aluno, observe um detalhe crítico:

Pegue o PID novo e veja o saldo:

```elixir
[{_, novo_pid, _, _}] = Supervisor.which_children(LabConcorrencia.Supervisor)
	ContaBancaria.ver_saldo(novo_pid)
# 1000
```

Se você tivesse depositado 500 reais antes do crash, o saldo seria 1500. Após o crash, ele voltou para 1000 (estado inicial).

**Por quê?**
Porque processos guardam estado em **Memória RAM**. Se o processo morre, a memória vai junto. O Supervisor reinicia o processo "do zero" (clean state).

**Engenharia de Sistemas Robustos:**

- **O Supervisor garante disponibilidade:** O serviço do banco continua no ar (o processo existe).
- **O Banco de Dados garante persistência:** Para não perder o saldo, o `GenServer` deveria salvar o valor num banco (Postgres/Redis) a cada depósito e ler do banco no `init`.

---

**Parabéns!** Você construiu um sistema que:

1. Roda concorrentemente (Processos).
2. Gerencia estado (GenServer).
3. Se recupera de falhas sozinho (Supervisors).

Isso é a base de sistemas como WhatsApp e Discord.

Em um sistema real com milhares de processos, nós não ficamos anotando PIDs (`#PID<0.155.0>`) num caderninho. PIDs são efêmeros; eles mudam toda vez que o processo morre e renasce. Precisamos de um **Nome Fixo** (DNS interno). Vamos resolver isso ?

---

## Processos Nomeados (Named Processes)

**O Problema:** Quando o Supervisor reinicia a `ContaBancaria` após um crash, o PID muda. Se o seu código dependia do PID antigo, ele quebra.

**A Solução:** Registramos o processo com um nome (geralmente um Átomo). Assim, mandamos mensagens para o **Nome**, e o Elixir descobre quem é o PID atual daquele nome.

### Passo 1: Atualizando o Código

Abra `lib/conta_bancaria.ex`. Vamos fazer apenas duas pequenas alterações:

1. No `start_link`, vamos adicionar a opção `name`.
2. Na API do cliente, vamos permitir chamar pelo nome.

```elixir
defmodule ContaBancaria do
  use GenServer

  # --- API DO CLIENTE ---

  # Alteração 1: name: __MODULE__
  # Isso registra o processo com o nome "ContaBancaria" (o átomo do módulo).
  def start_link(saldo_inicial \\ 0) do
    GenServer.start_link(__MODULE__, saldo_inicial, name: __MODULE__)
  end

  # Alteração 2: Default para o nome
  # Se o usuário não passar o PID, assumimos que ele quer falar com a conta padrão.
  def ver_saldo(pid_ou_nome \\ __MODULE__) do
    GenServer.call(pid_ou_nome, :mostrar_saldo)
  end

  def depositar(pid_ou_nome \\ __MODULE__, valor) do
    GenServer.cast(pid_ou_nome, {:depositar, valor})
  end

  def sacar(pid_ou_nome \\ __MODULE__, valor) do
    GenServer.call(pid_ou_nome, {:sacar, valor})
  end

  def causar_bug(pid_ou_nome \\ __MODULE__) do
    GenServer.cast(pid_ou_nome, :bug_fatal)
  end

  # ... (O resto do código e callbacks continuam IGUAIS) ...
end
```

### Passo 2: O Teste Definitivo

Reinicie seu terminal (`iex -S mix`) e veja a mágica da **Transparência de Localização**.

1. **Uso direto (Sem PID):**Elixir
    
    ```elixir
    # Não preciso mais buscar PID. Chamo direto pelo nome do módulo.
    ContaBancaria.ver_saldo()
    # 1000
    ```
    
2. **Crash e Recuperação Transparente:**Elixir
    
    ```elixir
    # Vamos matar o processo
    ContaBancaria.causar_bug()
    # [Log de erro gigante...]
    
    # IMEDIATAMENTE tente ver o saldo de novo:
    ContaBancaria.ver_saldo()
    # 1000
    ```
    

**O que aconteceu?**

1. O processo antigo morreu.
2. O Supervisor criou um novo PID.
3. O novo processo, ao nascer, gritou: *"Agora EU sou o `ContaBancaria`!"*
4. Quando você chamou `ver_saldo()`, o Elixir mandou a mensagem para o novo PID automaticamente.

Isso é **Alta Disponibilidade**. O cliente nem percebeu que o servidor caiu e voltou.

## Módulo Intermediário: Especialistas (`Agent` e `Task`)

Em alguns casos, ao invés de usar **GenServer** (que faz tudo), podemos usar seus "irmãos menores". Eles são processos especializados:

1. **`Agent`**: Especialista em **Guardar Estado** (Simples).
2. **`Task`**: Especialista em **Processamento Paralelo** (Cálculo).

---

### 1. O `Agent` (O Cofre de Estado)

Imagine que você só quer guardar um valor (como uma configuração global ou um contador) e ler/atualizar isso de forma segura entre vários processos.
Criar um `GenServer` inteiro com `handle_call` e `handle_cast` só para isso é "matar formiga com bazuca".

O **Agent** abstrai tudo isso.

#### Prática: O Cofre do Banco

Vamos criar um cofre que guarda as reservas totais do banco.
Crie o arquivo `lib/cofre.ex`:

```elixir
defmodule Cofre do
  # Inicia o Agent.
  # A função anônima (fn -> ...) define o estado inicial.
  def start_link(valor_inicial) do
    Agent.start_link(fn -> valor_inicial end, name: __MODULE__)
  end

  # Leitura Síncrona
  # O Agent.get pega o estado atual e retorna o que você quiser.
  # Aqui retornamos o próprio saldo (&(&1) é um atalho para fn x -> x end).
  def ver_total do
    Agent.get(__MODULE__, fn saldo -> saldo end)
  end

  # Atualização Síncrona
  # O Agent.update altera o estado.
  def depositar(valor) do
    Agent.update(__MODULE__, fn saldo -> saldo + valor end)
  end
end
```

#### Teste no Terminal (`iex -S mix`)

Olhe como é simples (sem callbacks complexos):

```elixir
# 1. Iniciar com 50 mil reais
Cofre.start_link(50_000)

# 2. Consultar
Cofre.ver_total()
# 50000

# 3. Atualizar
Cofre.depositar(10_000)
Cofre.ver_total()
# 60000
```

**Conceito de Engenharia:** O `Agent` garante **Atomicidade**. Se 100 processos tentarem atualizar o cofre ao mesmo tempo, o Agent coloca todos em fila e atende um por um. Não há risco de corromper o dado.

---

### 2. A `Task` (O Trabalhador Assíncrono)

E se precisarmos fazer algo demorado (ex: enviar e-mail de confirmação ou gerar PDF) e não quisermos travar o nosso Banco?

Lembre-se: O Elixir é síncrono por padrão. Se você colocar um `Process.sleep(3000)` no meio do código, aquele processo fica parado esperando.
A **Task** serve para jogar esse trabalho para um processo descartável secundário.

#### Prática: O Notificador

Crie o arquivo `lib/notificador.ex`:

```elixir
defmodule Notificador do
  def enviar_email(cliente) do
    IO.puts("📧 Iniciando envio de email para #{cliente}...")
    
    # Simula uma conexão lenta com servidor de email (3 segundos)
    Process.sleep(3000)
    
    IO.puts("✅ Email enviado com sucesso para #{cliente}!")
  end
end
```

#### Teste Comparativo (IEx)

**Cenário A: O jeito travado (Sem Task)**
Rode isso e veja seu terminal congelar:

```elixir
Notificador.enviar_email("Joao")
# ... (você não consegue digitar nada por 3 segundos) ...
# "✅ Email enviado..."
```

*Em um servidor web, isso seria desastroso. O usuário ficaria vendo a ampulheta girar.*

**Cenário B: O jeito fluido (Com Task)**
Agora vamos usar `Task.start/1`.

```elixir
Task.start(fn -> Notificador.enviar_email("Maria") end)
# {:ok, #PID<0.165.0>}
# O terminal libera IMEDIATAMENTE! Você pode continuar digitando.
```

*Três segundos depois, a mensagem "✅ Email enviado..." aparece magicamente no meio do que você estiver fazendo.*

**3. Resumo da Arquitetura: Quando usar o quê?**

| Ferramenta | Analogia | Uso Principal | Complexidade |
| --- | --- | --- | --- |
| **spawn** | Célula-tronco | Baixo nível, criar processos brutos (evitar em produção). | Alta (Manual) |
| **Task** | Freelancer | Tarefas pontuais, paralelas e descartáveis (cálculos, emails). | Baixa |
| **Agent** | Armário / Cofre | Guardar e compartilhar um estado simples. | Baixa |
| **GenServer** | Gerente | Lógica de negócio complexa, estado robusto e longa vida. | Média |

---

## Revisão Geral do Módulo

Parabéns, caro aluno. Você completou a jornada fundamental da plataforma BEAM.
Vamos recapitular o modelo mental que você construiu, pois é isso que importa na sua carreira.

### 1. O Paradigma Funcional

- **Imutabilidade:** Dados nunca mudam. `lista = [1]` cria uma nova lista, não altera a antiga. Isso elimina efeitos colaterais.
- **Pipeline (`|>`):** Programar é transformar dados. `DadoBruto |> Tratar |> Salvar`.
- **Modules vs Classes:** Módulos são sacos de funções. Não guardam estado.
- **Pattern Matching:** A forma como tomamos decisões (`def handle(10), do: ...`) e extraímos dados, substituindo `if/else` complexos.

### 2. O Modelo de Concorrência (Atores)

- **Processos Leves:** A unidade básica não é a Thread do SO, mas o Processo da BEAM. Isolados, leves (KB) e massivos (milhões).
- **Mensageria:** Processos não compartilham memória. Eles trocam cópias de dados via `send` e `receive`.
- **Assíncrono (`cast`) vs Síncrono (`call`):** A distinção vital em sistemas distribuídos.

### 3. As Ferramentas de Trabalho

Não reinventamos a roda. Usamos abstrações:

- **`Task`:** Para rodar scripts paralelos ("Faça isso e me avise").
- **`Agent`:** Para guardar estado simples ("Segure este valor").
- **`GenServer`:** O servidor genérico. A peça fundamental para lógica de negócios com estado e concorrência.

### 4. Tolerância a Falhas (The Erlang Way)

- **Let it Crash:** Não trate erros impossíveis. Deixe o processo morrer.
- **Supervisors:** O "Anjo da Guarda" que vigia processos e os reinicia para um estado limpo e conhecido.
- **Árvore de Supervisão:** A arquitetura hierárquica do sistema. Se um worker falha, o supervisor local resolve. Se o supervisor falha, o supervisor dele resolve.

---

### Próximos Passos na sua Carreira

Agora que você domina a base, para onde ir?

1. **Phoenix Framework:** É o "Django/Rails" do Elixir, mas capaz de aguentar milhões de conexões WebSocket simultâneas (graças aos Processos que você acabou de aprender!).
2. **Ecto:** A biblioteca de banco de dados. Você aprendeu que processos perdem estado ao reiniciar. O Ecto é quem salva isso no Postgres.
3. **Broadway/GenStage:** Para processamento de dados em massa (Data Pipelines) com back-pressure.

Foi uma honra guiá-lo nesta jornada. Você agora tem o mindset de um Engenheiro de Software Concorrente. **Sucesso!**

### Veja mais

[Excelente palestra do Fabio Akita de 2015](https://www.youtube.com/watch?v=GeGXXfNvdSA)