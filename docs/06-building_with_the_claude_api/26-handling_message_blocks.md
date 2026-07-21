# Manipulação de blocos de mensagens

```
notebooks\14-tool_functions.ipynb
```

## 1. Ideia central

A aula explica como lidar com **mensagens multi-block** ao usar ferramentas na API do Claude.

Antes, as respostas podiam ser tratadas como texto simples. Com o uso de tools, Claude pode retornar uma mensagem do assistant composta por múltiplos blocos, por exemplo:

1. Um bloco de texto explicando o que ele fará.
2. Um bloco `tool_use` indicando qual ferramenta deve ser chamada e com quais argumentos.

A ideia central é: **quando Claude usa ferramentas, a resposta não é apenas texto; ela vira uma lista estruturada de blocos que precisa ser preservada corretamente no histórico da conversa.**

---

## 2. Principais conceitos

### Tool-enabled API calls

Para permitir que Claude use ferramentas, é necessário passar um parâmetro `tools` na chamada da API.

Exemplo da aula:

```python
messages = []
messages.append({
    "role": "user",
    "content": "What is the exact time, formatted as HH:MM:SS?"
})

response = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=messages,
    tools=[get_current_datetime_schema],
)
```

Nesse exemplo:

* O usuário pergunta o horário exato.
* A chamada inclui uma lista de schemas de ferramentas disponíveis.
* Claude pode decidir chamar a ferramenta `get_current_datetime`.

O parâmetro `tools` recebe uma lista de **JSON Schemas** que descrevem as funções disponíveis para Claude.

---

### Mensagens multi-block

Quando Claude decide usar uma ferramenta, ele pode retornar uma mensagem do assistant cujo campo `content` é uma lista de blocos.

Essa lista pode conter diferentes tipos de conteúdo.

No exemplo visual da aula, a mensagem do assistant contém:

```text
Text Block
```

Com uma resposta como:

```text
I can help you find out the current time. Let me find that information for you
```

E também contém:

```text
ToolUse Block
```

Com a chamada estruturada da ferramenta:

```python
ToolUseBlock(
    id='toolu_...',
    input={'date_format': '%H:%M:%S'},
    name='get_current_datetime',
    type='tool_use'
)
```

Isso mostra que Claude não executa diretamente a função. Ele informa ao seu código **qual ferramenta chamar** e **com quais parâmetros**.

---

### Text Block

O **Text Block** é a parte legível para humanos.

Ele geralmente explica o que Claude está fazendo.

Exemplo:

```text
I can help you find out the current time. Let me find that information for you
```

Esse bloco é útil para comunicação natural com o usuário, mas não é a chamada da ferramenta em si.

---

### ToolUse Block

O **ToolUse Block** contém as instruções estruturadas para o código executar a ferramenta.

Ele inclui:

* `id`: identificador da chamada da ferramenta.
* `name`: nome da função que deve ser chamada.
* `input`: argumentos da função.
* `type`: tipo do bloco, normalmente `tool_use`.

Exemplo:

```python
ToolUseBlock(
    id='toolu_010tKMDNkU1FBi73NP2FbF8w',
    input={'date_format': '%H:%M:%S'},
    name='get_current_datetime',
    type='tool_use'
)
```

Esse bloco indica que o código deve chamar a ferramenta `get_current_datetime` usando o formato de data `HH:MM:SS`.

---

### Histórico de conversa

Um ponto essencial da aula é que **Claude não armazena automaticamente o histórico da conversa**.

A aplicação é responsável por manter o histórico em uma lista de mensagens.

Quando há mensagens multi-block, é fundamental preservar a estrutura completa do conteúdo.

Exemplo correto:

```python
messages.append({
    "role": "assistant",
    "content": response.content
})
```

Isso preserva:

* O bloco de texto.
* O bloco `tool_use`.
* A estrutura completa necessária para a próxima chamada da API.

Erro comum seria salvar apenas o texto e descartar o bloco de ferramenta. Isso quebra o fluxo, porque Claude perde o contexto da chamada da ferramenta.

---

### Fluxo completo de uso de ferramentas

O processo completo segue esta sequência:

1. Enviar a mensagem do usuário para Claude junto com o schema da ferramenta.
2. Receber uma mensagem do assistant com blocos de texto e `tool_use`.
3. Extrair as informações do bloco `tool_use`.
4. Executar a função real no seu código.
5. Enviar o resultado da ferramenta de volta para Claude junto com o histórico completo.
6. Receber a resposta final de Claude.

