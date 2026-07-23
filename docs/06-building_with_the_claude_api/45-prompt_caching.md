# Cache de prompt

## 1. Ideia central

O conteúdo apresenta o recurso de **Prompt Caching**.

A ideia central é que Claude normalmente precisa reprocessar todo o prompt a cada request, mesmo quando parte do conteúdo já foi enviada antes. Com **prompt caching**, Claude pode reutilizar parte do trabalho computacional feito em uma request anterior, tornando requests futuras **mais rápidas e mais baratas** quando elas repetem o mesmo conteúdo.

Em resumo:

**Prompt caching salva o processamento de partes repetidas do prompt para reutilizá-las em chamadas futuras.**

---

## 2. Principais conceitos

### Como Claude processa uma request normalmente

Quando o servidor envia uma mensagem para Claude, o modelo não começa imediatamente a gerar a resposta.

Antes disso, Claude precisa processar a entrada:

1. tokenizar o prompt;
2. criar embeddings para os tokens;
3. adicionar contexto com base no texto ao redor;
4. gerar a resposta final.

Esse processamento é feito sobre a mensagem de entrada.

Exemplo:

```text
User Message:
Summarize this long text: <text>
```

Claude processa o texto longo, gera um resumo e devolve uma resposta.

O problema é que, sem cache, depois da resposta todo esse trabalho é descartado.

---

### O problema de descartar o trabalho

Imagine que o usuário faz uma primeira request:

```text
Summarize this long text: <text>
```

Claude processa todo o texto e gera:

```text
<Summary>
```

Depois, o usuário faz uma follow-up request:

```text
The summary needs more focus on...
```

Para manter contexto, o servidor envia novamente o histórico da conversa, incluindo o mesmo texto longo original.

Sem cache, Claude precisa processar de novo aquele conteúdo que já havia processado momentos antes.

Isso é ineficiente porque:

* aumenta custo;
* aumenta latência;
* repete processamento;
* desperdiça trabalho computacional.

---

### Como o Prompt Caching resolve isso

Com prompt caching, o trabalho feito para processar partes repetidas do prompt pode ser armazenado.

Na request inicial:

```text
Initial Request:
Please summarize this...
```

Claude processa o conteúdo e grava parte desse trabalho no cache.

Em uma request seguinte, se o mesmo conteúdo aparecer novamente, Claude pode reutilizar o processamento salvo em vez de refazer tudo do zero.

O cache funciona como uma espécie de tabela:

```text
Se eu vir esta mesma mensagem novamente,
posso reutilizar o trabalho que já fiz.
```

---

### Escrita e leitura do cache

O conteúdo destaca dois momentos:

#### Initial request

A primeira request **escreve no cache**.

Ou seja, Claude processa o conteúdo normalmente e salva o trabalho feito.

```text
Initial request → write to cache
```

#### Follow-up request

Requests seguintes podem **ler do cache**.

```text
Follow-up request → read from cache
```

Assim, a parte repetida do conteúdo pode ser processada mais rapidamente e com menor custo.

---

### Benefícios principais

Prompt caching traz dois benefícios centrais:

| Benefício      | Explicação                                          |
| -------------- | --------------------------------------------------- |
| Menor custo    | Conteúdo em cache é mais barato de processar        |
| Menor latência | Requests com conteúdo cacheado executam mais rápido |

Isso é especialmente útil quando o mesmo conteúdo grande aparece repetidamente em várias chamadas.

---

### Duração do cache

O conteúdo informa que o cache dura **uma hora**.

Isso significa que o conteúdo cacheado não fica disponível indefinidamente.

```text
Cache lives for one hour
```

Portanto, o recurso é mais útil em fluxos onde o usuário faz várias perguntas ou revisões em sequência sobre o mesmo conteúdo.

---

### Quando o Prompt Caching é útil

Prompt caching é útil quando você envia repetidamente o mesmo conteúdo.

Exemplos:

* fazer várias perguntas sobre o mesmo documento longo;
* revisar várias versões de um resumo;
* trabalhar com uma base de instruções longa;
* analisar o mesmo PDF ou texto em várias etapas;
* manter uma conversa longa com contexto repetido;
* usar prompts de sistema grandes e reutilizados frequentemente.

Exemplo prático:

```text
Request 1:
Summarize this long text.

Request 2:
Make the summary more technical.

Request 3:
Now focus on the risks.

Request 4:
Extract the key terms.
```

Se o texto longo é reenviado em todas essas requests, o cache pode economizar custo e tempo.

---

### Quando o Prompt Caching não ajuda muito

Prompt caching não é útil quando cada request contém conteúdo totalmente diferente.

Exemplo:

```text
Request 1: Resuma o documento A.
Request 2: Resuma o documento B.
Request 3: Resuma o documento C.
```

Se o conteúdo muda completamente, não há muito trabalho reutilizável.

O recurso só faz sentido quando existe repetição significativa no prompt.

---

## 3. Pontos importantes para certificação

* Prompt caching reutiliza trabalho computacional de requests anteriores.
* Claude normalmente tokeniza, cria embeddings, contextualiza e gera texto.
* Sem cache, esse trabalho é descartado após a resposta.
* Com cache, parte do trabalho pode ser salva para uso posterior.
* Requests com conteúdo cacheado são mais rápidas.
* Requests com conteúdo cacheado são mais baratas.
* A primeira request escreve no cache.
* Requests seguintes podem ler do cache.
* O cache dura uma hora.
* Prompt caching é útil quando o mesmo conteúdo é enviado repetidamente.
* É especialmente útil para documentos longos, follow-ups e workflows iterativos.
* Não traz muito benefício quando cada request é totalmente diferente.
* O maior ganho aparece quando há conteúdo grande e repetido.

---

## 4. Termos-chave

**Prompt Caching**
Recurso que permite reutilizar processamento feito em prompts anteriores.

**Cache**
Armazenamento temporário de trabalho computacional já realizado.

**Initial Request**
Primeira chamada que processa o conteúdo e pode escrever no cache.

**Follow-up Request**
Chamada posterior que pode reutilizar conteúdo já cacheado.

**Tokenization**
Processo de dividir o texto em tokens.

**Embeddings**
Representações numéricas criadas para tokens ou textos.

**Contextualization**
Etapa em que o modelo interpreta tokens considerando o contexto ao redor.

**Latency**
Tempo que a request leva para ser processada e respondida.

**Cached content**
Conteúdo cujo processamento já foi salvo e pode ser reutilizado.

**Cache lifetime**
Tempo durante o qual o conteúdo permanece disponível no cache.

---

## 5. Boas práticas

Use prompt caching quando houver conteúdo grande e repetido.

Bons casos:

```text
Várias perguntas sobre o mesmo documento
Várias revisões do mesmo texto
Prompt de sistema longo usado em várias chamadas
Análise iterativa de um relatório
Workflow de resumo, extração e refinamento
```

Organize o prompt para que a parte repetida fique estável entre requests.

Isso aumenta a chance de reutilização do cache.

Também é importante considerar o tempo: como o cache dura uma hora, ele é mais útil em sessões ativas ou fluxos próximos no tempo.

---

## 6. Limitações, riscos e cuidados

Prompt caching não deve ser visto como solução universal.

Principais limitações:

* o cache dura apenas uma hora;
* só ajuda quando o mesmo conteúdo é reenviado;
* não melhora muito requests totalmente novas;
* ainda há custo na primeira request;
* o benefício depende da frequência de reutilização;
* pode exigir organização cuidadosa dos prompts.

Também é importante não confundir prompt caching com memória permanente.

O cache não é uma memória de longo prazo do usuário. Ele apenas reutiliza processamento temporário de conteúdo repetido.

Outro cuidado: se o conteúdo muda muito entre requests, o cache pode ser pouco aproveitado.

---

## 7. Resumo para revisão rápida

* Claude normalmente reprocessa o prompt a cada request.
* Esse processamento inclui tokenização, embeddings e contextualização.
* Sem cache, o trabalho é descartado após a resposta.
* Prompt caching salva parte desse trabalho.
* A primeira request escreve no cache.
* Follow-ups podem ler do cache.
* Requests cacheadas são mais rápidas.
* Requests cacheadas são mais baratas.
* O cache dura uma hora.
* É útil para conteúdo repetido.
* É ótimo para documentos longos e conversas iterativas.
* Não ajuda muito quando cada request é diferente.
* Cache não é memória permanente.

---

## 8. Perguntas simuladas

