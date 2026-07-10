# Acessando a API

# Resumo do conteúdo

## 1. Ideia central

O conteúdo explica o **fluxo completo de uma chamada à API do Claude**, desde o momento em que o usuário envia uma pergunta no aplicativo até a resposta gerada pelo modelo aparecer na interface.

A ideia mais importante é:

**O cliente nunca deve chamar diretamente a API da Anthropic.**

A chamada deve passar pelo **servidor da aplicação**, porque é ali que a **API key secreta** deve ser armazenada com segurança.

O fluxo geral é:

**Cliente → Seu servidor → API da Anthropic → Seu servidor → Cliente**

---

## 2. Principais conceitos

### 1. Request to Server

O usuário interage com o aplicativo, por exemplo digitando:

> “What is quantum computing?”
> 

Essa pergunta é enviada primeiro para o **servidor da sua aplicação**, não diretamente para a Anthropic.

Isso é importante porque chamadas para a Anthropic exigem uma **API key**, que deve permanecer secreta. Se a chave fosse colocada no front-end, qualquer pessoa poderia inspecionar o código da aplicação e roubá-la.

---

### 2. Request to Anthropic API

Depois que o servidor recebe a pergunta do usuário, ele monta uma requisição para a API da Anthropic.

Essa requisição pode ser feita usando:

- o **SDK oficial da Anthropic**;
- uma chamada HTTP direta.

A requisição normalmente contém campos como:

| Campo | Função |
| --- | --- |
| **API Key** | Identifica e autentica quem está chamando a API |
| **Model** | Define qual modelo Claude será usado |
| **Messages** | Contém as mensagens da conversa |
| **Max Tokens** | Define o limite máximo de tokens que o modelo pode gerar |

A mensagem do usuário entra em uma lista de mensagens com o papel `"user"`.

Exemplo conceitual:

```json
{
  "model": "claude-sonnet-...",
  "max_tokens": 1024,
  "messages": [
    {
      "role": "user",
      "content": "What is quantum computing?"
    }
  ]
}
```

---

### 3. Model Processing

Essa é a etapa em que o modelo processa a entrada e gera uma resposta.

Ela pode ser entendida em quatro subetapas principais:

### a) Tokenization

O texto é dividido em **tokens**.

Tokens podem ser:

- palavras inteiras;
- partes de palavras;
- pontuação;
- espaços ou fragmentos especiais.

Por exemplo, uma pergunta como “What is quantum computing?” é quebrada em unidades menores para que o modelo consiga processá-la matematicamente.

A palavra **“quantum”** pode ter diferentes sentidos dependendo do contexto. Por isso, o modelo precisa analisar as palavras ao redor.

---

### b) Embedding

Cada token é transformado em um **vetor numérico**.

Esse vetor representa características semânticas do token, ou seja, uma representação matemática do seu significado.

Em termos simples:

**Texto → Tokens → Números**

O modelo não “entende” texto como humanos; ele opera sobre representações numéricas.

---

### c) Contextualization

Nessa etapa, os tokens passam a ser interpretados em relação uns aos outros.

O significado de cada token é ajustado com base no contexto.

Por exemplo, em:

> “What is quantum computing?”
> 

o modelo entende que **“quantum”** está relacionado a **computação quântica**, não apenas a física em sentido amplo.

Essa contextualização é essencial para que o modelo não interprete palavras isoladamente.

---

### d) Generation

Depois de processar o contexto, o modelo começa a gerar a resposta **token por token**.

A cada passo, ele calcula probabilidades para o próximo token possível.

Exemplo simplificado:

| Próximo token possível | Probabilidade |
| --- | --- |
| “Quantum” | 30% |
| “Great” | 23% |
| “It” | 15% |
| “The” | 10% |

O modelo escolhe um token e repete o processo até formar uma resposta completa.

A geração para quando ocorre uma destas condições:

1. O modelo atinge o limite de **max tokens**.
2. O modelo encontra uma **stop sequence**.
3. O modelo gera um token de fim de sequência, chamado **EOS**.

---

### 4. Response to Server

Depois de gerar a resposta, a API da Anthropic devolve o resultado ao servidor da aplicação.

A resposta normalmente inclui:

| Campo | Significado |
| --- | --- |
| **Message** | A resposta gerada pelo modelo |
| **Usage** | Quantidade de tokens usados na entrada e na saída |
| **Stop Reason** | Motivo pelo qual a geração foi interrompida |

A resposta do modelo vem como uma mensagem com papel `"assistant"`.

---

### 5. Response to Client

Por fim, o servidor envia a resposta para o aplicativo do usuário.

O app exibe o texto gerado na tela.

Essa etapa fecha o ciclo:

**Usuário envia pergunta → Claude gera resposta → usuário vê a resposta no app**

---

## 3. Pontos importantes para certificação

- A aplicação cliente **não deve chamar diretamente a API da Anthropic**.
- A **API key deve ficar no servidor**, nunca no front-end.
- O servidor atua como intermediário entre o app e a Anthropic.
- A requisição para a API normalmente inclui: **API key, model, messages e max tokens**.
- A mensagem do usuário usa o papel `"user"`.
- A resposta do modelo usa o papel `"assistant"`.
- O texto é processado primeiro por **tokenization**.
- Tokens são convertidos em vetores numéricos por meio de **embeddings**.
- O modelo usa o contexto para interpretar corretamente o significado dos tokens.
- A geração acontece **token por token**, com base em probabilidades.
- A resposta inclui metadados importantes, como **usage** e **stop reason**.
- O campo **max tokens** limita o tamanho máximo da resposta gerada.
- A geração pode parar por limite de tokens, stop sequence ou fim natural da sequência.

---

## 4. Termos-chave

**API**

Interface que permite que uma aplicação se comunique com outro sistema, neste caso a API da Anthropic.

**API key**

Chave secreta usada para autenticar chamadas à API. Deve ser protegida no servidor.

**Cliente**

Aplicativo usado pelo usuário, como um app web ou mobile.

**Servidor**

Camada intermediária que recebe a solicitação do cliente, chama a API da Anthropic e devolve a resposta.

**SDK**

Biblioteca oficial que facilita a integração com a API.

**Model**

Campo que define qual modelo Claude será usado na chamada.

**Messages**

Lista de mensagens da conversa enviada ao modelo.

**Role**

Papel da mensagem, como `"user"` para o usuário e `"assistant"` para o modelo.

**Max tokens**

Limite máximo de tokens que o modelo pode gerar na resposta.

**Tokenization**

Processo de quebrar o texto em tokens.

**Token**

Unidade de texto processada pelo modelo. Pode ser uma palavra, parte de palavra ou símbolo.

**Embedding**

Representação numérica de um token.

**Contextualization**

Processo pelo qual o modelo ajusta o significado de cada token com base no contexto.

**Generation**

Processo de geração da resposta token por token.

**Stop sequence**

Sequência configurada para interromper a geração quando aparece na saída.

**EOS**

End of Sequence, token que indica o fim natural da resposta.

**Usage**

Metadado que mostra quantos tokens foram usados na entrada e na saída.

**Stop reason**

Metadado que explica por que o modelo parou de gerar texto.

---

## 5. Boas práticas

- Nunca expor a **API key** no código do cliente.
- Fazer chamadas para a Anthropic a partir do **servidor da aplicação**.
- Usar o SDK oficial quando possível, pois ele simplifica a integração.
- Definir corretamente o campo **model** de acordo com a tarefa.
- Configurar **max tokens** para controlar tamanho da resposta, custo e latência.
- Monitorar o campo **usage** para acompanhar consumo de tokens.
- Verificar o **stop reason** para entender se a resposta terminou naturalmente ou foi cortada.
- Usar **stop sequences** quando for necessário controlar onde a geração deve parar.
- Separar claramente mensagens do usuário e do assistente usando os papéis `"user"` e `"assistant"`.
- Proteger a infraestrutura contra abuso, vazamento de chave e chamadas não autorizadas.

---

## 6. Limitações, riscos e cuidados

O principal risco apresentado é o **vazamento da API key**.

Se a chave for colocada no front-end, ela pode ser exposta por meio de:

- inspeção do código-fonte;
- ferramentas de desenvolvedor do navegador;
- interceptação de requisições;
- engenharia reversa de apps mobile.

Outro cuidado importante é o uso de **max tokens**. Se o limite for muito baixo, a resposta pode ser cortada. Se for muito alto, pode aumentar custo e latência.

