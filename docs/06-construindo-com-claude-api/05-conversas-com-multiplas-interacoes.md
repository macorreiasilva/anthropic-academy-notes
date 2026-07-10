# Conversas com múltiplas interações

# Resumo do conteúdo

## 1. Ideia central

A ideia central é que a API da Anthropic é **stateless**, ou seja, **Claude não armazena automaticamente o histórico da conversa entre chamadas**.

Cada requisição feita à API é independente. Portanto, se você quiser criar uma conversa com múltiplas interações, precisa **manter o histórico manualmente no seu próprio código** e reenviar esse histórico completo a cada nova chamada.

Em resumo:

**Claude não lembra conversas anteriores por conta própria.Sua aplicação precisa enviar o histórico completo em `messages`.**

---

## 2. Principais conceitos

### 1. Conversas stateless

Uma chamada stateless é uma chamada **sem memória de estado anterior**.

Isso significa que, quando você faz uma nova requisição à API, Claude só sabe o que está dentro daquela requisição específica.

Se antes você perguntou:

```
What is quantum computing?
```

e depois envia apenas:

```
Write another sentence
```

Claude não sabe que “another sentence” se refere à explicação sobre computação quântica. Para ele, essa segunda chamada não tem contexto anterior.

---

### 2. Estado da conversa

O **estado da conversa** é o histórico de mensagens trocadas entre usuário e assistente.

Esse histórico precisa ser armazenado pela aplicação, geralmente em uma lista chamada `messages`.

A lista deve conter tanto mensagens do usuário quanto respostas anteriores de Claude.

Exemplo conceitual:

```python
messages = [
    {"role": "user", "content": "Define quantum computing in one sentence"},
    {"role": "assistant", "content": "Quantum computing is..."},
    {"role": "user", "content": "Write another sentence"}
]
```

Quando essa lista completa é enviada, Claude consegue interpretar o pedido de continuação.

---

### 3. Envio do histórico completo

Para manter contexto em uma conversa multi-turn, a aplicação deve:

1. adicionar a primeira mensagem do usuário;
2. enviar a lista para Claude;
3. receber a resposta;
4. salvar a resposta como mensagem `"assistant"`;
5. adicionar a próxima mensagem do usuário;
6. reenviar todo o histórico.

O ponto importante é que não basta enviar apenas a última pergunta. É preciso enviar **a conversa inteira relevante**.

---

### 4. Papéis das mensagens

Cada mensagem na lista possui um papel, indicado por `role`.

Os papéis principais são:

| Role | Significado |
| --- | --- |
| `"user"` | Mensagem enviada pelo usuário |
| `"assistant"` | Resposta anterior gerada por Claude |

Esses papéis ajudam o modelo a entender a estrutura da conversa.

---

### 5. Funções auxiliares

O conteúdo apresenta três funções auxiliares para facilitar o gerenciamento da conversa.

A primeira adiciona mensagens do usuário:

```python
def add_user_message(messages, text):
    user_message = {"role": "user", "content": text}
    messages.append(user_message)
```

A segunda adiciona mensagens do assistente:

```python
def add_assistant_message(messages, text):
    assistant_message = {"role": "assistant", "content": text}
    messages.append(assistant_message)
```

A terceira envia a conversa para Claude:

```python
def chat(messages):
    message = client.messages.create(
        model=model,
        max_tokens=1000,
        messages=messages,
    )
    return message.content[0].text
```

Essas funções tornam o código mais organizado e reutilizável.

---

### 6. Exemplo demonstrado no conteúdo

O exemplo começa com uma lista vazia:

```python
messages = []
```

Depois adiciona uma pergunta inicial:

```python
add_user_message(messages, "Define quantum computing in one sentence")
```

Em seguida, envia a pergunta para Claude:

```python
answer = chat(messages)
```

Depois, salva a resposta de Claude no histórico:

```python
add_assistant_message(messages, answer)
```

Então adiciona uma pergunta de acompanhamento:

```python
add_user_message(messages, "Write another sentence")
```

Por fim, envia novamente o histórico completo:

```python
final_answer = chat(messages)
```