Esse fluxo é importante porque Claude decide **quando** e **como** usar a ferramenta, mas quem realmente executa a função é a aplicação.

---

### Atualização de helper functions

A aula também mostra que funções auxiliares simples podem precisar ser atualizadas.

Exemplo de helpers antigos:

```python
def add_user_message(messages, text):
    user_message = {"role": "user", "content": text}
    messages.append(user_message)

def add_assistant_message(messages, text):
    assistant_message = {"role": "assistant", "content": text}
    messages.append(assistant_message)
```

Essas funções funcionam bem para mensagens de texto simples, mas podem ser insuficientes para mensagens multi-block.

O problema está especialmente em `add_assistant_message`, porque agora o conteúdo do assistant pode não ser apenas uma string. Pode ser uma lista com blocos de texto e blocos de ferramenta.

Uma versão mais adequada deveria aceitar conteúdo estruturado:

```python
def add_assistant_message(messages, content):
    assistant_message = {
        "role": "assistant",
        "content": content
    }
    messages.append(assistant_message)
```

Assim, `content` pode ser tanto texto quanto uma lista de blocos.

---

## 3. Pontos importantes para certificação

* Para habilitar tools na API, é preciso passar o parâmetro `tools`.
* O parâmetro `tools` recebe uma lista de JSON Schemas das ferramentas disponíveis.
* Quando Claude decide usar uma ferramenta, ele retorna uma mensagem multi-block.
* Uma mensagem multi-block pode conter:

  * `Text Block`
  * `ToolUse Block`
* O `Text Block` explica em linguagem natural o que Claude está fazendo.
* O `ToolUse Block` informa qual ferramenta chamar e quais argumentos usar.
* O `ToolUse Block` normalmente contém:

  * `id`
  * `name`
  * `input`
  * `type`
* O tipo do bloco de uso de ferramenta é `tool_use`.
* Claude não executa a função diretamente; sua aplicação executa a função.
* Claude não armazena o histórico automaticamente.
* A aplicação precisa preservar o histórico manualmente.
* Ao salvar uma resposta do assistant no histórico, preserve `response.content` inteiro.
* Não salve apenas o texto quando houver uso de ferramentas.
* O fluxo correto é:

  * usuário pergunta;
  * Claude retorna `tool_use`;
  * aplicação executa a ferramenta;
  * aplicação envia o resultado de volta;
  * Claude gera a resposta final.
* Helper functions antigas podem precisar ser adaptadas para aceitar conteúdo estruturado, não apenas strings.

---

## 4. Termos-chave

### Multi-block message

Mensagem cujo conteúdo é uma lista de blocos, podendo incluir texto, chamadas de ferramenta e outros tipos de conteúdo.

### Block

Parte individual de uma mensagem. Também pode ser chamado de “part”.

### Text Block

Bloco textual usado para comunicação legível por humanos.

### ToolUse Block

Bloco estruturado que indica que Claude quer usar uma ferramenta específica.

### `tool_use`

Tipo de bloco usado para representar uma chamada de ferramenta solicitada por Claude.

### Tool schema

JSON Schema que descreve uma ferramenta disponível, incluindo nome, descrição e parâmetros esperados.

### `tools`

Parâmetro da API usado para informar a Claude quais ferramentas estão disponíveis.

### `response.content`

Conteúdo completo retornado por Claude, que pode conter múltiplos blocos.

### Conversation history

Histórico da conversa mantido manualmente pela aplicação.

### `id`

Identificador da chamada de ferramenta. Serve para rastrear qual chamada está sendo respondida.

### `name`

Nome da ferramenta ou função que deve ser chamada.

### `input`

Argumentos que devem ser passados para a ferramenta.

### Helper functions

Funções auxiliares usadas para adicionar mensagens ao histórico ou simplificar chamadas à API.

---

## 5. Boas práticas

* Sempre passe os schemas das ferramentas no parâmetro `tools`.
* Trate `response.content` como conteúdo estruturado, não apenas como string.
* Preserve todos os blocos da resposta do assistant no histórico.
* Ao adicionar uma resposta do assistant ao histórico, use:

```python
messages.append({
    "role": "assistant",
    "content": response.content
})
```