Também é importante lembrar que o modelo gera texto com base em **probabilidades**, não por consulta direta a uma base de verdades garantidas. Por isso, respostas podem exigir validação, especialmente em contextos críticos.

A etapa de contextualização melhora a interpretação do texto, mas não elimina completamente ambiguidades. Um prompt mal formulado pode gerar respostas incompletas, genéricas ou incorretas.

---

## 7. Resumo para revisão rápida

- O app envia a pergunta para o **seu servidor**.
- O servidor chama a **API da Anthropic**.
- A **API key nunca deve ficar no cliente**.
- A requisição inclui **model, messages, max tokens e autenticação**.
- O texto é quebrado em **tokens**.
- Tokens viram **embeddings**, ou vetores numéricos.
- O modelo usa o contexto para interpretar os tokens.
- A resposta é gerada **um token por vez**.
- A geração para por **max tokens, stop sequence ou EOS**.
- A API retorna a resposta com metadados como **usage** e **stop reason**.
- O servidor envia o texto final de volta para o cliente.
- O fluxo seguro é: **cliente → servidor → Anthropic → servidor → cliente**.

---

## 8. Perguntas simuladas

### 1. Por que o cliente não deve chamar diretamente a API da Anthropic?

A) Porque o navegador não consegue enviar requisições HTTP

B) Porque a API só funciona em servidores Linux

C) Porque a API key secreta poderia ficar exposta no front-end

D) Porque modelos Claude não aceitam mensagens de usuários finais

**Resposta correta: C.**

**Explicação:** A API key é uma credencial secreta. Se ela ficar no front-end, pode ser roubada por qualquer pessoa que inspecione o app.

---

### 2. Qual é o papel do servidor no fluxo de chamada à API?

A) Substituir completamente o modelo Claude

B) Intermediar a comunicação entre o cliente e a API da Anthropic

C) Fazer tokenization manualmente antes da API

D) Exibir diretamente a resposta sem chamar a Anthropic

**Resposta correta: B.**

**Explicação:** O servidor recebe a solicitação do cliente, adiciona a autenticação necessária, chama a API da Anthropic e retorna a resposta ao app.

---

### 3. Qual campo define o limite máximo de tokens que o modelo pode gerar?

A) `messages`

B) `role`

C) `max_tokens`

D) `usage`

**Resposta correta: C.**

**Explicação:** `max_tokens` define o tamanho máximo da saída gerada pelo modelo.

---

### 4. Em uma chamada à API, qual papel representa a mensagem enviada pelo usuário?

A) `"assistant"`

B) `"system"`

C) `"user"`

D) `"server"`

**Resposta correta: C.**

**Explicação:** Mensagens do usuário são enviadas com o papel `"user"`.

---

### 5. O que acontece durante a tokenization?

A) O texto é criptografado

B) O texto é dividido em unidades menores chamadas tokens

C) O modelo escolhe o servidor mais próximo

D) A resposta final é exibida ao usuário

**Resposta correta: B.**

**Explicação:** Tokenization é o processo de quebrar o texto em tokens para que o modelo consiga processá-lo.

---

### 6. O que são embeddings?

A) Mensagens finais exibidas no aplicativo

B) Vetores numéricos que representam tokens

C) Chaves secretas de autenticação

D) Limites de custo da API

**Resposta correta: B.**

**Explicação:** Embeddings são representações numéricas que permitem ao modelo operar matematicamente sobre o texto.

---

### 7. O que significa contextualization no processamento do modelo?

A) Armazenar permanentemente a conversa do usuário

B) Ajustar o significado dos tokens com base nos tokens ao redor

C) Traduzir automaticamente todas as mensagens

D) Reduzir o número de tokens da resposta

**Resposta correta: B.**

**Explicação:** Contextualization permite que o modelo interprete palavras de acordo com o contexto em que aparecem.

---

### 8. Como o modelo gera uma resposta?

A) Ele copia uma resposta pronta de um banco de dados

B) Ele gera todos os parágrafos ao mesmo tempo

C) Ele escolhe tokens sucessivos com base em probabilidades

D) Ele envia a pergunta para outro usuário responder

**Resposta correta: C.**

**Explicação:** Modelos de linguagem geram texto token por token, calculando probabilidades para o próximo token.

---

### 9. Qual destes pode ser um motivo para a geração parar?

