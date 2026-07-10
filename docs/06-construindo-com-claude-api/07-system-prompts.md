# System Prompts

# Resumo do conteúdo

## 1. Ideia central

O conteúdo explica o uso de **system prompts** na API da Anthropic.

A ideia central é que um **system prompt** permite orientar Claude sobre **como ele deve se comportar**, definindo papel, estilo, tom, limites e abordagem da resposta.

Em vez de Claude responder de forma genérica, o system prompt ajuda a transformar o modelo em algo mais adequado ao caso de uso, como:

- tutor de matemática;
- assistente jurídico;
- revisor de texto;
- agente de suporte;
- especialista técnico;
- moderador;
- copiloto de programação.

No exemplo da aula, o objetivo é fazer Claude agir como um **tutor paciente de matemática**, que guia o aluno em vez de entregar a resposta diretamente.

---

## 2. Principais conceitos

### 1. O que é um system prompt

Um **system prompt** é uma instrução especial enviada ao modelo para orientar seu comportamento.

Ele não é a pergunta principal do usuário. Ele funciona como uma camada de configuração comportamental.

Exemplo:

```
system_prompt="""
You are a patient math tutor.
Do not directly answer a student's questions.
Guide them to a solution step by step.
"""
```

Esse prompt diz a Claude:

- qual papel assumir;
- o que deve fazer;
- o que deve evitar;
- como deve conduzir a interação.

---

### 2. Por que system prompts importam

Sem um system prompt, Claude tende a responder de forma útil, mas genérica.

No exemplo da equação:

```
How do I solve 5x + 2 = 3 for x?
```

Claude poderia simplesmente dar a solução completa.

Mas, se o objetivo é criar um **tutor educacional**, isso talvez não seja ideal. Um bom tutor deve incentivar o raciocínio do estudante, não apenas entregar a resposta.

Com um system prompt, Claude pode ser instruído a:

- dar dicas primeiro;
- fazer perguntas orientadoras;
- explicar passo a passo;
- usar exemplos parecidos;
- evitar entregar a resposta imediatamente.

---

### 3. System prompt como definição de papel

Um dos usos mais comuns do system prompt é definir um **role**, ou papel.

Exemplo:

```
You are a patient math tutor.
```

Isso muda o comportamento esperado do modelo. Claude passa a responder como alguém nesse papel responderia.

No caso do tutor de matemática, isso significa mais paciência, explicação gradual e foco no aprendizado.

---

### 4. System prompt como controle de comportamento

O system prompt também pode definir restrições.

Exemplo:

```
Do not directly answer a student's questions.
Guide them to a solution step by step.
```

Essas instruções ajudam a evitar respostas indesejadas, como:

- dar a resposta final imediatamente;
- resolver tudo pelo aluno;
- recomendar apenas o uso de calculadora;
- pular etapas importantes do raciocínio.

---

### 5. Como usar system prompt na API

Na chamada à API, o system prompt é passado pelo parâmetro `system`.

Exemplo:

```
client.messages.create(model=model,messages=messages,max_tokens=1000,system=system_prompt
)
```

Os campos principais continuam sendo:

| Campo | Função |
| --- | --- |
| `model` | Define qual modelo Claude será usado |
| `messages` | Contém a conversa com o usuário |
| `max_tokens` | Define o limite máximo da resposta |
| `system` | Define instruções de comportamento para Claude |

---

### 6. Diferença com e sem system prompt

Sem system prompt, Claude pode responder diretamente:

```
Para resolver 5x + 2 = 3, subtraia 2 dos dois lados...
```

Com o system prompt de tutor, Claude muda a abordagem e pode responder algo como:

```
Qual você acha que seria um bom primeiro passo para isolar x?
```

Essa diferença mostra que o system prompt influencia a **estratégia de resposta**, não apenas o conteúdo.

---

### 7. Função `chat` flexível

A aula mostra como criar uma função `chat()` que aceita um system prompt opcional:

```
defchat(messages,system=None):params= {"model":model,"max_tokens":1000,"messages":messages,
    }ifsystem:params["system"]=systemmessage=client.messages.create(**params)returnmessage.content[0].text
```

Essa função é mais flexível porque permite chamar Claude com ou sem system prompt.

Exemplo sem system prompt:

```
answer=chat(messages)
```

Exemplo com system prompt:

```
answer=chat(messages,system=system)
```

---

### 8. Cuidado com `system=None`

