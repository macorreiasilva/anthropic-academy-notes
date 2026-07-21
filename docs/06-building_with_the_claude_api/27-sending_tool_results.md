# Envio de resultados da ferramenta

```
notebooks\14-tool_functions.ipynb
```

## 1. Ideia central

A aula explica a etapa final do fluxo de uso de ferramentas com Claude: **executar a função solicitada por Claude e enviar o resultado de volta usando um bloco `tool_result`**.

Depois que Claude retorna um bloco `tool_use`, ele ainda não tem o resultado da ferramenta. A aplicação precisa:

1. Ler os argumentos no bloco `tool_use`.
2. Executar a função real no servidor/aplicação.
3. Criar uma nova mensagem de usuário com um bloco `tool_result`.
4. Enviar essa mensagem de volta para Claude junto com o histórico completo.
5. Receber a resposta final em linguagem natural.

---

## 2. Principais conceitos

### Executando a função da ferramenta

Quando Claude retorna um bloco `tool_use`, os parâmetros da ferramenta ficam disponíveis no campo `input`.

Exemplo da aula:

```python
response.content[1].input
```

Esse campo retorna um dicionário com os argumentos que Claude quer passar para a função.

No exemplo, o input era algo como:

```python
{
    "date_format": "%H:%M:%S"
}
```

Como a função `get_current_datetime` espera argumentos nomeados, é possível usar o operador `**` do Python para desempacotar o dicionário:

```python
get_current_datetime(**response.content[1].input)
```

Isso equivale a chamar:

```python
get_current_datetime(date_format="%H:%M:%S")
```

O resultado mostrado na imagem foi um horário no formato solicitado, por exemplo:

```text
14:41:05
```

Esse exemplo demonstra como transformar a solicitação estruturada de Claude em uma chamada real de função no código.

---

### Tool Result Block

Depois de executar a ferramenta, o resultado precisa ser enviado de volta para Claude usando um bloco do tipo:

```text
tool_result
```

Esse bloco fica dentro da lista `content` de uma nova mensagem com role `user`.

Exemplo:

```python
messages.append({
    "role": "user",
    "content": [{
        "type": "tool_result",
        "tool_use_id": response.content[1].id,
        "content": "15:04:22",
        "is_error": False
    }]
})
```

Esse bloco comunica a Claude o que aconteceu quando a ferramenta foi executada.

---

### Campos do `tool_result`

O bloco `tool_result` possui campos importantes:

### `type`

Indica o tipo do bloco.

Neste caso:

```text
tool_result
```

### `tool_use_id`

Deve corresponder exatamente ao `id` do bloco `tool_use` que originou aquela execução.

Esse campo é essencial para Claude saber qual resultado pertence a qual chamada de ferramenta.

### `content`

Contém a saída da função executada.

Importante: o resultado deve ser serializado como string.

Exemplo:

```text
"12:47:13"
```

### `is_error`

Indica se ocorreu erro na execução da ferramenta.

* `False`: a ferramenta executou com sucesso.
* `True`: ocorreu algum erro.

---

### Mensagem de usuário com resultado da ferramenta

Um ponto importante da aula é que o resultado da ferramenta é enviado como uma **mensagem de usuário**, não como mensagem do assistant.

Estrutura:

```python
{
    "role": "user",
    "content": [
        {
            "type": "tool_result",
            "tool_use_id": "...",
            "content": "...",
            "is_error": False
        }
    ]
}
```

Isso pode parecer estranho no início, mas faz sentido no fluxo da API: Claude pediu para a aplicação executar uma ferramenta, e agora a aplicação responde a Claude com o resultado.

---

### Múltiplas chamadas de ferramenta

Claude pode solicitar mais de uma ferramenta em uma única resposta.

Exemplo da aula:

Usuário pergunta:

```text
What's 10 + 10 and what's 30 + 30?
```

Claude pode retornar dois blocos `tool_use`:

```text
ToolUse One
Tool name: calculator
ID: ab3
Input: 10 + 10

ToolUse Two
Tool name: calculator
ID: po9
Input: 30 + 30
```

Depois, a aplicação pode enviar dois blocos `tool_result`:

```text
ToolResult One
ID: po9
Output: 60

ToolResult Two
ID: ab3
Output: 20
```

