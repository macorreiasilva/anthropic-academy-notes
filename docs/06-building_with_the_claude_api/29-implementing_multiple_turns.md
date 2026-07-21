# Implementando múltiplas turnos de uso de ferramentas

```
notebooks\16-implementing_multiple_turns.ipynb
```

## 1. Ideia central

A aula explica como implementar, na prática, um sistema de conversa com Claude que suporte **múltiplos turnos de uso de ferramentas**.

A ideia principal é criar um loop que continue chamando Claude enquanto ele estiver solicitando ferramentas. Quando Claude parar de pedir `tool_use`, significa que ele já tem informação suficiente para entregar a resposta final ao usuário.

Esse padrão é essencial para aplicações reais, porque Claude pode precisar usar uma ou várias ferramentas, em uma ou várias rodadas, antes de responder.

---

## 2. Principais conceitos

### Detectando quando Claude quer usar uma ferramenta

O campo mais importante para saber se Claude está pedindo uma ferramenta é:

```python
response.stop_reason
```

Quando Claude decide chamar uma ferramenta, esse campo recebe o valor:

```python
"tool_use"
```

Exemplo:

```python
if response.stop_reason != "tool_use":
    break
```

Isso significa:

* Se `stop_reason == "tool_use"`, Claude quer usar uma ferramenta.
* Se `stop_reason != "tool_use"`, Claude terminou e provavelmente já tem uma resposta final.

Esse é o mecanismo usado para controlar o loop da conversa.

---

### Loop de conversa com ferramentas

A função principal da aula segue este padrão:

```python
def run_conversation(messages):
    while True:
        response = chat(messages, tools=[get_current_datetime_schema])
        add_assistant_message(messages, response)
        print(text_from_message(response))

        if response.stop_reason != "tool_use":
            break

        tool_results = run_tools(response)
        add_user_message(messages, tool_results)

    return messages
```

Esse loop faz o seguinte:

1. Envia o histórico atual para Claude.
2. Recebe uma resposta.
3. Adiciona a resposta do assistant ao histórico.
4. Exibe os blocos de texto da resposta.
5. Verifica se Claude pediu ferramenta.
6. Se não pediu, encerra o loop.
7. Se pediu, executa as ferramentas solicitadas.
8. Adiciona os resultados como mensagem de usuário.
9. Repete o processo.

---

### Processando múltiplos `ToolUseBlock`

Claude pode pedir mais de uma ferramenta em uma única resposta.

A mensagem pode conter uma lista de blocos como:

```text
TextBlock:
"I'll solve these"

ToolUseBlock:
id='ab3'
input={'expression': '10 + 10'}
name='calculator'
type='tool_use'

ToolUseBlock:
id='po9'
input={'expression': '30 + 30'}
name='calculator'
type='tool_use'
```

Nesse caso, a aplicação precisa processar **cada bloco `tool_use` separadamente**.

A função `run_tools` começa filtrando apenas os blocos de ferramenta:

```python
def run_tools(message):
    tool_requests = [
        block for block in message.content if block.type == "tool_use"
    ]
    tool_result_blocks = []

    for tool_request in tool_requests:
        # Process each tool request...
```

Essa abordagem evita depender de posições fixas como `response.content[1]`, que podem falhar quando há vários blocos ou quando a ordem muda.

---

### Executando cada ferramenta solicitada

Para cada `ToolUseBlock`, a aplicação deve:

1. Identificar o nome da ferramenta.
2. Ler os inputs.
3. Executar a função correspondente.
4. Criar um `ToolResultBlock`.
5. Adicionar esse bloco à lista de resultados.

Exemplo conceitual:

```python
tool_result_block = {
    "type": "tool_result",
    "tool_use_id": tool_request.id,
    "content": json.dumps(tool_output),
    "is_error": False
}
```

O campo mais importante aqui é:

```python
"tool_use_id": tool_request.id
```

Ele conecta o resultado da ferramenta ao pedido original de Claude.

---

### `ToolResultBlock`

Cada `ToolUseBlock` precisa receber um `ToolResultBlock` correspondente.

Estrutura básica:

```python
tool_result_block = {
    "type": "tool_result",
    "tool_use_id": tool_request.id,
    "content": json.dumps(tool_output),
    "is_error": False
}
```

Campos principais:

* `type`: indica que o bloco é um resultado de ferramenta.
* `tool_use_id`: deve corresponder ao `id` do `ToolUseBlock`.
* `content`: resultado da ferramenta, serializado como string.
* `is_error`: indica se houve erro na execução.

