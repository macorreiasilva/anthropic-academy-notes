# Fazer um pedido

# Resumo do conteúdo

## 1. Ideia central

O conteúdo ensina como fazer a **primeira requisição à API da Anthropic usando Python**.

A ideia central é que, para chamar Claude via API, você precisa:

1. instalar as dependências;
2. armazenar a API key com segurança;
3. criar um cliente da Anthropic;
4. usar `client.messages.create()`;
5. extrair o texto gerado da resposta.

O fluxo básico é:

**configurar ambiente → criar cliente → enviar mensagem → receber resposta → extrair texto**

---

## 2. Principais conceitos

### 1. Configuração do ambiente

Antes de chamar a API, é necessário instalar os pacotes usados no exemplo:

```python
%pip install anthropic python-dotenv
```

Esses pacotes têm funções diferentes:

| Pacote | Função |
| --- | --- |
| `anthropic` | SDK oficial para interagir com a API da Anthropic |
| `python-dotenv` | Permite carregar variáveis de ambiente a partir de um arquivo `.env` |

O exemplo usa um **Jupyter notebook**, por isso o comando aparece com `%pip`.

---

### 2. Armazenamento seguro da API key

A API key deve ser colocada em um arquivo `.env`, no mesmo diretório do notebook:

```
ANTHROPIC_API_KEY="your-api-key-here"
```

Isso evita colocar a chave diretamente no código.

Esse ponto é muito importante para segurança: a API key é uma credencial secreta e não deve ser compartilhada, publicada nem enviada para repositórios.

Também é recomendado adicionar `.env` ao arquivo `.gitignore`, para impedir que ele seja enviado acidentalmente ao GitHub ou a outro sistema de versionamento.

---

### 3. Carregando variáveis de ambiente

Depois de criar o `.env`, o código carrega as variáveis com:

```python
from dotenv import load_dotenv
load_dotenv()
```

Isso permite que o SDK encontre automaticamente a variável:

```
ANTHROPIC_API_KEY
```

Assim, não é necessário escrever a API key dentro do código Python.

---

### 4. Criando o cliente da API

Depois de carregar a variável de ambiente, o exemplo cria o cliente da Anthropic:

```python
from anthropic import Anthropic

client = Anthropic()
model = "claude-sonnet-4-0"
```

O objeto `client` é usado para enviar requisições à API.

O campo `model` define qual modelo Claude será usado. No exemplo, o modelo escolhido é:

```python
"claude-sonnet-4-0"
```

Em uma situação real, o nome do modelo deve corresponder a um modelo disponível na conta e na documentação atual da API.

---

### 5. A função `client.messages.create()`

O núcleo da chamada à API é a função:

```python
client.messages.create()
```

Ela envia uma mensagem para Claude e retorna uma resposta.

Essa função exige três parâmetros principais:

| Parâmetro | Função |
| --- | --- |
| `model` | Define qual modelo Claude será usado |
| `max_tokens` | Define o limite máximo de tokens de saída |
| `messages` | Contém a conversa enviada para Claude |

---

### 6. O papel de `max_tokens`

O parâmetro `max_tokens` é um **limite de segurança**, não uma meta.

Isso significa que, se você definir:

```python
max_tokens=1000
```

Claude pode responder com menos de 1000 tokens se a resposta adequada for curta.

Ele só será interrompido se tentar ultrapassar esse limite.

Portanto:

**`max_tokens` limita a resposta, mas não obriga Claude a usar todos os tokens.**

---

### 7. Estrutura de mensagens

As mensagens simulam uma conversa, parecida com um chat.

Cada mensagem é um dicionário com dois campos:

```python
{
    "role": "user",
    "content": "What is quantum computing? Answer in one sentence"
}
```

Os dois papéis principais são:

| Role | Significado |
| --- | --- |
| `"user"` | Mensagem enviada pelo usuário |
| `"assistant"` | Resposta gerada pelo Claude |

No primeiro exemplo, há apenas uma mensagem do usuário.

---

### 8. Fazendo a primeira requisição

O exemplo completo de chamada é:

```python
message = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=[
        {
            "role": "user",
            "content": "What is quantum computing? Answer in one sentence"
        }
    ]
)
```

Esse código envia uma pergunta para Claude:

> “What is quantum computing? Answer in one sentence”
> 

Claude processa a entrada e retorna um objeto de resposta com o texto gerado e metadados da chamada.