### 1. O que é Prompt Caching?

A) Um recurso que reutiliza processamento de prompts anteriores
B) Um método para gerar imagens
C) Um tipo de busca BM25
D) Um recurso que remove a necessidade de API key

**Resposta correta: A.**

**Explicação:** Prompt caching salva trabalho computacional de prompts repetidos para reutilização posterior.

---

### 2. O que Claude normalmente faz antes de gerar a resposta?

A) Tokeniza o prompt, cria embeddings e adiciona contexto
B) Apenas copia a resposta de outro usuário
C) Apaga o prompt do servidor
D) Cria automaticamente um banco vetorial

**Resposta correta: A.**

**Explicação:** O conteúdo mostra que Claude faz várias etapas de processamento antes da geração.

---

### 3. O que acontece sem prompt caching após uma request?

A) O trabalho de processamento é descartado
B) O cache fica salvo para sempre
C) A resposta vira automaticamente um PDF
D) O modelo deixa de funcionar

**Resposta correta: A.**

**Explicação:** Sem cache, Claude joga fora o processamento feito para aquela entrada.

---

### 4. Qual é o principal benefício do prompt caching?

A) Requests com conteúdo repetido ficam mais rápidas e baratas
B) Todas as respostas ficam criativas
C) O modelo deixa de precisar de tokens
D) O cache substitui RAG completamente

**Resposta correta: A.**

**Explicação:** O cache reduz custo e latência quando o mesmo conteúdo é reutilizado.

---

### 5. O que a initial request faz no cache?

A) Escreve no cache
B) Lê do cache obrigatoriamente
C) Apaga todo o histórico
D) Gera embeddings para uma base vetorial externa

**Resposta correta: A.**

**Explicação:** A primeira request processa o conteúdo e pode salvar o trabalho no cache.

---

### 6. O que uma follow-up request pode fazer?

A) Ler do cache
B) Ignorar todo o conteúdo anterior sempre
C) Criar uma imagem automaticamente
D) Substituir o documento por uma API key

**Resposta correta: A.**

**Explicação:** Requests seguintes podem reutilizar o processamento salvo anteriormente.

---

### 7. Por quanto tempo o cache vive, segundo o conteúdo?

A) Uma hora
B) Um minuto
C) Um ano
D) Para sempre

**Resposta correta: A.**

**Explicação:** O conteúdo afirma que o cache vive por uma hora.

---

### 8. Quando prompt caching é mais útil?

A) Quando o mesmo conteúdo é enviado repetidamente
B) Quando cada request é totalmente diferente
C) Quando não há prompt
D) Quando se usa apenas temperature alta

**Resposta correta: A.**

**Explicação:** O recurso depende da repetição de conteúdo para gerar economia.

---

### 9. Qual exemplo se beneficia de prompt caching?

A) Fazer várias perguntas sobre o mesmo documento longo
B) Enviar um documento diferente a cada chamada
C) Gerar uma única frase sem contexto
D) Remover o servidor da arquitetura

**Resposta correta: A.**

**Explicação:** Documentos longos reutilizados em follow-ups são um caso clássico de uso.

---

### 10. Prompt caching é equivalente a memória permanente?

A) Não, é um cache temporário de processamento
B) Sim, salva tudo para sempre
C) Sim, substitui bancos de dados
D) Sim, elimina a necessidade de histórico de mensagens

**Resposta correta: A.**

**Explicação:** O cache dura por tempo limitado e serve para reutilizar processamento, não para memória permanente.

---

### 11. Qual é uma limitação importante do prompt caching?

A) Só é útil se houver conteúdo repetido
B) Não funciona com texto
C) Só funciona para imagens
D) Impede follow-up requests

**Resposta correta: A.**

**Explicação:** Se o conteúdo muda sempre, há pouco ou nenhum benefício.

---

### 12. Qual frase resume melhor Prompt Caching?

A) Prompt caching torna requests repetidas mais rápidas e baratas ao reutilizar processamento feito anteriormente.
B) Prompt caching gera citações automaticamente em PDFs.
C) Prompt caching é uma técnica de chunking semântico.
D) Prompt caching substitui todas as ferramentas externas.

**Resposta correta: A.**

**Explicação:** Esse é o objetivo principal do recurso: reutilizar trabalho computacional para reduzir custo e latência.
