# Streaming de resposta

```
005_streaming.ipynb
```

## 1. Ideia central

O conteúdo explica o conceito de **response streaming** na API da Anthropic.

A ideia principal é que, em vez de esperar Claude gerar a resposta inteira para só então exibi-la ao usuário, a aplicação pode receber e mostrar a resposta **em pequenos pedaços**, conforme o modelo vai gerando o texto.

Isso melhora muito a experiência do usuário em aplicações de chat, porque evita que a pessoa fique esperando olhando para um loading spinner por 10, 20 ou 30 segundos.

Em resumo:

**Sem streaming:** o usuário espera a resposta completa.
**Com streaming:** o usuário vê a resposta aparecer gradualmente.

---

## 2. Principais conceitos

### O problema das respostas tradicionais

Em uma chamada comum à API, o fluxo é assim:

1. O usuário envia uma mensagem.
2. O servidor envia a requisição para Claude.
3. Claude gera a resposta completa.
4. Só depois a API devolve a resposta.
5. O servidor envia o texto final para o cliente.

O problema é que, enquanto Claude está gerando, o usuário não vê nada além de um indicador de carregamento.

Isso pode causar uma sensação de lentidão, especialmente quando a resposta leva vários segundos para ser concluída.

---

### Como o streaming funciona

Com **streaming**, Claude começa a enviar eventos assim que a geração começa.

A resposta não chega inteira de uma vez. Ela chega em partes pequenas, chamadas de **chunks** ou eventos de stream.

A aplicação pode encaminhar esses pedaços para o front-end conforme eles chegam.

Assim, o usuário vê algo parecido com:

```text
Claude está escrevendo a resposta aos poucos...
```

Esse comportamento dá uma sensação de velocidade e responsividade, mesmo que o tempo total de geração seja parecido.

---

### Streaming ainda é uma única requisição

Um ponto importante para certificação: todos os eventos recebidos durante o streaming fazem parte de **uma única requisição** para Claude.

Ou seja, o streaming não significa fazer várias chamadas separadas para a API.

A diferença é apenas a forma como a resposta é entregue: em partes, em vez de um bloco completo no final.

---

### Tipos de eventos de stream

Quando o streaming está ativado, Claude envia vários tipos de eventos.

| Evento              | Significado                                               |
| ------------------- | --------------------------------------------------------- |
| `MessageStart`      | Indica que uma nova mensagem começou                      |
| `ContentBlockStart` | Início de um bloco de conteúdo                            |
| `ContentBlockDelta` | Pedaço do texto gerado                                    |
| `ContentBlockStop`  | Fim de um bloco de conteúdo                               |
| `MessageDelta`      | Atualização indicando que a mensagem está sendo concluída |
| `MessageStop`       | Fim das informações da mensagem                           |

O evento mais importante para exibir texto ao usuário é:

```text
ContentBlockDelta
```

Ele contém os pedaços reais do texto gerado.

---

### Implementação básica com `stream=True`

Para ativar streaming em uma chamada comum, basta adicionar:

```python
stream=True
```

Exemplo:

```python
messages = []
add_user_message(messages, "Write a 1 sentence description of a fake database")

stream = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=messages,
    stream=True
)

for event in stream:
    print(event)
```

Nesse exemplo, o código imprime todos os eventos do stream.

Isso é útil para entender o funcionamento interno, mas em uma aplicação real normalmente você quer apenas o texto gerado.

---

### Streaming simplificado de texto

O SDK também oferece uma interface mais simples para streaming, usando:

```python
client.messages.stream()
```

Exemplo:

```python
with client.messages.stream(
    model=model,
    max_tokens=1000,
    messages=messages
) as stream:
    for text in stream.text_stream:
        print(text, end="")
```

Essa abordagem filtra automaticamente os eventos e retorna apenas o texto.

Ou seja, em vez de você lidar com `MessageStart`, `ContentBlockDelta`, `MessageStop` e outros eventos, você recebe diretamente os pedaços de texto em:

```python
stream.text_stream
```

Esse método é mais prático para interfaces de chat.

---

### Obtendo a mensagem completa

Mesmo usando streaming, muitas aplicações precisam salvar a resposta completa no banco de dados ou usá-la em outra etapa do sistema.

Para isso, depois que o streaming termina, é possível chamar:

```python
final_message = stream.get_final_message()
```

Exemplo:

```python
with client.messages.stream(
    model=model,
    max_tokens=1000,
    messages=messages
) as stream:
    for text in stream.text_stream:
        # Enviar cada pedaço para o cliente
        pass
    
    # Obter a mensagem completa para salvar no banco
    final_message = stream.get_final_message()
```

Esse padrão oferece o melhor dos dois mundos:

* o usuário vê a resposta em tempo real;
* a aplicação ainda obtém o objeto final completo.

---

## 3. Pontos importantes para certificação

* **Response streaming** permite receber a resposta em partes.
* Streaming melhora a experiência do usuário em aplicações de chat.
* Sem streaming, a aplicação espera a resposta completa antes de exibir algo.
* Com streaming, o texto aparece conforme Claude gera.
* O streaming evita a sensação de “travamento” ou espera longa.
* Todos os eventos de streaming pertencem a **uma única requisição**.
* O evento mais importante para exibir texto é `ContentBlockDelta`.
* Para ativar streaming em `messages.create`, usa-se `stream=True`.
* O SDK oferece uma interface simplificada com `client.messages.stream()`.
* `stream.text_stream` retorna apenas os pedaços de texto.
* `stream.get_final_message()` retorna a mensagem completa após o fim do streaming.
* Streaming é útil para UI, mas a mensagem final ainda é importante para armazenamento, logs e histórico de conversa.

---

## 4. Termos-chave

**Response streaming**
Técnica que permite receber a resposta do modelo em partes, conforme ela é gerada.

**Chunk**
Pequeno pedaço da resposta recebido durante o streaming.

**Stream event**
Evento enviado pela API durante o processo de streaming.

**`stream=True`**
Parâmetro usado para ativar streaming em uma chamada `client.messages.create()`.

**`client.messages.stream()`**
Interface simplificada do SDK para trabalhar com streaming.

**`text_stream`**
Iterador que retorna apenas os pedaços de texto gerado, sem exigir parsing manual dos eventos.

**`ContentBlockDelta`**
Evento que contém partes reais do texto gerado.

**`MessageStart`**
Evento que indica o início de uma nova mensagem.

**`ContentBlockStart`**
Evento que indica o início de um bloco de conteúdo.

**`ContentBlockStop`**
Evento que indica o fim de um bloco de conteúdo.

**`MessageDelta`**
Evento de atualização indicando progresso ou conclusão da mensagem.

**`MessageStop`**
Evento que indica o fim da mensagem.

**`get_final_message()`**
Método usado para obter a mensagem completa depois que o streaming terminou.

**Loading spinner**
Indicador visual de carregamento exibido enquanto o usuário aguarda resposta.

---

## 5. Boas práticas

Use streaming em aplicações de chat, assistentes interativos e interfaces em que a experiência do usuário importa.

Prefira a interface simplificada:

```python
client.messages.stream()
```

quando você só precisa exibir texto ao usuário.

Use:

```python
stream.text_stream
```

para mostrar a resposta conforme ela chega.

Depois do streaming, use:

```python
stream.get_final_message()
```

para obter a resposta completa e salvá-la no histórico, banco de dados ou logs.

Também é uma boa prática separar duas responsabilidades:

* **streaming para a interface do usuário**;
* **mensagem final para persistência e processamento interno**.

Em aplicações reais, o servidor pode receber os pedaços da API da Anthropic e repassá-los ao cliente usando tecnologias como WebSockets, Server-Sent Events ou streaming HTTP.

---

## 6. Limitações, riscos e cuidados

Streaming melhora a percepção de velocidade, mas não significa necessariamente que Claude terminou de gerar mais rápido.

O tempo total da resposta pode ser semelhante ao de uma chamada normal. A diferença é que o usuário começa a ver texto antes.

Outro cuidado é que os chunks chegam incompletos. Portanto, a aplicação não deve tratar cada pedaço como uma resposta final.

Por exemplo, um chunk pode conter apenas parte de uma frase. Para salvar, analisar ou reutilizar a resposta, é melhor usar a mensagem final completa.

Também é importante lembrar que lidar manualmente com eventos de stream pode ser mais complexo. Por isso, quando o objetivo é apenas exibir texto, `stream.text_stream` é mais simples.

Em sistemas de produção, é necessário tratar falhas durante o streaming, como:

* conexão interrompida;
* resposta parcial;
* erro de rede;
* cancelamento pelo usuário;
* inconsistência entre texto exibido e texto salvo.

---

## 7. Resumo para revisão rápida