O ponto importante é que os resultados podem até ser enviados em ordem diferente, mas cada resultado precisa ter o `tool_use_id` correto.

Claude usa o ID para associar:

```text
po9 → 30 + 30 → 60
ab3 → 10 + 10 → 20
```

Portanto, **a correspondência pelo ID é mais importante do que a ordem dos blocos**.

---

### Histórico completo da conversa

A chamada seguinte para Claude precisa incluir o histórico completo:

1. Mensagem original do usuário.
2. Mensagem do assistant com o bloco `tool_use`.
3. Mensagem do usuário com o bloco `tool_result`.

Exemplo conceitual:

```text
User Message:
"What is the current time?"

Assistant Message:
Text Block + ToolUse Block

User Message:
ToolResult Block
```

Esse histórico completo permite que Claude entenda:

* O que o usuário pediu.
* Qual ferramenta Claude solicitou.
* Qual resultado a aplicação obteve.
* Como formular a resposta final ao usuário.

---

### Enviando a chamada final para Claude

Mesmo depois de enviar o resultado da ferramenta, a próxima chamada ainda deve incluir o schema da ferramenta:

```python
client.messages.create(
    model=model,
    max_tokens=1000,
    messages=messages,
    tools=[get_current_datetime_schema]
)
```

Isso é importante porque Claude precisa entender as referências às ferramentas presentes no histórico da conversa.

Mesmo que você não espere uma nova chamada de ferramenta, o schema ainda ajuda Claude a interpretar corretamente o fluxo anterior.

---

## 3. Pontos importantes para certificação

* Depois de receber um bloco `tool_use`, a aplicação deve executar a função real.
* Os argumentos da ferramenta ficam em:

```python
response.content[1].input
```

* Em Python, `**` pode ser usado para desempacotar os argumentos:

```python
get_current_datetime(**response.content[1].input)
```

* O resultado da ferramenta deve ser enviado de volta usando um bloco `tool_result`.
* O bloco `tool_result` fica dentro de uma mensagem com role `user`.
* O campo `tool_use_id` deve corresponder ao `id` do bloco `tool_use`.
* O campo `content` contém o resultado da ferramenta, serializado como string.
* O campo `is_error` indica se houve erro na execução.
* Claude pode solicitar múltiplas ferramentas em uma única resposta.
* Cada chamada de ferramenta tem um ID único.
* Em múltiplas chamadas, os resultados precisam ser associados pelo ID, não apenas pela ordem.
* A chamada de follow-up deve incluir o histórico completo.
* O histórico completo inclui:

  * mensagem original do usuário;
  * resposta do assistant com `tool_use`;
  * mensagem do usuário com `tool_result`.
* A chamada final ainda deve incluir o schema da ferramenta no parâmetro `tools`.
* Claude usa o resultado da ferramenta para produzir a resposta final em linguagem natural.

---

## 4. Termos-chave

### `tool_use`

Bloco retornado por Claude indicando que ele quer chamar uma ferramenta.

### `tool_result`

Bloco enviado pela aplicação para Claude contendo o resultado da execução da ferramenta.

### `tool_use_id`

Identificador que conecta um `tool_result` ao `tool_use` correspondente.

### `content`

Campo que contém o resultado da ferramenta, normalmente serializado como string.

### `is_error`

Campo booleano que indica se a execução da ferramenta falhou.

### `input`

Campo do `tool_use` com os argumentos que devem ser passados para a função.

### Unpacking `**`

Sintaxe do Python usada para transformar um dicionário em argumentos nomeados de função.

### Conversation history

Histórico completo da conversa enviado manualmente para Claude em cada chamada da API.

### Tool schema

JSON Schema que descreve a ferramenta disponível para Claude.

### Follow-up request

Chamada posterior feita para Claude depois que a aplicação executa a ferramenta e adiciona o resultado ao histórico.

---

## 5. Boas práticas

* Sempre leia o campo `input` do bloco `tool_use` para obter os argumentos.
* Use o `name` do bloco `tool_use` para decidir qual função executar.
* Use o `id` do bloco `tool_use` para preencher o `tool_use_id` no resultado.
* Envie o resultado da ferramenta em uma nova mensagem com role `user`.
* Serialize o resultado como string no campo `content`.
* Use `is_error: False` quando a execução for bem-sucedida.
* Use `is_error: True` quando ocorrer erro.
* Preserve o histórico completo da conversa.
* Não descarte a mensagem do assistant com `tool_use`.
* Inclua o schema da ferramenta novamente na chamada final.
* Em múltiplas chamadas de ferramenta, associe cada resultado pelo ID correto.
* Não dependa apenas da ordem dos resultados.
* Valide os argumentos antes de executar funções sensíveis.
* Trate erros da ferramenta e informe Claude usando `is_error`.