Esse exemplo demonstra o conceito de **manter contexto em conversas multi-turn**. Claude entende que “Write another sentence” se refere à resposta anterior porque a aplicação reenviou a conversa completa.

---

## 3. Pontos importantes para certificação

- Claude **não armazena automaticamente o histórico da conversa**.
- Cada chamada à API é **independente**.
- Para criar conversas multi-turn, a aplicação precisa **gerenciar o estado da conversa**.
- O histórico deve ser mantido em uma lista de mensagens.
- A cada nova requisição, deve-se enviar o **histórico completo relevante**.
- Mensagens do usuário usam `role: "user"`.
- Respostas anteriores de Claude devem ser salvas com `role: "assistant"`.
- Perguntas como “continue”, “explique melhor” ou “escreva outra frase” só funcionam bem se o histórico anterior for enviado junto.
- Funções auxiliares ajudam a organizar o gerenciamento da conversa.
- O histórico enviado aumenta o número de tokens de entrada, impactando custo e limite de contexto.

---

## 4. Termos-chave

**Multi-turn conversation**

Conversa com múltiplas interações entre usuário e assistente.

**Stateless**

Característica de uma API que não mantém memória automática entre chamadas.

**Conversation state**

Estado ou histórico da conversa mantido pela aplicação.

**Messages**

Lista de mensagens enviada para Claude a cada requisição.

**User message**

Mensagem escrita pelo usuário, com `role: "user"`.

**Assistant message**

Mensagem gerada anteriormente por Claude, com `role: "assistant"`.

**Message history**

Histórico completo ou parcial das mensagens anteriores.

**Context**

Informações anteriores necessárias para Claude entender a solicitação atual.

**Helper function**

Função auxiliar criada para simplificar tarefas repetitivas no código.

**`append()`**

Método Python usado para adicionar um item ao final de uma lista.

**`client.messages.create()`**

Função usada para enviar a lista de mensagens para Claude e obter uma resposta.

---

## 5. Boas práticas

- Sempre armazenar o histórico da conversa quando quiser criar experiências de chat.
- Adicionar respostas de Claude ao histórico como mensagens `"assistant"`.
- Adicionar novas entradas humanas como mensagens `"user"`.
- Reenviar o histórico completo ou a parte relevante do histórico a cada chamada.
- Criar funções auxiliares para evitar repetir código.
- Controlar o tamanho do histórico para não ultrapassar a janela de contexto.
- Remover ou resumir mensagens antigas quando a conversa ficar muito longa.
- Ter cuidado com custo, pois reenviar histórico aumenta tokens de entrada.
- Salvar o histórico no backend ou em um mecanismo seguro, não depender de memória interna do modelo.

---

## 6. Limitações, riscos e cuidados

A principal limitação é que Claude **não tem memória persistente automática via API**.

Isso significa que, se a aplicação não reenviar o histórico, Claude não saberá o que foi discutido antes.

Outro cuidado importante é o crescimento da lista `messages`. Em conversas longas, reenviar todo o histórico pode:

- aumentar o custo;
- aumentar a latência;
- ultrapassar o limite de contexto;
- incluir informações antigas ou irrelevantes.

Por isso, em aplicações reais, pode ser necessário implementar estratégias como:

- manter apenas as últimas mensagens;
- resumir partes antigas da conversa;
- armazenar histórico em banco de dados;
- recuperar apenas trechos relevantes;
- controlar dados sensíveis presentes no histórico.

Também é importante lembrar que salvar conversas envolve cuidados de privacidade, principalmente quando há dados pessoais, informações internas ou conteúdo sensível.

---

## 7. Resumo para revisão rápida

- A API do Claude é **stateless**.
- Claude não lembra automaticamente chamadas anteriores.
- Cada requisição é independente.
- Para conversas multi-turn, a aplicação precisa manter o histórico.
- O histórico fica em uma lista chamada `messages`.
- Mensagens do usuário usam `role: "user"`.
- Respostas de Claude usam `role: "assistant"`.
- A cada nova chamada, envie o histórico completo relevante.
- Perguntas de acompanhamento dependem do contexto anterior.
- Funções auxiliares ajudam a adicionar mensagens e chamar a API.
- Histórico longo aumenta custo, latência e uso de contexto.

