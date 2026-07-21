# O fluxo completo de RAG

## 1. Ideia central

O conteúdo explica o **fluxo completo de RAG — Retrieval Augmented Generation**.

A ideia central é mostrar como todas as peças vistas anteriormente se conectam:

1. dividir o documento em chunks;
2. gerar embeddings dos chunks;
3. armazenar os embeddings em um banco vetorial;
4. gerar embedding da pergunta do usuário;
5. encontrar os chunks mais similares;
6. inserir os chunks relevantes no prompt;
7. pedir para Claude gerar a resposta final.

Em resumo:

**RAG permite responder perguntas com base em documentos grandes, recuperando apenas os trechos semanticamente mais relevantes para a pergunta do usuário.**

---

## 2. Principais conceitos

### Fluxo completo de RAG

O pipeline completo segue esta sequência:

```text
Chunk source text
↓
Generate embeddings
↓
Store embeddings in vector database
↓
Embed user query
↓
Find closest embeddings
↓
Add related chunks to the prompt
↓
Claude generates the final answer
```

Esse fluxo combina duas etapas principais:

* **pré-processamento**, feito antes da pergunta do usuário;
* **recuperação em tempo de consulta**, feita quando o usuário faz uma pergunta.

---

### Etapa 1: dividir o texto em chunks

Primeiro, o documento original é dividido em partes menores.

No exemplo, há dois chunks:

```text
Section 1: Medical Research
This year saw significant strides in our understanding of XDR-47, a “bug” we have not seen before.
```

```text
Section 2: Software Engineering
This division dedicated significant effort to studying various infection vectors in our distributed systems.
```

Esses chunks representam seções diferentes do documento.

O detalhe importante é que a palavra **“bug”** aparece em um contexto médico, mas o usuário pode estar perguntando sobre engenharia de software. Por isso, só buscar palavras exatas pode levar ao trecho errado.

---

### Etapa 2: gerar embeddings

Depois de criar os chunks, cada chunk é convertido em um **embedding**.

Um embedding é uma lista de números que representa o significado do texto.

Para facilitar a explicação, o curso usa um modelo imaginário com apenas dois números:

```text
[quanto fala sobre medicina, quanto fala sobre engenharia de software]
```

Exemplo:

```text
Medical Research → [0.97, 0.34]
Software Engineering → [0.30, 0.97]
```

Isso significa:

* o chunk de Medical Research fala muito sobre medicina e um pouco sobre software;
* o chunk de Software Engineering fala muito sobre software e um pouco sobre medicina.

Na prática, embeddings reais têm muito mais dimensões, não apenas duas.

---

### Normalização

O conteúdo mostra que os vetores geralmente passam por uma etapa de **normalização**.

A normalização ajusta o vetor para ter magnitude igual a `1.0`.

Exemplo:

```text
[0.97, 0.34] → [0.944, 0.331]
[0.30, 0.97] → [0.295, 0.955]
```

Isso facilita a comparação entre vetores.

Você não precisa fazer essa matemática manualmente na maioria dos casos; normalmente a API ou o banco vetorial lida com isso.

---

### Etapa 3: armazenar embeddings em um banco vetorial

Depois de gerar os embeddings, eles são armazenados em um **vector database**.

Um banco vetorial é otimizado para armazenar e comparar listas longas de números.

Ele guarda algo como:

```text
Embedding: [0.944, 0.331]
Texto original: Section 1: Medical Research...
```

```text
Embedding: [0.295, 0.955]
Texto original: Section 2: Software Engineering...
```

Assim, o sistema consegue buscar rapidamente quais chunks são mais próximos da pergunta do usuário.

Essa etapa faz parte do pré-processamento.

---

### Etapa 4: gerar embedding da pergunta do usuário

Quando o usuário faz uma pergunta, a pergunta também é convertida em embedding.

Exemplo:

```text
I'm curious about the company. In particular, what did the software engineering dept do this year?
```

O embedding imaginário da pergunta seria algo como:

```text
[0.1, 0.89]
```

Após normalização:

```text
[0.112, 0.993]
```

Isso indica que a pergunta tem pouca relação com medicina e forte relação com engenharia de software.

---

### Etapa 5: encontrar embeddings mais próximos

Agora o sistema compara o embedding da pergunta com os embeddings dos chunks armazenados.

A ideia é descobrir qual chunk está semanticamente mais próximo da pergunta.

No exemplo:

```text
Query: [0.112, 0.993]
Software Engineering: [0.295, 0.955]
Medical Research: [0.944, 0.331]
```

A pergunta está muito mais próxima do chunk **Software Engineering** do que do chunk **Medical Research**.

Logo, o sistema recupera o chunk de engenharia de software.

---

### Cosine similarity

A comparação entre vetores geralmente usa **cosine similarity**.

Cosine similarity mede o cosseno do ângulo entre dois vetores.

Regras principais:

|  Valor | Significado                                       |
| -----: | ------------------------------------------------- |
|  `1.0` | Vetores na mesma direção; alta similaridade       |
|  `0.0` | Vetores perpendiculares; pouca ou nenhuma relação |
| `-1.0` | Vetores em direções opostas; baixa similaridade   |