---

## 6. Limitações, riscos e cuidados

* Claude não executa a ferramenta sozinho; a aplicação precisa executar.
* Se o `tool_use_id` estiver errado, Claude pode associar o resultado à chamada incorreta.
* Se o histórico completo não for enviado, Claude pode perder o contexto.
* Se o bloco `tool_use` for descartado, o `tool_result` pode não fazer sentido para Claude.
* Se o resultado não for serializado corretamente como string, a chamada pode falhar ou gerar comportamento inesperado.
* Em múltiplas ferramentas, confiar apenas na ordem dos blocos pode causar erros.
* O schema da ferramenta deve continuar sendo enviado na chamada final.
* Se a ferramenta falhar e `is_error` não for marcado corretamente, Claude pode interpretar o resultado como sucesso.
* A aplicação deve ter cuidado ao executar funções reais, especialmente se elas acessam arquivos, rede, banco de dados ou sistemas externos.
* É importante validar e limitar ferramentas disponíveis para reduzir riscos de segurança.

---

## 7. Resumo para revisão rápida

* Claude retorna `tool_use` quando quer usar uma ferramenta.
* A aplicação executa a função real.
* Os argumentos ficam em `response.content[1].input`.
* Em Python, use `**input` para passar argumentos nomeados.
* O resultado volta para Claude como `tool_result`.
* `tool_result` fica em uma mensagem de usuário.
* `tool_use_id` deve bater com o `id` do `tool_use`.
* `content` contém a saída da ferramenta como string.
* `is_error` indica sucesso ou erro.
* Claude pode solicitar várias ferramentas de uma vez.
* Cada resultado deve ser ligado ao ID correto.
* A ordem dos resultados não é o fator principal; o ID é.
* O follow-up precisa incluir o histórico completo.
* A chamada final ainda deve incluir o schema da ferramenta.
* Claude usa o resultado para responder ao usuário em linguagem natural.

---

## 8. Perguntas simuladas

### Pergunta 1

Depois que Claude retorna um bloco `tool_use`, quem executa a função real?

A. Claude internamente, sem intervenção da aplicação.
B. A aplicação ou servidor do desenvolvedor.
C. O JSON Schema automaticamente.
D. O usuário manualmente no chat.

**Resposta correta: B**

**Explicação:** Claude solicita a chamada da ferramenta, mas a execução real acontece no código da aplicação.

---

### Pergunta 2

Onde ficam os argumentos que Claude quer passar para a ferramenta?

A. `response.model`
B. `response.content[1].input`
C. `messages[0].role`
D. `tools[0].description`

**Resposta correta: B**

**Explicação:** O campo `input` do bloco `tool_use` contém os argumentos da chamada.

---

### Pergunta 3

O que faz a sintaxe `**response.content[1].input` em Python?

A. Converte o resultado em JSON Schema.
B. Desempacota um dicionário como argumentos nomeados para a função.
C. Remove o histórico da conversa.
D. Cria automaticamente um bloco `tool_result`.

**Resposta correta: B**

**Explicação:** O operador `**` passa pares chave-valor de um dicionário como argumentos nomeados.

---

### Pergunta 4

Qual bloco deve ser usado para enviar o resultado da ferramenta de volta para Claude?

A. `text`
B. `tool_use`
C. `tool_result`
D. `assistant_result`

**Resposta correta: C**

**Explicação:** O resultado da execução da ferramenta deve ser enviado em um bloco `tool_result`.

---

### Pergunta 5

Em qual tipo de mensagem o bloco `tool_result` deve ser colocado?

A. Em uma mensagem com role `assistant`.
B. Em uma mensagem com role `user`.
C. Em uma mensagem com role `system`.
D. Em uma mensagem sem role.

**Resposta correta: B**

**Explicação:** O `tool_result` é enviado dentro da lista `content` de uma mensagem de usuário.

---