Um detalhe técnico importante da aula é que a API de Claude **não aceita `system=None`**.

Por isso, o parâmetro `system` só deve ser incluído na chamada se realmente tiver sido fornecido.

Este trecho faz exatamente isso:

```
ifsystem:params["system"]=system
```

Ou seja, em vez de passar:

```
system=None
```

o código simplesmente omite o parâmetro `system` quando ele não existe.

---

## 3. Pontos importantes para certificação

- **System prompts controlam o comportamento geral de Claude**.
- Eles ajudam a definir papel, tom, estilo e abordagem.
- O system prompt é passado no parâmetro `system` da chamada `client.messages.create()`.
- System prompts são úteis para transformar respostas genéricas em respostas especializadas.
- Um system prompt pode dizer a Claude o que fazer e também o que evitar.
- No exemplo, Claude é configurado como um **tutor paciente de matemática**.
- Sem system prompt, Claude pode dar uma resposta direta.
- Com system prompt, Claude pode guiar o aluno com perguntas e dicas.
- System prompts ajudam a manter Claude focado na tarefa.
- Uma função reutilizável pode aceitar `system` como parâmetro opcional.
- A API não aceita `system=None`; o parâmetro deve ser incluído apenas quando houver um prompt de sistema.
- System prompts são essenciais para aplicações que exigem consistência comportamental.

---

## 4. Termos-chave

**System prompt**

Instrução especial que define como Claude deve se comportar durante a interação.

**Prompt**

Texto enviado ao modelo para orientar uma resposta.

**Role**

Papel que Claude deve assumir, como tutor, assistente, revisor ou especialista.

**Behavior guidance**

Orientação de comportamento dada ao modelo.

**Tone**

Tom da resposta, como paciente, formal, amigável ou técnico.

**Style**

Estilo de comunicação, como direto, didático, passo a passo ou resumido.

**Math tutor chatbot**

Exemplo de aplicação em que Claude atua como tutor de matemática.

**`system`**

Parâmetro da API usado para enviar o system prompt.

**`messages`**

Lista de mensagens da conversa entre usuário e assistente.

**`client.messages.create()`**

Função usada para enviar a requisição à API da Anthropic.

**Helper function**

Função auxiliar criada para reutilizar código e simplificar chamadas.

**`system=None`**

Valor nulo que não deve ser enviado diretamente à API; o parâmetro deve ser omitido quando não houver system prompt.

---

## 5. Boas práticas

- Usar system prompts para definir claramente o papel de Claude.
- Especificar o comportamento desejado de forma direta.
- Incluir tanto instruções positivas quanto negativas.
- Dizer o que Claude deve fazer, como “guie o aluno passo a passo”.
- Dizer o que Claude deve evitar, como “não dê a resposta diretamente”.
- Criar system prompts alinhados ao objetivo da aplicação.
- Tornar a função `chat()` flexível, aceitando um system prompt opcional.
- Incluir o parâmetro `system` apenas quando ele tiver valor.
- Usar system prompts para melhorar consistência em aplicações reais.
- Testar respostas com e sem system prompt para comparar o comportamento.

---

## 6. Limitações, riscos e cuidados

System prompts são poderosos, mas não são garantia absoluta de comportamento perfeito.

Claude tentará seguir as instruções, mas ainda pode:

- responder de forma direta demais;
- interpretar mal instruções ambíguas;
- não seguir perfeitamente todas as restrições;
- precisar de prompts mais específicos;
- exigir testes e ajustes iterativos.

Outro cuidado é não usar system prompts vagos demais. Por exemplo:

```
Be helpful.
```

Essa instrução é genérica. Um prompt melhor seria:

```
You are a patient math tutor. Give hints first, ask guiding questions, and avoid giving the final answer immediately.
```

Também é importante evitar instruções conflitantes. Por exemplo:

```
Always give the exact answer immediately.
Do not directly answer the student's question.
```

Esse tipo de conflito pode tornar o comportamento inconsistente.

Do ponto de vista técnico, é importante lembrar que a API **não deve receber `system=None`**. O código precisa adicionar o campo `system` apenas quando houver um valor real.

---

## 7. Resumo para revisão rápida