No exemplo:

```text
Query x Software Engineering = 0.983
Query x Medical Research = 0.398
```

Isso mostra que a pergunta é muito mais parecida semanticamente com o chunk de engenharia de software.

---

### Cosine distance

O conteúdo também cita **cosine distance**.

Ela é calculada assim:

```text
cosine distance = 1 - cosine similarity
```

Com cosine distance:

|          Valor | Significado        |
| -------------: | ------------------ |
| próximo de `0` | alta similaridade  |
|          maior | menor similaridade |

Ou seja:

* cosine similarity alto = textos parecidos;
* cosine distance baixo = textos parecidos.

---

### Etapa 6: criar o prompt final

Depois de encontrar o chunk mais relevante, o sistema monta o prompt final para Claude.

Exemplo:

```xml
Answer the user's question about the financial document.

<user_question>
How many bugs did engineers fix this year?
</user_question>

<report>
## Section 2: Software Engineering
This division dedicated significant effort to studying various infection vectors in our distributed systems.
</report>
```

Claude então responde com base na pergunta e no trecho recuperado.

O ponto importante é que Claude não recebe o documento inteiro, apenas o trecho mais relevante.

---

## 3. Pontos importantes para certificação

* O fluxo completo de RAG combina chunking, embeddings, banco vetorial, busca semântica e geração.
* O documento é dividido em chunks antes da consulta do usuário.
* Cada chunk recebe um embedding.
* Embeddings representam semanticamente o texto em forma numérica.
* Embeddings dos chunks são armazenados em um vector database.
* Quando o usuário pergunta algo, a pergunta também vira um embedding.
* O sistema compara o embedding da pergunta com os embeddings dos chunks.
* O chunk mais similar é recuperado.
* A similaridade geralmente é medida com **cosine similarity**.
* Cosine similarity próximo de `1.0` indica alta similaridade.
* Cosine distance é `1 - cosine similarity`.
* Cosine distance próximo de `0` indica alta similaridade.
* O prompt final inclui a pergunta do usuário e os chunks relevantes.
* Claude usa esses chunks como contexto para responder.
* RAG ajuda a evitar enviar documentos inteiros para o modelo.
* O exemplo mostra por que busca semântica é melhor que keyword search em casos ambíguos, como a palavra “bug”.

---

## 4. Termos-chave

**RAG**
Técnica que recupera informações relevantes e usa essas informações para gerar uma resposta.

**Chunk**
Pedaço menor de um documento.

**Embedding**
Lista de números que representa o significado de um texto.

**Embedding model**
Modelo que transforma texto em embedding.

**Vector database**
Banco de dados otimizado para armazenar e comparar embeddings.

**User query**
Pergunta ou consulta feita pelo usuário.

**Query embedding**
Embedding gerado a partir da pergunta do usuário.

**Stored embedding**
Embedding de um chunk armazenado no banco vetorial.

**Semantic similarity**
Grau de proximidade de significado entre dois textos.

**Cosine similarity**
Métrica que compara a direção de dois vetores.

**Cosine distance**
Métrica calculada como `1 - cosine similarity`.

**Normalization**
Processo de ajustar vetores para magnitude padrão, geralmente `1.0`.

**Retrieval**
Etapa de buscar os chunks mais relevantes.

**Final prompt**
Prompt enviado a Claude contendo a pergunta e os chunks recuperados.

---

## 5. Boas práticas

* Gere embeddings para todos os chunks no pré-processamento.
* Armazene embeddings junto com o texto original do chunk.
* Use o mesmo modelo de embedding para chunks e perguntas.
* Compare a pergunta do usuário com os chunks usando similaridade vetorial.
* Recupere apenas os chunks mais relevantes.
* Use XML tags para separar a pergunta do usuário e o contexto recuperado.
* Inclua no prompt apenas o contexto necessário.
* Preserve metadados dos chunks, como seção, título, página ou origem.
* Avalie se os chunks recuperados realmente respondem à pergunta.
* Use cosine similarity ou métrica equivalente para encontrar proximidade semântica.

Exemplo de prompt bem estruturado:

```xml
<user_question>
{user_question}
</user_question>

<retrieved_context>
{relevant_chunk}
</retrieved_context>

Answer the user's question using only the retrieved context.
```

---

## 6. Limitações, riscos e cuidados

O RAG depende muito da qualidade da recuperação.

Se o sistema recuperar o chunk errado, Claude pode gerar uma resposta errada mesmo que o modelo esteja funcionando corretamente.

Riscos comuns:

* embeddings pouco representativos;
* chunks mal divididos;
* pergunta ambígua;
* chunk recuperado sem contexto suficiente;
* similaridade alta com um trecho irrelevante;
* excesso de confiança em cosine similarity;
* perda de metadados importantes.

Outro cuidado é que embeddings reais não têm apenas duas dimensões. O exemplo com `[medicina, software]` é didático. Na prática, os vetores têm muitas dimensões e não sabemos exatamente o significado individual de cada uma.

Também é importante lembrar que a recuperação não é a resposta final. Ela apenas seleciona o contexto que será enviado a Claude. A geração final ainda precisa ser bem instruída no prompt.

