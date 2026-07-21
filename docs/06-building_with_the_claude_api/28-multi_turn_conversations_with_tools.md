# Conversas com múltiplas interações usando ferramentas

```
notebooks\15-multi_turn_conversations_with_tools.ipynb
```

## 1. Ideia central

A aula explica como lidar com **conversas multi-turn com tools** na API do Claude.

Em casos simples, Claude pede uma ferramenta, a aplicação executa, devolve o resultado e Claude responde. Mas em casos mais complexos, Claude pode precisar chamar **várias ferramentas em sequência** antes de conseguir responder ao usuário.

Exemplo da aula:

> “What day is 103 days from today?”

Para responder, Claude precisa primeiro saber a data atual usando `get_current_datetime`. Depois, precisa somar 103 dias usando outra ferramenta, como `add_duration_to_datetime`. Só então ele terá informação suficiente para gerar a resposta final.

A ideia central é: **sua aplicação precisa manter um loop de conversa que continue executando tools enquanto Claude ainda estiver solicitando chamadas de ferramenta.**

---

## 2. Principais conceitos

### Conversas multi-turn com tools

Uma conversa multi-turn acontece quando Claude precisa de mais de uma rodada de interação com ferramentas.

Fluxo do exemplo:

1. Usuário pergunta:

```text
What day is 103 days from today?
```

2. Claude solicita a ferramenta:

```text
get_current_datetime
```

3. O servidor executa a função e devolve o resultado.

4. Claude percebe que ainda não tem informação suficiente.

5. Claude solicita outra ferramenta:

```text
add_duration_to_datetime
```

6. O servidor executa essa segunda função e devolve o resultado.

7. Claude finalmente responde ao usuário em linguagem natural.

Esse padrão mostra que o uso de tools pode ser **iterativo**, não apenas uma chamada única.

---

### Claude pode pedir ferramentas em sequência

Um ponto importante da aula é que Claude pode perceber, depois de receber o primeiro resultado, que ainda precisa de outra informação.

No exemplo:

* Saber a data atual não basta.
* Claude também precisa calcular a data futura.
* Por isso, ele faz uma segunda chamada de ferramenta.

Isso demonstra que aplicações robustas não devem assumir que haverá apenas um `tool_use`.

---

### Conversation loop

Para lidar com esse comportamento, a aplicação precisa implementar um **loop de conversa**.

Pseudocódigo apresentado na aula:

```python
def run_conversation(messages):
    while True:
        response = chat(messages)

        add_assistant_message(messages, response)

        # Pseudo code
        if response isn't asking for a tool:
            break

        tool_result_blocks = run_tools(response)
        add_user_message(messages, tool_result_blocks)

    return messages
```

A lógica é:

1. Enviar o histórico para Claude.
2. Receber a resposta.
3. Adicionar a resposta do assistant ao histórico.
4. Verificar se Claude pediu uma tool.
5. Se não pediu tool, a resposta final foi alcançada.
6. Se pediu tool, executar a ferramenta.
7. Adicionar o resultado como mensagem de usuário.
8. Chamar Claude novamente.
9. Repetir até não haver mais `tool_use`.

---

### Quando parar o loop

O loop deve parar quando Claude **não estiver mais solicitando uma ferramenta**.

Isso significa que Claude já tem informação suficiente para responder ao usuário.

A condição de parada é algo como:

```text
Se a resposta não contém tool_use, então é resposta final.
```

Esse é um ponto essencial para prova: **a presença ou ausência de blocos `tool_use` determina se a aplicação continua o loop ou encerra.**

---

### Atualização das helper functions

As funções auxiliares antigas provavelmente assumiam que mensagens eram apenas texto. Com tools, isso não é suficiente.

Agora, mensagens podem conter:

* Strings simples.
* Listas de blocos.
* Objetos completos de mensagem.
* Blocos `text`.
* Blocos `tool_use`.
* Blocos `tool_result`.

Por isso, a aula recomenda refatorar as funções auxiliares.

Exemplo:

```python
from anthropic.types import Message

def add_user_message(messages, message):
    user_message = {
        "role": "user",
        "content": message.content if isinstance(message, Message) else message
    }
    messages.append(user_message)
```

Essa função permite adicionar diferentes formatos de mensagem ao histórico.

---

### Atualização da função `chat`

A função `chat` também precisa mudar.

Antes, ela poderia retornar apenas texto. Agora, precisa retornar o **objeto completo da mensagem**, porque ele pode conter blocos de ferramenta.

Exemplo:

```python
def chat(messages, system=None, temperature=1.0, stop_sequences=[], tools=None):
    params = {
        "model": model,
        "max_tokens": 1000,
        "messages": messages,
        "temperature": temperature,
        "stop_sequences": stop_sequences,
    }

    if tools:
        params["tools"] = tools

    if system:
        params["system"] = system

    message = client.messages.create(**params)
    return message
```

Pontos importantes:

* A função aceita `tools`.
* Os schemas das tools são passados quando disponíveis.
* A função retorna a mensagem completa, não apenas texto.
* Isso preserva blocos como `tool_use`.