---

### 9. Extraindo a resposta

A resposta completa contém várias informações, mas normalmente o desenvolvedor quer apenas o texto gerado.

Para acessar o texto, usa-se:

```python
message.content[0].text
```

Esse comando acessa o primeiro bloco de conteúdo da resposta e extrai o texto.

---

## 3. Pontos importantes para certificação

- Para usar a API da Anthropic em Python, instala-se o pacote `anthropic`.
- O pacote `python-dotenv` permite carregar variáveis de ambiente a partir de um arquivo `.env`.
- A API key deve ficar em uma variável chamada `ANTHROPIC_API_KEY`.
- A API key não deve ser escrita diretamente no código.
- O arquivo `.env` deve ser incluído no `.gitignore`.
- O cliente é criado com `client = Anthropic()`.
- A chamada principal é feita com `client.messages.create()`.
- Os três parâmetros essenciais são: `model`, `max_tokens` e `messages`.
- `max_tokens` é um limite máximo, não uma meta de tamanho.
- As mensagens usam papéis como `"user"` e `"assistant"`.
- O conteúdo da mensagem fica no campo `content`.
- A resposta textual pode ser acessada com `message.content[0].text`.
- O objeto de resposta contém texto gerado e metadados.

---

## 4. Termos-chave

**SDK**

Biblioteca que facilita o uso da API sem precisar montar manualmente todas as chamadas HTTP.

**Anthropic SDK**

Pacote Python usado para interagir com a API da Anthropic.

**Jupyter notebook**

Ambiente interativo usado para executar código em células.

**`python-dotenv`**

Pacote que carrega variáveis de ambiente a partir de um arquivo `.env`.

**`.env`**

Arquivo usado para armazenar variáveis sensíveis, como API keys.

**`.gitignore`**

Arquivo que define quais arquivos ou pastas não devem ser enviados ao controle de versão.

**API key**

Chave secreta usada para autenticar chamadas à API.

**Environment variable**

Variável de ambiente usada para guardar configurações fora do código.

**`ANTHROPIC_API_KEY`**

Nome da variável de ambiente esperada pelo SDK para autenticação.

**Client**

Objeto usado para se comunicar com a API.

**`client.messages.create()`**

Função usada para enviar uma mensagem para Claude e receber uma resposta.

**`model`**

Parâmetro que define qual modelo Claude será usado.

**`max_tokens`**

Limite máximo de tokens que Claude pode gerar na resposta.

**`messages`**

Lista de mensagens que representa a conversa enviada ao modelo.

**`role`**

Campo que indica quem escreveu a mensagem: usuário ou assistente.

**`content`**

Texto da mensagem enviada ou recebida.

**Response object**

Objeto retornado pela API contendo a resposta gerada e metadados.

---

## 5. Boas práticas

- Armazenar a API key no arquivo `.env`, não diretamente no código.
- Adicionar `.env` ao `.gitignore`.
- Usar `load_dotenv()` para carregar a chave de forma segura.
- Usar o SDK oficial para simplificar chamadas à API.
- Definir `max_tokens` de acordo com o tamanho esperado da resposta.
- Lembrar que `max_tokens` é um limite de segurança, não um objetivo.
- Começar com prompts simples para validar a integração.
- Extrair apenas o texto necessário da resposta quando for exibir ao usuário.
- Verificar se o modelo informado em `model` existe e está disponível.
- Não compartilhar notebooks ou arquivos contendo chaves secretas.

---

## 6. Limitações, riscos e cuidados

O principal risco é **expor a API key**.

Se a chave for colocada diretamente no código, ela pode ser enviada por acidente para um repositório, compartilhada em um notebook ou vazada em prints e logs.

Outro cuidado é com o parâmetro `max_tokens`. Se ele for muito baixo, a resposta pode ser cortada. Se for muito alto, pode aumentar o custo máximo possível da chamada.

Também é importante entender que o objeto retornado pela API não contém apenas texto puro. Ele é uma estrutura com vários campos, por isso é necessário acessar corretamente:

```python
message.content[0].text
```

Por fim, o nome do modelo usado no curso, como `"claude-sonnet-4-0"`, deve ser tratado como exemplo didático. Em projetos reais, é necessário confirmar o identificador correto do modelo na documentação ou no ambiente usado.

---

## 7. Resumo para revisão rápida