---

## 7. Resumo para revisão rápida

* RAG começa dividindo documentos em chunks.
* Cada chunk vira um embedding.
* Embeddings são vetores numéricos de significado.
* Os embeddings são armazenados em um vector database.
* A pergunta do usuário também vira embedding.
* O sistema compara o embedding da pergunta com os chunks.
* O chunk mais próximo é recuperado.
* Cosine similarity mede proximidade entre vetores.
* Similaridade próxima de `1.0` indica alta relação.
* Cosine distance é `1 - cosine similarity`.
* Distância próxima de `0` indica alta relação.
* O prompt final recebe a pergunta e o chunk recuperado.
* Claude responde usando o contexto recuperado.
* RAG reduz custo, latência e ruído em documentos grandes.

---

## 8. Perguntas simuladas

### 1. Qual é a primeira etapa do fluxo completo de RAG?

A) Gerar a resposta final
B) Dividir o documento em chunks
C) Criar uma API key
D) Remover embeddings antigos

**Resposta correta: B.**

**Explicação:** O pipeline começa dividindo o documento fonte em partes menores chamadas chunks.

---

### 2. O que acontece depois de criar os chunks?

A) Cada chunk é convertido em embedding
B) Cada chunk é apagado
C) Claude é treinado novamente
D) A pergunta do usuário é ignorada

**Resposta correta: A.**

**Explicação:** Os chunks são transformados em vetores numéricos que representam seu significado.

---

### 3. Para que serve um vector database?

A) Para armazenar e comparar embeddings
B) Para criar arquivos Markdown
C) Para substituir Claude
D) Para guardar apenas imagens

**Resposta correta: A.**

**Explicação:** Bancos vetoriais são otimizados para armazenar e buscar vetores similares.

---

### 4. Quando o usuário faz uma pergunta, o que o sistema faz com ela?

A) Converte a pergunta em embedding
B) Envia a pergunta diretamente para todos os usuários
C) Apaga os chunks
D) Converte a pergunta em API key

**Resposta correta: A.**

**Explicação:** A pergunta é transformada em vetor para ser comparada com os embeddings dos chunks.

---

### 5. No exemplo, qual chunk é mais relevante para a pergunta sobre engenharia de software?

A) Medical Research
B) Software Engineering
C) Executive Summary
D) Auditor’s Report

**Resposta correta: B.**

**Explicação:** A pergunta fala sobre o departamento de engenharia de software, então esse chunk é semanticamente mais próximo.

---

### 6. O que mede cosine similarity?

A) O cosseno do ângulo entre dois vetores
B) A quantidade de páginas de um PDF
C) O custo total da API
D) O número de palavras exatas iguais

**Resposta correta: A.**

**Explicação:** Cosine similarity mede a proximidade direcional entre vetores.

---

### 7. O que significa cosine similarity próxima de `1.0`?

A) Alta similaridade entre os vetores
B) Nenhuma relação entre os vetores
C) Vetores completamente opostos
D) Erro no banco de dados

**Resposta correta: A.**

**Explicação:** Vetores na mesma direção ou quase na mesma direção têm similaridade próxima de `1.0`.

---

### 8. Como é calculada a cosine distance?

A) `1 - cosine similarity`
B) `cosine similarity + 1`
C) `embedding / prompt`
D) `chunk_size - overlap`

**Resposta correta: A.**

**Explicação:** Cosine distance é uma forma alternativa de medir distância entre vetores.

---

### 9. Em cosine distance, o que significa valor próximo de `0`?

A) Alta similaridade
B) Baixa similaridade
C) Falha na normalização
D) Documento vazio

**Resposta correta: A.**

**Explicação:** Como cosine distance é `1 - similarity`, valores baixos indicam vetores muito parecidos.

---

### 10. Qual é a última etapa antes de Claude gerar a resposta?

A) Adicionar os chunks relevantes ao prompt
B) Apagar o vector database
C) Enviar todos os documentos sem filtro
D) Trocar o modelo de embedding por uma imagem

**Resposta correta: A.**

**Explicação:** O sistema monta o prompt final com a pergunta do usuário e os trechos recuperados.

---

### 11. Por que o exemplo da palavra “bug” é importante?

A) Porque mostra que palavras iguais podem ter significados diferentes dependendo do contexto
B) Porque prova que busca por palavra-chave sempre é suficiente
C) Porque embeddings não funcionam com palavras ambíguas
D) Porque todo documento financeiro fala de insetos

**Resposta correta: A.**

**Explicação:** “Bug” pode significar um problema de software ou algo em contexto médico; por isso a busca semântica é importante.

---

### 12. Qual frase resume melhor o fluxo completo de RAG?

A) O sistema divide documentos, gera embeddings, busca chunks similares à pergunta e envia esses chunks para Claude responder.
B) O sistema sempre envia o documento inteiro para Claude.
C) Claude ignora os documentos e responde apenas com conhecimento interno.
D) RAG serve apenas para editar arquivos locais.

**Resposta correta: A.**

**Explicação:** Essa é a sequência essencial do pipeline RAG completo.