---

### Tratamento de erros

Aplicações robustas precisam lidar com falhas na execução de ferramentas.

Exemplo:

```python
try:
    tool_output = run_tool(tool_request.name, tool_request.input)
    tool_result_block = {
        "type": "tool_result",
        "tool_use_id": tool_request.id,
        "content": json.dumps(tool_output),
        "is_error": False
    }
except Exception as e:
    tool_result_block = {
        "type": "tool_result",
        "tool_use_id": tool_request.id,
        "content": f"Error: {e}",
        "is_error": True
    }
```

Mesmo quando uma ferramenta falha, a aplicação ainda deve retornar um bloco `tool_result` para Claude.

A diferença é que, nesse caso:

```python
"is_error": True
```

Isso permite que Claude saiba que a ferramenta falhou e responda de forma apropriada.

---

### Roteamento de ferramentas

Para suportar várias ferramentas, a aula recomenda criar uma função de roteamento.

Exemplo:

```python
def run_tool(tool_name, tool_input):
    if tool_name == "get_current_datetime":
        return get_current_datetime(**tool_input)
    elif tool_name == "another_tool":
        return another_tool(**tool_input)
```

Essa função mapeia o nome da ferramenta solicitada por Claude para a implementação real no código.

Esse padrão é importante porque separa:

* A lógica geral da conversa.
* A lógica específica de cada ferramenta.

Assim, fica mais fácil adicionar novas tools sem alterar o fluxo principal.

---

### Fluxo completo

O fluxo completo é:

1. Usuário envia uma mensagem.
2. A aplicação envia a mensagem para Claude com os schemas das ferramentas.
3. Claude responde com texto e/ou blocos `tool_use`.
4. A aplicação executa todas as ferramentas solicitadas.
5. A aplicação cria blocos `tool_result`.
6. A aplicação envia os resultados como mensagem de usuário.
7. Claude pode pedir mais ferramentas ou responder de forma final.
8. O loop continua até não haver mais `tool_use`.

---

## 3. Pontos importantes para certificação

* `response.stop_reason` indica por que Claude parou de gerar resposta.
* Quando `response.stop_reason == "tool_use"`, Claude está solicitando uso de ferramenta.
* Quando `response.stop_reason != "tool_use"`, o loop pode encerrar.
* O loop de conversa deve continuar enquanto houver solicitação de ferramenta.
* A função `run_conversation` deve adicionar a resposta do assistant ao histórico.
* Depois de executar ferramentas, os resultados devem ser adicionados como mensagem de usuário.
* Claude pode solicitar múltiplas ferramentas em uma única resposta.
* A aplicação deve filtrar blocos com:

```python
block.type == "tool_use"
```

* Cada `ToolUseBlock` deve receber um `ToolResultBlock`.
* O `tool_use_id` deve corresponder ao `id` do bloco `tool_use`.
* O resultado da ferramenta deve ser serializado no campo `content`, por exemplo com `json.dumps`.
* `is_error: False` indica sucesso.
* `is_error: True` indica erro.
* Mesmo em caso de erro, é necessário retornar um `tool_result` para Claude.
* O roteamento de ferramentas deve mapear `tool_name` para a função correta.
* Separar `run_conversation`, `run_tools` e `run_tool` torna o código mais escalável.
* Não se deve assumir que haverá apenas uma ferramenta ou apenas um bloco.
* O histórico completo deve ser preservado ao longo de todas as rodadas.

---

## 4. Termos-chave

### `response.stop_reason`

Campo que indica por que Claude parou de gerar. Quando é `"tool_use"`, indica que Claude quer chamar uma ferramenta.

### `tool_use`

Tipo de bloco retornado por Claude quando ele solicita a execução de uma ferramenta.

### `ToolUseBlock`

Bloco que contém o nome da ferramenta, os inputs e o ID da chamada.

### `tool_result`

Tipo de bloco enviado pela aplicação com o resultado da ferramenta.

### `ToolResultBlock`

Bloco que contém o resultado da execução de uma ferramenta e o ID correspondente.

### `tool_use_id`

Campo que conecta um `tool_result` ao `tool_use` original.

### `run_conversation`

Função responsável por manter o loop da conversa até Claude parar de pedir ferramentas.

### `run_tools`

Função responsável por encontrar e executar todos os blocos `tool_use` de uma resposta.

### `run_tool`

Função de roteamento que escolhe qual implementação executar com base no nome da ferramenta.