---

### Extração de texto da resposta final

Como agora a resposta pode conter múltiplos blocos, é útil criar uma função para extrair apenas o texto quando necessário.

Exemplo:

```python
def text_from_message(message):
    return "\n".join(
        [block.text for block in message.content if block.type == "text"]
    )
```

Essa função percorre os blocos da mensagem e junta apenas os blocos do tipo `text`.

Ela é útil para exibir a resposta final ao usuário, sem misturar blocos técnicos de tools.

---

## 3. Pontos importantes para certificação

* Claude pode precisar chamar várias tools em sequência para responder uma única pergunta.
* Uma conversa com tools pode ter múltiplos turnos antes da resposta final.
* Exemplo clássico:

  * Usuário pergunta uma data futura.
  * Claude chama `get_current_datetime`.
  * Depois chama `add_duration_to_datetime`.
  * Só então responde.
* A aplicação deve implementar um loop de conversa.
* O loop continua enquanto Claude retornar blocos `tool_use`.
* O loop para quando Claude não solicitar mais ferramentas.
* Cada resposta do assistant deve ser adicionada ao histórico completo.
* Cada resultado de tool deve ser enviado de volta como mensagem de usuário com `tool_result`.
* Helper functions precisam aceitar conteúdo estruturado, não apenas texto.
* A função `chat` deve retornar o objeto completo da mensagem.
* A função `chat` deve aceitar o parâmetro `tools`.
* Retornar apenas texto pode perder blocos importantes como `tool_use`.
* É útil criar uma função como `text_from_message` para extrair apenas blocos de texto.
* O histórico completo da conversa deve ser preservado em todas as chamadas.
* Aplicações robustas devem lidar com múltiplos blocos e múltiplas chamadas de ferramenta.

---

## 4. Termos-chave

### Multi-turn conversation

Conversa com várias rodadas entre usuário, Claude e ferramentas antes da resposta final.

### Tool loop

Loop que continua chamando Claude e executando tools até que Claude pare de solicitar ferramentas.

### `tool_use`

Bloco retornado por Claude quando ele quer que a aplicação execute uma ferramenta.

### `tool_result`

Bloco enviado pela aplicação para Claude com o resultado da execução da ferramenta.

### `get_current_datetime`

Ferramenta usada no exemplo para obter a data ou hora atual.

### `add_duration_to_datetime`

Ferramenta usada no exemplo para adicionar uma duração, como 103 dias, a uma data.

### Helper functions

Funções auxiliares usadas para adicionar mensagens, chamar Claude ou extrair texto.

### `Message`

Objeto completo de resposta da API, que pode conter múltiplos blocos.

### `text_from_message`

Função auxiliar para extrair apenas blocos de texto de uma mensagem complexa.

### Conversation history

Histórico completo de mensagens que a aplicação precisa manter manualmente.

---

## 5. Boas práticas

* Não assuma que Claude fará apenas uma chamada de ferramenta.
* Implemente um loop que continue até não haver mais `tool_use`.
* Preserve o histórico completo da conversa.
* Adicione cada resposta do assistant ao histórico.
* Adicione cada `tool_result` como uma nova mensagem de usuário.
* Retorne o objeto completo da mensagem na função `chat`.
* Passe os schemas de tools sempre que estiver em um fluxo com ferramentas.
* Atualize helpers para aceitar strings, listas de blocos e objetos completos.
* Use uma função separada para extrair texto final para exibição.
* Verifique todos os blocos da resposta, não apenas uma posição fixa como `response.content[1]`.
* Em produção, implemente tratamento de erros para tools.
* Defina um limite máximo de iterações para evitar loops infinitos.
* Valide entradas antes de executar ferramentas reais.

---

## 6. Limitações, riscos e cuidados

* Se a aplicação só tratar respostas como texto, ela perderá blocos `tool_use`.
* Se o histórico não for preservado, Claude pode perder contexto entre chamadas.
* Se o loop não for implementado, Claude pode parar no meio do processo sem resposta final.
* Se o loop não tiver limite, há risco de chamadas repetidas indefinidamente.
* Se helper functions antigas aceitarem apenas string, podem quebrar com mensagens multi-block.
* Se a função `chat` retornar apenas texto, a aplicação não conseguirá detectar tools.
* Claude pode pedir mais de uma tool ou pedir tools em sequência.
* A aplicação precisa mapear corretamente cada tool solicitada para uma função real.
* Ferramentas que acessam dados externos, arquivos ou sistemas sensíveis devem ter validação e controle de permissão.
* O desenvolvedor deve diferenciar resposta final de solicitação intermediária de ferramenta.

---

## 7. Resumo para revisão rápida

* Conversas com tools podem ter vários turnos.
* Claude pode precisar de várias ferramentas para responder uma pergunta.
* Exemplo: data atual + soma de 103 dias.
* A aplicação deve rodar um loop.
* O loop chama Claude, verifica `tool_use`, executa tools e devolve resultados.
* O loop para quando não houver mais `tool_use`.
* Helper functions devem aceitar mensagens estruturadas.
* A função `chat` deve aceitar `tools`.
* A função `chat` deve retornar o objeto completo da mensagem.
* Use `text_from_message` para extrair apenas texto da resposta final.
* Preserve sempre o histórico completo.
* Não trate `response.content` como string simples.
* Aplicações robustas precisam lidar com múltiplas tools e múltiplos blocos.