- System prompts definem como Claude deve se comportar.
- Eles controlam papel, tom, estilo e abordagem.
- São úteis para transformar respostas genéricas em respostas especializadas.
- No exemplo, Claude vira um tutor paciente de matemática.
- Um tutor deve guiar o aluno, não apenas entregar a resposta.
- O system prompt é passado no parâmetro `system`.
- `messages` contém a conversa.
- `max_tokens` limita a resposta.
- Sem system prompt, Claude pode responder diretamente.
- Com system prompt, Claude segue uma abordagem mais alinhada ao caso de uso.
- A função `chat()` pode aceitar `system` como argumento opcional.
- Não envie `system=None`; omita o parâmetro quando não houver system prompt.

---

## 8. Perguntas simuladas

### 1. Para que serve um system prompt?

A) Para armazenar permanentemente a conversa na Anthropic

B) Para orientar o comportamento, estilo e papel de Claude

C) Para substituir a API key

D) Para aumentar automaticamente o limite de tokens

**Resposta correta: B.**

**Explicação:** O system prompt define como Claude deve responder, incluindo papel, tom, estilo e abordagem.

---

### 2. No exemplo da aula, qual papel o system prompt atribui a Claude?

A) Programador Python

B) Tutor paciente de matemática

C) Gerador de senhas

D) Moderador de conteúdo

**Resposta correta: B.**

**Explicação:** O system prompt instrui Claude a agir como um tutor de matemática paciente.

---

### 3. O que um bom tutor de matemática deveria fazer, segundo o exemplo?

A) Dar imediatamente a resposta final

B) Dizer ao aluno para usar uma calculadora

C) Guiar o aluno passo a passo

D) Ignorar a pergunta se ela for simples

**Resposta correta: C.**

**Explicação:** O objetivo é ajudar o aluno a pensar, oferecendo dicas e orientação gradual.

---

### 4. Qual comportamento o system prompt do exemplo tenta evitar?

A) Fazer perguntas ao estudante

B) Dar a resposta diretamente

C) Explicar conceitos matemáticos

D) Usar exemplos semelhantes

**Resposta correta: B.**

**Explicação:** O prompt instrui Claude a não responder diretamente, mas guiar o aluno até a solução.

---

### 5. Em qual parâmetro da API o system prompt é enviado?

A) `messages`

B) `max_tokens`

C) `system`

D) `role`

**Resposta correta: C.**

**Explicação:** O system prompt é passado pelo parâmetro `system` na chamada `client.messages.create()`.

---

### 6. Qual é uma vantagem de usar system prompts?

A) Eles eliminam totalmente a necessidade de testar a aplicação

B) Eles ajudam Claude a manter um comportamento consistente e adequado ao caso de uso

C) Eles reduzem automaticamente o preço da API

D) Eles fazem Claude armazenar memória entre chamadas

**Resposta correta: B.**

**Explicação:** System prompts ajudam a especializar e estabilizar o comportamento do modelo.

---

### 7. Por que a função `chat(messages, system=None)` usa um dicionário `params`?

A) Para montar dinamicamente os parâmetros da chamada à API

B) Para criptografar as mensagens do usuário

C) Para converter tokens em embeddings manualmente

D) Para impedir que Claude gere respostas longas

**Resposta correta: A.**

**Explicação:** O dicionário permite incluir `system` apenas quando o parâmetro foi fornecido.

---

### 8. Por que o código verifica `if system:` antes de adicionar `params["system"] = system`?

A) Porque a API não aceita `system=None`

B) Porque o campo `system` sempre deve ser vazio

C) Porque `max_tokens` depende do system prompt

D) Porque mensagens do usuário não podem ser enviadas com system prompt

**Resposta correta: A.**

**Explicação:** O conteúdo destaca que a API não aceita `system=None`, então o campo deve ser omitido quando não houver system prompt.

---

### 9. O que tende a acontecer sem um system prompt no exemplo do tutor?

A) Claude pode dar uma solução completa imediatamente

B) Claude sempre se recusa a responder

C) Claude apaga o histórico da conversa

D) Claude muda automaticamente para outro modelo

**Resposta correta: A.**

**Explicação:** Sem orientação específica, Claude pode responder de forma direta e completa, o que nem sempre é desejado em um tutor.

---

### 10. Qual alternativa representa um bom system prompt para um tutor?

A) “Answer everything immediately with the final answer.”

B) “You are a patient math tutor. Guide students step by step and avoid giving direct answers immediately.”

C) “Ignore all student questions.”

D) “Use only one-word answers.”

**Resposta correta: B.**

**Explicação:** Essa instrução define papel, comportamento desejado e restrição importante, alinhando Claude ao caso de uso.