### Pergunta 6

Qual campo do `tool_result` deve corresponder ao `id` do bloco `tool_use`?

A. `content`
B. `type`
C. `tool_use_id`
D. `is_error`

**Resposta correta: C**

**Explicação:** `tool_use_id` conecta o resultado da ferramenta à chamada original feita por Claude.

---

### Pergunta 7

O que deve conter o campo `content` de um `tool_result`?

A. O nome da ferramenta apenas.
B. A saída da ferramenta, serializada como string.
C. O schema completo da ferramenta.
D. O prompt original do usuário.

**Resposta correta: B**

**Explicação:** O campo `content` contém o resultado da execução da função, geralmente como string.

---

### Pergunta 8

Para que serve o campo `is_error`?

A. Para indicar se ocorreu erro ao executar a ferramenta.
B. Para informar se Claude deve esquecer o histórico.
C. Para escolher o modelo da próxima chamada.
D. Para definir o nome da função.

**Resposta correta: A**

**Explicação:** `is_error` deve ser `True` quando a execução da ferramenta falha e `False` quando é bem-sucedida.

---

### Pergunta 9

Se Claude solicitar duas chamadas de ferramenta em uma resposta, como a aplicação deve associar cada resultado?

A. Pela ordem alfabética do nome da ferramenta.
B. Pelo campo `tool_use_id`, correspondente ao ID de cada `tool_use`.
C. Pela posição visual na interface.
D. Pelo tamanho do conteúdo retornado.

**Resposta correta: B**

**Explicação:** Cada chamada possui um ID único. O resultado correto deve usar o `tool_use_id` correspondente.

---

### Pergunta 10

Em múltiplas chamadas de ferramenta, os resultados precisam ser enviados na mesma ordem em que foram solicitados?

A. Sim, sempre; o ID é irrelevante.
B. Não necessariamente; o mais importante é que cada `tool_use_id` corresponda ao ID correto.
C. Não, porque Claude ignora todos os IDs.
D. Sim, mas apenas quando a ferramenta é de calculadora.

**Resposta correta: B**

**Explicação:** A ordem pode variar, desde que cada resultado esteja associado ao ID correto.

---

### Pergunta 11

Qual histórico deve ser enviado na chamada de follow-up para Claude?

A. Apenas o resultado da ferramenta.
B. Apenas a primeira mensagem do usuário.
C. O histórico completo: mensagem original, `tool_use` do assistant e `tool_result` do usuário.
D. Apenas o schema da ferramenta.

**Resposta correta: C**

**Explicação:** Claude precisa do histórico completo para entender o pedido, a chamada da ferramenta e o resultado retornado.

---

### Pergunta 12

Na chamada final para Claude, após enviar o `tool_result`, o schema da ferramenta ainda deve ser incluído?

A. Sim, porque Claude precisa entender as referências de ferramenta no histórico.
B. Não, schemas nunca devem ser reenviados.
C. Não, porque o `tool_result` substitui o schema.
D. Apenas se ocorreu erro.

**Resposta correta: A**

**Explicação:** Mesmo sem esperar nova chamada de ferramenta, o schema deve ser enviado para Claude interpretar corretamente o histórico.

---

### Pergunta 13

Qual estrutura representa corretamente um bloco `tool_result`?

A.

```json
{
  "type": "tool_result",
  "tool_use_id": "toolu_01...",
  "content": "12:47:13",
  "is_error": false
}
```

B.

```json
{
  "type": "tool_use",
  "name": "get_current_datetime",
  "input": {}
}
```

C.

```json
{
  "role": "assistant",
  "text": "done"
}
```

D.

```json
{
  "schema": "tool_result",
  "model": "sonnet"
}
```

**Resposta correta: A**

**Explicação:** Um `tool_result` precisa conter `type`, `tool_use_id`, `content` e `is_error`.

---

### Pergunta 14

Qual é um erro comum ao enviar resultados de ferramentas?

A. Associar o resultado ao `tool_use_id` correto.
B. Preservar o histórico completo.
C. Enviar o resultado sem corresponder ao ID correto do `tool_use`.
D. Incluir o schema da ferramenta na chamada final.

**Resposta correta: C**

**Explicação:** Se o `tool_use_id` não corresponder ao `id` certo, Claude pode interpretar o resultado de forma incorreta.