---

## 8. Perguntas simuladas

### 1. O que significa dizer que a API do Claude é stateless?

A) Claude armazena todas as conversas automaticamente

B) Cada requisição é independente e não tem memória das anteriores

C) Claude só responde a uma mensagem por dia

D) A API não aceita histórico de conversa

**Resposta correta: B.**

**Explicação:** Stateless significa que a API não mantém estado automaticamente entre chamadas. A aplicação precisa enviar o contexto necessário.

---

### 2. O que acontece se o usuário perguntar “Write another sentence” sem enviar o histórico anterior?

A) Claude saberá automaticamente que deve continuar a resposta anterior

B) Claude poderá responder sem saber a que a frase se refere

C) A API sempre retornará erro

D) Claude recuperará a conversa do banco de dados da Anthropic

**Resposta correta: B.**

**Explicação:** Sem o histórico, Claude não tem contexto para entender o que “another sentence” significa.

---

### 3. Para manter uma conversa multi-turn, o que a aplicação deve fazer?

A) Enviar apenas a última mensagem do usuário

B) Salvar e reenviar o histórico da conversa

C) Criar uma nova API key para cada mensagem

D) Usar somente mensagens com role `"assistant"`

**Resposta correta: B.**

**Explicação:** O contexto é mantido reenviando o histórico completo ou relevante em `messages`.

---

### 4. Qual papel deve ser usado para mensagens escritas pelo usuário?

A) `"assistant"`

B) `"system"`

C) `"user"`

D) `"client"`

**Resposta correta: C.**

**Explicação:** Mensagens humanas são enviadas com `role: "user"`.

---

### 5. Qual papel deve ser usado para salvar respostas anteriores de Claude no histórico?

A) `"assistant"`

B) `"human"`

C) `"server"`

D) `"api"`

**Resposta correta: A.**

**Explicação:** Respostas anteriores geradas por Claude devem ser armazenadas como mensagens com `role: "assistant"`.

---

### 6. No exemplo, para que serve a função `add_user_message()`?

A) Para chamar diretamente a API da Anthropic

B) Para adicionar uma mensagem do usuário à lista `messages`

C) Para extrair o texto da resposta

D) Para apagar o histórico da conversa

**Resposta correta: B.**

**Explicação:** A função cria um dicionário com `role: "user"` e adiciona esse dicionário à lista de mensagens.

---

### 7. No exemplo, para que serve a função `add_assistant_message()`?

A) Para salvar a resposta de Claude como parte do histórico

B) Para instalar o SDK da Anthropic

C) Para definir o valor de `max_tokens`

D) Para trocar automaticamente o modelo usado

**Resposta correta: A.**

**Explicação:** Ela adiciona ao histórico uma mensagem com `role: "assistant"` contendo a resposta de Claude.

---

### 8. O que a função `chat(messages)` retorna no exemplo?

A) A API key usada na chamada

B) O texto da resposta gerada por Claude

C) A lista de modelos disponíveis

D) O arquivo `.env` atualizado

**Resposta correta: B.**

**Explicação:** A função chama `client.messages.create()` e retorna `message.content[0].text`.

---

### 9. Qual é um risco de reenviar um histórico muito longo?

A) Claude deixa de aceitar mensagens com role `"user"`

B) O custo e a latência podem aumentar

C) A API key é automaticamente deletada

D) O modelo passa a responder apenas em código

**Resposta correta: B.**

**Explicação:** Histórico longo consome mais tokens de entrada, o que pode aumentar custo e tempo de resposta.

---

### 10. Qual estratégia pode ajudar em conversas muito longas?

A) Nunca salvar mensagens do assistente

B) Resumir mensagens antigas ou manter apenas o contexto relevante

C) Colocar a API key dentro da lista `messages`

D) Enviar sempre apenas a palavra “continue”

**Resposta correta: B.**

**Explicação:** Resumir ou selecionar partes relevantes do histórico ajuda a controlar tokens, custo e limite de contexto.