* Verifique se há blocos do tipo `tool_use` antes de tentar executar ferramentas.
* Use o campo `name` para decidir qual função chamar.
* Use o campo `input` para passar os argumentos corretos.
* Use o campo `id` para associar o resultado da ferramenta à chamada correta.
* Atualize helper functions para aceitar conteúdo multi-block.
* Não descarte o `ToolUse Block`.
* Depois de executar a ferramenta, envie o resultado de volta para Claude com o histórico completo.
* Mantenha o fluxo da conversa consistente para que Claude consiga gerar a resposta final corretamente.

---

## 6. Limitações, riscos e cuidados

* Assumir que toda resposta de Claude é texto simples pode quebrar aplicações com tools.
* Salvar apenas o texto do assistant remove informações importantes sobre a chamada da ferramenta.
* Se o `ToolUse Block` não for preservado, Claude pode perder o contexto da ferramenta usada.
* Claude não executa a ferramenta automaticamente; a aplicação precisa executar.
* A aplicação precisa validar qual ferramenta foi solicitada antes de chamar funções reais.
* É importante tratar corretamente os argumentos em `input`.
* Helper functions antigas podem esconder bugs se aceitarem apenas strings.
* O histórico manual precisa ser completo e fiel à estrutura original.
* Em aplicações reais, é importante lidar com múltiplos blocos e possivelmente múltiplas chamadas de ferramentas.
* Se o resultado da ferramenta não for enviado de volta corretamente, Claude não consegue concluir a resposta final com base no resultado real.

---

## 7. Resumo para revisão rápida

* Com tools, Claude pode retornar mensagens multi-block.
* Blocos também podem ser chamados de “parts”.
* Uma resposta pode conter texto e chamada de ferramenta.
* `Text Block` explica o que Claude está fazendo.
* `ToolUse Block` diz qual função chamar.
* O bloco `tool_use` contém `id`, `name`, `input` e `type`.
* Para habilitar ferramentas, use o parâmetro `tools`.
* `tools` recebe JSON Schemas das funções disponíveis.
* Claude não executa a função; sua aplicação executa.
* Claude não guarda histórico automaticamente.
* Preserve `response.content` completo no histórico.
* Não transforme tudo em string.
* Atualize helpers para aceitar conteúdo estruturado.
* O fluxo é: pedir → receber `tool_use` → executar ferramenta → devolver resultado → receber resposta final.

---

## 8. Perguntas simuladas

### Pergunta 1

O que muda nas respostas de Claude quando tools são habilitadas?

A. Claude sempre retorna apenas uma string simples.
B. Claude pode retornar mensagens com múltiplos blocos, incluindo texto e uso de ferramenta.
C. Claude deixa de responder em linguagem natural.
D. Claude executa automaticamente todas as funções no servidor do usuário.

**Resposta correta: B**

**Explicação:** Com tools, Claude pode retornar uma mensagem multi-block contendo um `Text Block` e um `ToolUse Block`.

---

### Pergunta 2

Qual parâmetro deve ser incluído na chamada da API para disponibilizar ferramentas a Claude?

A. `functions_enabled`
B. `plugins`
C. `tools`
D. `tool_results_only`

**Resposta correta: C**

**Explicação:** O parâmetro `tools` recebe uma lista de JSON Schemas das ferramentas disponíveis.

---

### Pergunta 3

O que o parâmetro `tools` recebe?

A. Uma lista de JSON Schemas que descrevem funções disponíveis.
B. Apenas strings com nomes de arquivos.
C. O resultado final das ferramentas.
D. O histórico completo da conversa.

**Resposta correta: A**

**Explicação:** Cada tool é descrita por um schema que informa a Claude como e quando a função pode ser chamada.

---

### Pergunta 4

Qual é a função do `Text Block` em uma mensagem multi-block?

A. Executar diretamente uma função Python.
B. Explicar em linguagem natural o que Claude está fazendo.
C. Armazenar o resultado da ferramenta obrigatoriamente.
D. Substituir o schema da ferramenta.

**Resposta correta: B**

**Explicação:** O `Text Block` é a parte textual da resposta, usada para comunicação legível por humanos.

---

### Pergunta 5

Qual é a função do `ToolUse Block`?

A. Informar à aplicação qual ferramenta chamar e com quais argumentos.
B. Mostrar apenas uma mensagem visual para o usuário.
C. Executar o código automaticamente dentro de Claude.
D. Apagar o histórico da conversa.