- Instale os pacotes: `anthropic` e `python-dotenv`.
- Crie um arquivo `.env`.
- Salve a chave como `ANTHROPIC_API_KEY`.
- Nunca coloque a API key diretamente no código.
- Adicione `.env` ao `.gitignore`.
- Use `load_dotenv()` para carregar a chave.
- Crie o cliente com `client = Anthropic()`.
- Use `client.messages.create()` para chamar Claude.
- Parâmetros principais: `model`, `max_tokens`, `messages`.
- `max_tokens` é limite máximo, não meta.
- Mensagens usam `role` e `content`.
- `"user"` representa a entrada humana.
- `"assistant"` representa respostas do Claude.
- Extraia o texto com `message.content[0].text`.

---

## 8. Perguntas simuladas

### 1. Qual pacote Python é usado para interagir com a API da Anthropic?

A) `requests-html`

B) `anthropic`

C) `claude-env`

D) `openai-tools`

**Resposta correta: B.**

**Explicação:** O pacote `anthropic` é o SDK usado para fazer chamadas à API da Anthropic em Python.

---

### 2. Para que serve o pacote `python-dotenv`?

A) Para gerar embeddings automaticamente

B) Para carregar variáveis de ambiente a partir de um arquivo `.env`

C) Para escolher o melhor modelo Claude

D) Para converter respostas em HTML

**Resposta correta: B.**

**Explicação:** `python-dotenv` permite carregar variáveis definidas em um arquivo `.env`, como a API key.

---

### 3. Qual é a forma recomendada de armazenar a API key no exemplo?

A) Escrevê-la diretamente dentro do código Python

B) Colocá-la em um arquivo `.env`

C) Enviá-la em cada prompt para Claude

D) Guardá-la no campo `messages`

**Resposta correta: B.**

**Explicação:** O arquivo `.env` permite manter a chave fora do código-fonte.

---

### 4. Por que o arquivo `.env` deve ser adicionado ao `.gitignore`?

A) Para aumentar a velocidade da API

B) Para evitar que a API key seja enviada ao controle de versão

C) Para permitir que Claude gere respostas maiores

D) Para transformar tokens em embeddings

**Resposta correta: B.**

**Explicação:** O `.gitignore` impede que arquivos sensíveis, como `.env`, sejam enviados acidentalmente para repositórios.

---

### 5. Qual função é usada para carregar as variáveis do arquivo `.env`?

A) `create_client()`

B) `load_dotenv()`

C) `read_api_key()`

D) `messages.load()`

**Resposta correta: B.**

**Explicação:** `load_dotenv()` carrega as variáveis definidas no arquivo `.env`.

---

### 6. Qual objeto é criado para interagir com a API?

A) `ClaudeServer()`

B) `Anthropic()`

C) `MessageReader()`

D) `TokenCounter()`

**Resposta correta: B.**

**Explicação:** O cliente da API é criado com `client = Anthropic()`.

---

### 7. Qual função é o núcleo para fazer chamadas de mensagens à API?

A) `client.messages.create()`

B) `client.tokens.generate()`

C) `client.response.read()`

D) `client.models.run_local()`

**Resposta correta: A.**

**Explicação:** `client.messages.create()` envia mensagens ao modelo e retorna uma resposta.

---

### 8. Quais são três parâmetros essenciais de `client.messages.create()`?

A) `model`, `max_tokens` e `messages`

B) `username`, `password` e `email`

C) `temperature`, `browser` e `workspace`

D) `notebook`, `gitignore` e `dotenv`

**Resposta correta: A.**

**Explicação:** Esses três parâmetros definem o modelo, o limite da resposta e a conversa enviada ao Claude.

---

### 9. O que significa dizer que `max_tokens` é um limite, não uma meta?

A) Claude sempre gera exatamente esse número de tokens

B) Claude pode gerar menos tokens e só para se atingir o máximo

C) Claude ignora completamente esse valor

D) Claude usa esse valor para escolher a API key

**Resposta correta: B.**

**Explicação:** `max_tokens` define o teto da resposta. Claude não tenta usar todos os tokens, apenas para se atingir o limite.

---

### 10. Como se extrai o texto gerado da resposta no exemplo?

A) `message.text`

B) `message.output`

C) `message.content[0].text`

D) `client.response.text()`

**Resposta correta: C.**

**Explicação:** A resposta vem em um objeto estruturado, e o texto principal é acessado por `message.content[0].text`.