### `json.dumps`

Função usada para serializar o resultado da ferramenta como string JSON.

### Tool routing

Padrão de código que direciona chamadas de ferramentas para as funções corretas.

---

## 5. Boas práticas

* Use `response.stop_reason` para detectar se Claude quer usar ferramenta.
* Não dependa de posições fixas como `response.content[1]`.
* Percorra todos os blocos de `message.content`.
* Filtre blocos com `block.type == "tool_use"`.
* Execute cada ferramenta solicitada.
* Retorne um `tool_result` para cada `tool_use`.
* Sempre preserve o `tool_use_id` correto.
* Serialize a saída da ferramenta com `json.dumps`.
* Implemente tratamento de erro com `try/except`.
* Mesmo em caso de falha, envie um `tool_result` com `is_error: True`.
* Use uma função de roteamento para organizar múltiplas ferramentas.
* Separe a lógica do loop da lógica de execução das ferramentas.
* Defina um limite de iterações em aplicações reais para evitar loops infinitos.
* Valide inputs antes de executar ferramentas sensíveis.
* Continue enviando os schemas de ferramentas enquanto estiver no fluxo com tools.

---

## 6. Limitações, riscos e cuidados

* Se o código não verificar `stop_reason`, pode encerrar cedo demais ou continuar sem necessidade.
* Se a aplicação assumir apenas uma ferramenta, pode falhar quando Claude solicitar múltiplas tools.
* Se a aplicação usar posições fixas da lista de blocos, pode ignorar ferramentas ou processar o bloco errado.
* Se o `tool_use_id` estiver incorreto, Claude pode associar resultados às chamadas erradas.
* Se erros não forem enviados com `is_error: True`, Claude pode interpretar falhas como resultados válidos.
* Se não houver tratamento de exceções, uma falha em uma ferramenta pode quebrar todo o fluxo.
* Se o loop não tiver limite, há risco de execução indefinida.
* Se o roteamento de ferramentas for mal feito, Claude pode pedir uma tool válida, mas a aplicação não saber executá-la.
* Ferramentas que executam ações reais devem ter validação, permissões e limites claros.

---

## 7. Resumo para revisão rápida

* Use `response.stop_reason` para detectar tool use.
* `stop_reason == "tool_use"` significa que Claude quer ferramenta.
* O loop continua enquanto Claude pedir tools.
* O loop para quando não houver mais `tool_use`.
* Claude pode solicitar várias ferramentas em uma resposta.
* Filtre blocos com `block.type == "tool_use"`.
* Execute cada ferramenta separadamente.
* Para cada `tool_use`, crie um `tool_result`.
* `tool_use_id` deve corresponder ao `id` original.
* Use `json.dumps` para serializar o resultado.
* Use `is_error: False` em sucesso.
* Use `is_error: True` em falha.
* Mesmo erros devem ser enviados de volta para Claude.
* Use uma função `run_tool` para rotear nomes de tools para funções reais.
* Preserve o histórico completo durante todo o fluxo.

---

## 8. Perguntas simuladas

### Pergunta 1

Qual campo da resposta indica que Claude quer usar uma ferramenta?

A. `response.model`
B. `response.stop_reason`
C. `response.temperature`
D. `response.role`

**Resposta correta: B**

**Explicação:** O campo `stop_reason` indica por que Claude parou. Quando seu valor é `"tool_use"`, Claude está solicitando uma ferramenta.

---

### Pergunta 2

O que significa `response.stop_reason == "tool_use"`?

A. Claude terminou a resposta final.
B. Claude quer que a aplicação execute uma ferramenta.
C. O usuário enviou um erro.
D. O schema da ferramenta está inválido.

**Resposta correta: B**

**Explicação:** Esse valor indica que a resposta contém uma solicitação de uso de ferramenta.

---

### Pergunta 3

Quando o loop de conversa deve parar?

A. Quando `response.stop_reason != "tool_use"`.
B. Sempre após a primeira resposta de Claude.
C. Quando houver qualquer bloco de texto.
D. Antes de adicionar a resposta ao histórico.

**Resposta correta: A**

**Explicação:** Se Claude não está pedindo ferramenta, entende-se que ele chegou à resposta final.

---

### Pergunta 4

Por que não é ideal acessar sempre `response.content[1]` para buscar uma tool?

A. Porque Claude nunca retorna listas.
B. Porque pode haver vários blocos, e a posição do bloco `tool_use` pode variar.
C. Porque `content[1]` sempre contém texto final.
D. Porque tools não têm conteúdo.