**Resposta correta: A**

**Explicação:** O `ToolUse Block` contém o nome da ferramenta, argumentos, ID e tipo da chamada.

---

### Pergunta 6

Quais campos aparecem tipicamente em um `ToolUse Block`?

A. `title`, `author`, `version`, `date`
B. `id`, `name`, `input`, `type`
C. `user`, `password`, `token`, `secret`
D. `schema`, `history`, `temperature`, `model`

**Resposta correta: B**

**Explicação:** O bloco de uso de ferramenta inclui identificador, nome da função, argumentos de entrada e tipo `tool_use`.

---

### Pergunta 7

No exemplo da aula, qual ferramenta Claude decide usar?

A. `send_email`
B. `get_current_datetime`
C. `search_database`
D. `summarize_pdf`

**Resposta correta: B**

**Explicação:** Claude usa a ferramenta `get_current_datetime` para obter o horário atual formatado.

---

### Pergunta 8

No exemplo, qual argumento é passado para a ferramenta de data e hora?

A. `timezone: "UTC"`
B. `language: "pt-BR"`
C. `date_format: "%H:%M:%S"`
D. `calendar: "gregorian"`

**Resposta correta: C**

**Explicação:** O bloco mostra `input={'date_format': '%H:%M:%S'}`, indicando formato de hora, minuto e segundo.

---

### Pergunta 9

Quem é responsável por executar a função real indicada pelo `ToolUse Block`?

A. A aplicação do desenvolvedor.
B. O navegador do usuário automaticamente.
C. O arquivo `CLAUDE.md`.
D. O próprio JSON Schema.

**Resposta correta: A**

**Explicação:** Claude solicita o uso da ferramenta, mas quem executa a função real é o código da aplicação.

---

### Pergunta 10

Como a resposta multi-block do assistant deve ser adicionada ao histórico da conversa?

A. Salvando apenas o texto visível ao usuário.
B. Convertendo todos os blocos para uma string única.
C. Preservando `response.content` completo.
D. Ignorando o bloco `tool_use`.

**Resposta correta: C**

**Explicação:** É essencial preservar todos os blocos, incluindo texto e chamada de ferramenta, para manter o contexto correto.

---

### Pergunta 11

Qual código representa a forma correta de preservar a resposta do assistant?

A.

```python
messages.append({
    "role": "assistant",
    "content": response.content
})
```

B.

```python
messages.append({
    "role": "assistant",
    "content": response.content[0].text
})
```

C.

```python
messages = []
```

D.

```python
messages.append({
    "role": "tool",
    "content": "ignored"
})
```

**Resposta correta: A**

**Explicação:** Usar `response.content` preserva a estrutura completa da mensagem multi-block.

---

### Pergunta 12

Por que helper functions antigas podem precisar ser atualizadas?

A. Porque agora o conteúdo do assistant pode ser uma estrutura multi-block, não apenas texto simples.
B. Porque Claude não aceita mais mensagens de usuário.
C. Porque JSON Schema foi removido.
D. Porque tools não usam mais parâmetros.

**Resposta correta: A**

**Explicação:** Funções que assumem que `content` é sempre uma string podem falhar ou perder dados ao lidar com `tool_use`.

---

### Pergunta 13

Qual é o fluxo correto de uso de ferramentas com Claude?

A. Executar ferramenta → criar schema → perguntar ao usuário → apagar histórico.
B. Enviar mensagem e schema → receber `tool_use` → executar função → enviar resultado → receber resposta final.
C. Enviar apenas texto → ignorar tools → finalizar conversa.
D. Carregar todas as ferramentas no histórico como strings.

**Resposta correta: B**

**Explicação:** Esse é o fluxo completo: Claude solicita a ferramenta, a aplicação executa, devolve o resultado e Claude responde com base nele.

---

### Pergunta 14

Qual é um erro comum ao lidar com mensagens multi-block?

A. Preservar todos os blocos no histórico.
B. Passar schemas no parâmetro `tools`.
C. Salvar apenas o texto e descartar o bloco `tool_use`.
D. Executar a função indicada pelo campo `name`.

**Resposta correta: C**

**Explicação:** Descartar o `ToolUse Block` quebra o contexto necessário para continuar corretamente a conversa com uso de ferramentas.