* Streaming permite mostrar a resposta enquanto Claude gera.
* Sem streaming, o usuário espera a resposta completa.
* Com streaming, o texto chega em chunks.
* Streaming melhora a experiência em chats.
* Todos os eventos pertencem a uma única requisição.
* `ContentBlockDelta` contém o texto gerado.
* `stream=True` ativa streaming em `messages.create`.
* `client.messages.stream()` simplifica o uso.
* `stream.text_stream` retorna apenas texto.
* `stream.get_final_message()` retorna a mensagem completa.
* Use streaming para UI e mensagem final para armazenamento.

---

## 8. Perguntas simuladas

### 1. Qual problema o response streaming resolve?

A) Elimina a necessidade de API key
B) Permite que o usuário veja a resposta aparecer enquanto Claude gera o texto
C) Aumenta automaticamente a janela de contexto
D) Faz Claude armazenar conversas permanentemente

**Resposta correta: B.**

**Explicação:** Streaming melhora a experiência do usuário porque mostra a resposta em partes, em vez de esperar a resposta completa.

---

### 2. O que acontece em uma resposta tradicional sem streaming?

A) A resposta chega em pequenos pedaços
B) O cliente recebe todos os tokens antes do servidor
C) O servidor espera a resposta completa antes de enviar algo ao cliente
D) Claude envia apenas eventos `ContentBlockDelta`

**Resposta correta: C.**

**Explicação:** Sem streaming, a aplicação geralmente só mostra a resposta depois que Claude terminou de gerar tudo.

---

### 3. Em streaming, os vários eventos recebidos representam o quê?

A) Várias requisições separadas para Claude
B) Uma única requisição com resposta entregue em partes
C) Várias API keys diferentes
D) Múltiplos modelos respondendo ao mesmo tempo

**Resposta correta: B.**

**Explicação:** Todos os eventos do streaming fazem parte de uma única chamada à API.

---

### 4. Qual evento contém os pedaços reais do texto gerado?

A) `MessageStart`
B) `ContentBlockDelta`
C) `MessageStop`
D) `ContentBlockStop`

**Resposta correta: B.**

**Explicação:** `ContentBlockDelta` contém os chunks do texto que devem ser exibidos ao usuário.

---

### 5. Como ativar streaming usando `client.messages.create()`?

A) Adicionando `stream=True`
B) Usando `max_tokens=0`
C) Removendo o campo `messages`
D) Trocando `role` para `"stream"`

**Resposta correta: A.**

**Explicação:** O parâmetro `stream=True` ativa o envio da resposta em eventos de stream.

---

### 6. Qual interface simplificada do SDK pode ser usada para streaming?

A) `client.messages.batch()`
B) `client.messages.stream()`
C) `client.tokens.stream()`
D) `client.responses.final()`

**Resposta correta: B.**

**Explicação:** `client.messages.stream()` fornece uma forma mais prática de trabalhar com streaming.

---

### 7. Para que serve `stream.text_stream`?

A) Para retornar apenas os pedaços de texto gerado
B) Para criar uma API key temporária
C) Para apagar eventos do servidor
D) Para definir o modelo usado

**Resposta correta: A.**

**Explicação:** `stream.text_stream` filtra os eventos e entrega apenas o conteúdo textual.

---

### 8. Para que serve `stream.get_final_message()`?

A) Para obter a mensagem completa depois do streaming
B) Para interromper o streaming antes do fim
C) Para alterar o system prompt
D) Para reduzir automaticamente o custo da chamada

**Resposta correta: A.**

**Explicação:** Depois que o streaming termina, esse método retorna o objeto final completo da mensagem.

---

### 9. Por que ainda precisamos da mensagem completa se já exibimos os chunks ao usuário?

A) Para salvar no banco de dados, histórico ou usar em processamento posterior
B) Para esconder a resposta do usuário
C) Para gerar uma nova API key
D) Para impedir que Claude responda novamente

**Resposta correta: A.**

**Explicação:** Os chunks são úteis para UI em tempo real, mas a aplicação geralmente precisa da resposta final consolidada.

---

### 10. Qual é um cuidado ao usar streaming?

A) Tratar cada chunk como uma resposta final independente
B) Ignorar falhas de conexão
C) Lembrar que chunks podem ser partes incompletas de frases
D) Colocar a API key no front-end para acelerar a resposta

**Resposta correta: C.**

**Explicação:** Cada chunk pode conter apenas parte de uma frase ou palavra. A resposta final deve ser reconstruída ou obtida com `get_final_message()`.