**Resposta correta: B**

**Explicação:** A resposta pode conter múltiplos blocos. O correto é filtrar por `block.type == "tool_use"`.

---

### Pergunta 5

Como encontrar todos os blocos de ferramenta em uma mensagem?

A.

```python
block for block in message.content if block.type == "tool_use"
```

B.

```python
message.content[0].text
```

C.

```python
message.role == "tool"
```

D.

```python
response.stop_reason == "end_turn"
```

**Resposta correta: A**

**Explicação:** É necessário percorrer `message.content` e selecionar apenas blocos cujo tipo seja `tool_use`.

---

### Pergunta 6

O que a função `run_tools` deve fazer?

A. Apagar o histórico da conversa.
B. Processar todos os blocos `tool_use` e retornar blocos `tool_result`.
C. Criar o JSON Schema das ferramentas.
D. Responder ao usuário sem chamar Claude.

**Resposta correta: B**

**Explicação:** `run_tools` identifica as ferramentas solicitadas, executa cada uma e monta os resultados correspondentes.

---

### Pergunta 7

Cada `ToolUseBlock` deve ser respondido com qual tipo de bloco?

A. `text`
B. `assistant_message`
C. `tool_result`
D. `system_result`

**Resposta correta: C**

**Explicação:** Cada chamada de ferramenta precisa receber um bloco `tool_result` correspondente.

---

### Pergunta 8

Qual campo conecta o resultado da ferramenta ao pedido original?

A. `content`
B. `tool_use_id`
C. `is_error`
D. `temperature`

**Resposta correta: B**

**Explicação:** `tool_use_id` deve corresponder ao `id` do `ToolUseBlock`.

---

### Pergunta 9

Qual é a função de `json.dumps(tool_output)` no `tool_result`?

A. Remover o resultado da ferramenta.
B. Serializar a saída da ferramenta como string.
C. Definir o nome da ferramenta.
D. Alterar o `stop_reason`.

**Resposta correta: B**

**Explicação:** O campo `content` do `tool_result` deve conter a saída serializada, geralmente como string JSON.

---

### Pergunta 10

O que deve acontecer se uma ferramenta falhar durante a execução?

A. A aplicação deve ignorar a falha e não responder a Claude.
B. A aplicação deve criar um `tool_result` com `is_error: True`.
C. A aplicação deve apagar o `tool_use`.
D. Claude deve executar a ferramenta sozinho.

**Resposta correta: B**

**Explicação:** Mesmo em erro, Claude precisa receber um resultado indicando a falha.

---

### Pergunta 11

Para que serve a função `run_tool(tool_name, tool_input)`?

A. Para mapear o nome da ferramenta para a implementação real.
B. Para gerar automaticamente a resposta final do usuário.
C. Para substituir o histórico da conversa.
D. Para transformar qualquer resposta em texto simples.

**Resposta correta: A**

**Explicação:** `run_tool` funciona como roteador, chamando a função correta conforme o nome solicitado por Claude.

---

### Pergunta 12

Qual é a principal vantagem de usar uma função de roteamento de tools?

A. Remover a necessidade de schemas.
B. Facilitar a adição de novas ferramentas sem mudar a lógica principal da conversa.
C. Impedir Claude de solicitar ferramentas.
D. Fazer com que ferramentas sejam executadas dentro do modelo.

**Resposta correta: B**

**Explicação:** O roteamento separa a lógica de conversa da execução específica de cada ferramenta.

---

### Pergunta 13

Qual é o fluxo correto de uma conversa multi-turn com tools?

A. Usuário pergunta → Claude responde final → aplicação ignora ferramentas.
B. Usuário pergunta → Claude pede ferramenta → aplicação executa → envia resultado → repete até resposta final.
C. Aplicação executa todas as ferramentas antes de Claude pedir.
D. Claude cria `tool_result` sem passar pela aplicação.

**Resposta correta: B**

**Explicação:** A aplicação deve executar ferramentas enquanto Claude solicitar `tool_use`, até ele produzir a resposta final.

---

### Pergunta 14

Qual cuidado é importante em aplicações reais com loops de tools?

A. Não usar tratamento de erros.
B. Definir limite máximo de iterações para evitar loops infinitos.
C. Descartar o histórico antigo.
D. Retornar apenas o primeiro bloco de conteúdo.

**Resposta correta: B**

**Explicação:** Um limite de iterações protege contra ciclos indefinidos de chamadas de ferramentas.
