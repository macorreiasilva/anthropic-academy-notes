# Exercício de Chat Bot

### Crie um chatbot usando as três funções auxiliares que acabamos de preparar.
1. Solicite ao usuário que insira algo usando a função integrada "input"
2. Adicione a entrada a uma lista de mensagens
3. Chame a API
4. Adicione o texto gerado à lista de mensagens
5. Exiba o texto gerado
6. Repita a partir do passo 1
---
```bash
%pip install anthropic python-dotenv
```
---
```python
from dotenv import load_dotenv
from sympy import content
load_dotenv()  # take environment variables from .env.

from anthropic import Anthropic

client = Anthropic()
model = "claude-haiku-4-5-20251001"

def add_user_message(messages, text):
    user_message = {"role": "user", "content": text}
    messages.append(user_message)

def add_assistant_message(messages, text):
    assistant_message = {"role": "assistant", "content": text}
    messages.append(assistant_message)

def chat(messages):
    message = client.messages.create(
        model=model,
        max_tokens=1024,
        messages=messages
    )
    return message.content[0].text

# Start with an empty message list
messages = []

# Add the initial user question
add_user_message(messages, "What is the capital of Brazil?")

# Get Claude's response
answer = chat(messages)

# Add Claude's response to the conversation history
add_assistant_message(messages, answer)

# Add a follow-up question
add_user_message(messages, "What is the population of that city?")

# Get the follow-up response with full context
final_answer = chat(messages)

final_answer
```