---

## 8. Perguntas simuladas

### Pergunta 1

Por que uma pergunta como “What day is 103 days from today?” pode exigir múltiplas chamadas de ferramentas?

A. Porque Claude nunca consegue responder perguntas sobre datas.
B. Porque Claude primeiro precisa saber a data atual e depois calcular a data futura.
C. Porque toda pergunta exige exatamente duas tools.
D. Porque tools só funcionam em pares.

**Resposta correta: B**

**Explicação:** Claude precisa primeiro chamar algo como `get_current_datetime` e depois usar outra ferramenta para somar 103 dias.

---

### Pergunta 2

O que é um conversation loop no contexto de tools?

A. Um loop que repete a resposta final várias vezes.
B. Um processo que continua chamando Claude e executando tools enquanto houver solicitações de ferramenta.
C. Um comando para apagar o histórico.
D. Um schema JSON usado para definir uma única tool.

**Resposta correta: B**

**Explicação:** O loop mantém a conversa ativa até Claude parar de solicitar `tool_use`.

---

### Pergunta 3

Quando o loop de conversa deve parar?

A. Quando Claude retorna uma resposta sem solicitar tool use.
B. Depois da primeira chamada de ferramenta, sempre.
C. Quando a mensagem do usuário é adicionada.
D. Quando o schema é removido.

**Resposta correta: A**

**Explicação:** Se não há `tool_use`, entende-se que Claude chegou à resposta final.

---

### Pergunta 4

No pseudocódigo da aula, o que acontece se Claude quiser usar uma ferramenta?

A. A aplicação encerra imediatamente.
B. A aplicação executa a ferramenta, coloca o resultado em uma mensagem de usuário e chama Claude novamente.
C. A aplicação ignora o pedido.
D. O schema é deletado.

**Resposta correta: B**

**Explicação:** Esse é o padrão correto: executar tool, adicionar `tool_result` e reenviar o histórico para Claude.

---

### Pergunta 5

Por que as helper functions precisam ser refatoradas?

A. Porque mensagens com tools podem conter blocos estruturados, não apenas texto simples.
B. Porque Claude não aceita mais mensagens de usuário.
C. Porque tools substituem o histórico da conversa.
D. Porque a API só retorna números.

**Resposta correta: A**

**Explicação:** Com tools, `content` pode conter blocos `text`, `tool_use` e `tool_result`.

---

### Pergunta 6

Qual é uma melhoria importante na função `chat`?

A. Ela deve retornar apenas o primeiro texto da resposta.
B. Ela deve retornar o objeto completo da mensagem.
C. Ela deve ignorar o parâmetro `tools`.
D. Ela deve apagar mensagens anteriores.

**Resposta correta: B**

**Explicação:** O objeto completo preserva todos os blocos, incluindo possíveis chamadas de ferramentas.

---

### Pergunta 7

Para que serve a função `text_from_message`?

A. Para executar uma ferramenta.
B. Para extrair os blocos de texto de uma mensagem complexa.
C. Para criar o schema JSON automaticamente.
D. Para substituir o campo `tool_use_id`.

**Resposta correta: B**

**Explicação:** Ela filtra blocos do tipo `text` e junta seu conteúdo para exibição ao usuário.

---

### Pergunta 8

Qual problema pode ocorrer se a aplicação retornar apenas texto da função `chat`?

A. Nenhum, porque tools sempre são texto.
B. A aplicação pode perder blocos `tool_use` e não saber que precisa executar uma ferramenta.
C. Claude passará a executar ferramentas sozinho.
D. O histórico será salvo automaticamente.

**Resposta correta: B**

**Explicação:** Blocos técnicos como `tool_use` são essenciais para controlar o fluxo de ferramentas.

---

### Pergunta 9

Qual destas opções melhor descreve o fluxo multi-turn com tools?

A. Usuário pergunta → Claude responde final imediatamente → aplicação ignora tools.
B. Usuário pergunta → Claude pede tool → aplicação executa → Claude pode pedir outra tool → aplicação executa → Claude responde final.
C. Aplicação executa todas as tools antes de Claude pedir.
D. Claude envia `tool_result` para si mesmo.

**Resposta correta: B**

**Explicação:** Esse é o padrão multi-turn, no qual Claude pode solicitar várias ferramentas antes da resposta final.

---

### Pergunta 10

Qual cuidado é recomendado para evitar loops infinitos?

A. Nunca usar tools.
B. Definir um limite máximo de iterações no loop.
C. Remover o histórico completo.
D. Ignorar todas as respostas com `tool_use`.

**Resposta correta: B**

**Explicação:** Em aplicações reais, é prudente limitar o número de ciclos de tool use para evitar execuções indefinidas.