A) Atingir o limite de `max_tokens`

B) O usuário fechar o navegador

C) O servidor trocar de IP

D) A API key mudar de cor

**Resposta correta: A.**

**Explicação:** A geração pode parar ao atingir `max_tokens`, encontrar uma stop sequence ou gerar um token EOS.

---

### 10. Para que serve o campo `usage` na resposta da API?

A) Para indicar o número de tokens usados na entrada e na saída

B) Para definir o modelo escolhido

C) Para armazenar a API key

D) Para substituir a mensagem do usuário

**Resposta correta: A.**

**Explicação:** `usage` mostra o consumo de tokens, informação importante para custo, monitoramento e otimização.

---

### 11. Qual é a ordem correta do fluxo de uma chamada à API do Claude?

A) Cliente → Anthropic API → servidor → cliente

B) Cliente → servidor → Anthropic API → servidor → cliente

C) Anthropic API → cliente → servidor → cliente

D) Servidor → cliente → Anthropic API → servidor

**Resposta correta: B.**

**Explicação:** O fluxo seguro passa pelo servidor da aplicação antes de chegar à API da Anthropic.

---

### 12. Por que não se deve chamar a API da Anthropic diretamente do front-end?

A) Porque o front-end não consegue processar JSON

B) Porque a API key secreta ficaria exposta

C) Porque Claude só aceita chamadas de apps mobile

D) Porque o modelo não funciona com navegadores

**Resposta correta: B.**

**Explicação:** A API key é uma credencial sensível. Se estiver no cliente, pode ser roubada.

---

### 13. Qual campo define o modelo Claude que será usado?

A) `usage`

B) `model`

C) `stop_reason`

D) `embedding`

**Resposta correta: B.**

**Explicação:** O campo `model` especifica qual modelo será chamado.

---

### 14. Qual campo limita a quantidade de tokens gerados na resposta?

A) `max_tokens`

B) `messages`

C) `api_key`

D) `temperature`

**Resposta correta: A.**

**Explicação:** `max_tokens` define o limite máximo de tokens de saída.

---

### 15. O que acontece na etapa de tokenization?

A) A API key é validada

B) O texto é dividido em tokens

C) O servidor envia a resposta ao cliente

D) O modelo escolhe o SDK correto

**Resposta correta: B.**

**Explicação:** Tokenization é o processo de quebrar o texto em unidades menores chamadas tokens.

---

### 16. O que é um embedding?

A) Uma resposta final exibida na interface

B) Uma representação numérica de um token

C) Uma chave secreta usada para autenticação

D) Uma sequência de parada da geração

**Resposta correta: B.**

**Explicação:** Embeddings representam tokens como listas de números que capturam significado semântico.

---

### 17. Por que a contextualização é importante?

A) Porque apaga tokens desnecessários

B) Porque define o preço da chamada

C) Porque ajusta o significado das palavras com base no contexto

D) Porque substitui o uso de API keys

**Resposta correta: C.**

**Explicação:** Muitas palavras têm múltiplos sentidos. A contextualização ajuda o modelo a interpretar o significado correto.

---

### 18. No exemplo do conteúdo, por que a palavra “quantum” precisa de contexto?

A) Porque ela só pode ser usada em computação

B) Porque ela pode ter diferentes significados dependendo da frase

C) Porque ela é sempre ignorada pelo modelo

D) Porque ela não pode ser tokenizada

**Resposta correta: B.**

**Explicação:** “Quantum” pode se referir a física, mecânica quântica, algo subatômico ou computação quântica.

---

### 19. Como Claude gera uma resposta?

A) Copiando uma resposta fixa de um banco de dados

B) Escolhendo token por token com base em probabilidades e aleatoriedade controlada

C) Enviando a pergunta para outro servidor humano

D) Traduzindo automaticamente a pergunta para código

**Resposta correta: B.**

**Explicação:** Claude gera texto sequencialmente, calculando probabilidades para o próximo token.

---

### 20. O que o campo `stop_reason` indica?

A) Qual usuário fez a pergunta

B) Qual modelo foi escolhido

C) Por que a geração terminou

D) Quantas chamadas foram feitas naquele mês

**Resposta correta: C.**

**Explicação:** `stop_reason` informa se a geração parou por limite de tokens, fim natural ou stop sequence.