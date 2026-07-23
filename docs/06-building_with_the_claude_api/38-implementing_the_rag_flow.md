# Implementando o fluxo RAG

```
23-implementing_the_rag_flow.ipynb
```

## 1. Ideia central

O conteúdo mostra como implementar, na prática, o fluxo básico de **RAG — Retrieval Augmented Generation**.

A ideia central é transformar documentos em **chunks**, gerar **embeddings** para esses chunks, armazená-los em um **vector store** e, quando o usuário fizer uma pergunta, buscar os chunks mais relevantes por similaridade.

Em resumo:

**RAG funciona convertendo textos em vetores numéricos, armazenando esses vetores e usando busca por similaridade para recuperar o contexto mais útil para Claude responder.**

---

## 2. Principais conceitos

### As cinco etapas da implementação de RAG

A implementação segue cinco passos principais:

```text
1. Dividir o texto em chunks
2. Gerar embeddings para cada chunk
3. Criar um vector store e armazenar os embeddings
4. Gerar embedding da pergunta do usuário
5. Buscar os chunks mais relevantes
```

Essas etapas conectam o conceito teórico de RAG com uma implementação funcional em código.

---

### Etapa 1: dividir o texto em chunks

Primeiro, o documento é carregado a partir de um arquivo:

```python
with open("./report.md", "r") as f:
    text = f.read()
```

Depois, o texto é dividido em seções:

```python
chunks = chunk_by_section(text)
```

A função `chunk_by_section()` divide o documento em partes menores, normalmente com base em seções ou cabeçalhos.

Essa etapa é importante porque o sistema não quer enviar o documento inteiro para Claude. Ele quer recuperar apenas as partes relevantes.

---

### Etapa 2: gerar embeddings dos chunks

Depois de criar os chunks, o sistema gera embeddings para todos eles:

```python
embeddings = generate_embedding(chunks)
```

A função `generate_embedding()` foi adaptada para aceitar tanto:

* uma única string;
* uma lista de strings.

Isso permite processar vários chunks de uma vez, o que é mais eficiente.

Cada embedding é uma lista de números que representa semanticamente o conteúdo daquele chunk.

---

### Etapa 3: armazenar embeddings em um vector store

Depois de gerar os embeddings, o sistema cria um índice vetorial:

```python
store = VectorIndex()
```

Em seguida, armazena cada embedding junto com seu texto original:

```python
for embedding, chunk in zip(embeddings, chunks):
    store.add_vector(embedding, {"content": chunk})
```

Aqui, `zip(embeddings, chunks)` combina cada embedding com o chunk correspondente.

O ponto mais importante é que o sistema não armazena apenas o vetor. Ele também armazena o conteúdo original do chunk.

---

### Por que armazenar o texto original?

Quando o sistema fizer uma busca, ele não quer receber apenas números como:

```text
[0.295, 0.955, ...]
```

Esses números são úteis para comparação matemática, mas não são úteis para montar o prompt final.

O sistema precisa recuperar o texto original, por exemplo:

```text
Section 2: Software Engineering
This division dedicated significant effort...
```

Por isso, cada embedding é armazenado com metadados:

```python
{"content": chunk}
```

Assim, quando o vector store encontra um embedding parecido com a pergunta, ele também consegue retornar o chunk textual correspondente.

---

### Etapa 4: gerar embedding da pergunta do usuário

Quando o usuário faz uma pergunta, essa pergunta também precisa ser convertida em embedding:

```python
user_embedding = generate_embedding("What did the software engineering dept do last year?")
```

Isso permite comparar a pergunta com os chunks armazenados.

A lógica é:

```text
embedding da pergunta
comparado com
embeddings dos chunks
```

---

### Etapa 5: buscar os chunks mais relevantes

Por fim, o sistema busca no vector store os chunks mais próximos da pergunta:

```python
results = store.search(user_embedding, 2)
```

O número `2` indica que queremos os dois chunks mais relevantes.

Depois, os resultados são exibidos:

```python
for doc, distance in results:
    print(distance, "\n", doc["content"][0:200], "\n")
```

Cada resultado contém:

* o documento ou chunk recuperado;
* a distância de similaridade.

---

### Interpretando os resultados

No exemplo, a pergunta é sobre o departamento de engenharia de software.

O sistema retorna:

1. **Section 2: Software Engineering** com distância `0.71`;
2. **Methodology section** com distância `0.72`.

Como se trata de **cosine distance**, valores menores indicam maior similaridade.

Portanto:

```text
0.71 é mais relevante que 0.72
```

A seção de engenharia de software é o chunk mais relevante para a pergunta.

---

### Distância menor significa maior similaridade

Um ponto importante:

Em algumas métricas, como **cosine similarity**, valores maiores indicam maior similaridade.

Mas em **cosine distance**, a lógica é invertida:

```text
menor distância = maior similaridade
```

Então, se dois resultados têm distâncias:

```text
0.71
0.72
```

o resultado `0.71` é considerado mais próximo da pergunta.

---

## 3. Pontos importantes para certificação

* RAG pode ser implementado em cinco etapas principais.
* Primeiro, o documento é dividido em chunks.
* Depois, cada chunk é convertido em embedding.
* Embeddings são representações numéricas do significado do texto.
* Os embeddings são armazenados em um vector store.
* É essencial armazenar também o texto original do chunk.
* A pergunta do usuário também é convertida em embedding.
* O sistema compara o embedding da pergunta com os embeddings armazenados.
* A busca retorna os chunks mais relevantes.
* Em cosine distance, valores menores indicam maior similaridade.
* O número passado em `store.search(user_embedding, 2)` indica quantos resultados recuperar.
* O chunk recuperado será usado depois para montar o prompt final enviado a Claude.
* RAG é, essencialmente, texto → números → busca por similaridade → contexto recuperado → resposta do modelo.

---

## 4. Termos-chave

**RAG**
Técnica que recupera trechos relevantes de documentos e os usa para gerar respostas com Claude.

**Chunk**
Parte menor de um documento, usada para facilitar busca e recuperação.

**Chunking**
Processo de dividir um documento em chunks.

**Embedding**
Lista de números que representa semanticamente um texto.

**Vector store**
Estrutura ou banco usado para armazenar embeddings e permitir buscas por similaridade.

**VectorIndex**
Classe usada no exemplo para criar um índice vetorial simples.

**`generate_embedding()`**
Função que transforma texto em embedding.

**`chunk_by_section()`**
Função que divide o documento em seções.

**`zip()`**
Função Python usada para combinar embeddings e chunks correspondentes.

**Metadata**
Informações associadas ao embedding, como o conteúdo original do chunk.

**User embedding**
Embedding gerado a partir da pergunta do usuário.

**Similarity search**
Busca pelos vetores mais parecidos com o vetor da pergunta.

**Cosine distance**
Métrica de distância em que valores menores indicam maior similaridade.

---

## 5. Boas práticas

Sempre armazene o texto original junto com o embedding.

Exemplo:

```python
store.add_vector(embedding, {"content": chunk})
```

Isso é necessário porque, no final, Claude precisa receber texto, não apenas vetores.

Também é uma boa prática recuperar mais de um chunk:

```python
store.search(user_embedding, 2)
```

Recuperar dois ou mais resultados pode ajudar quando a resposta depende de múltiplas seções do documento.

Outras boas práticas:

* use a mesma função de embedding para chunks e perguntas;
* preserve metadados como seção, página ou título;
* verifique se os chunks recuperados fazem sentido;
* avalie os resultados manualmente durante o desenvolvimento;
* teste perguntas diferentes para validar a qualidade do retrieval;
* lembre que menor cosine distance significa maior relevância.

---

## 6. Limitações, riscos e cuidados

Essa implementação funciona para casos básicos, mas pode falhar em situações mais complexas.

Alguns riscos:

* o chunk mais próximo pode não conter toda a informação necessária;
* a pergunta pode ser ambígua;
* dois chunks podem ter distâncias muito próximas;
* o chunking pode ter dividido mal o documento;
* o vector store pode recuperar um trecho semanticamente parecido, mas insuficiente;
* o sistema pode precisar recuperar mais chunks ou aplicar filtros adicionais.

Outro cuidado é que a busca vetorial retorna proximidade semântica, não garantia de resposta correta.

Por exemplo, se a pergunta for sobre “software engineering”, o sistema pode encontrar a seção certa. Mas se a resposta exigir números específicos, datas ou detalhes espalhados por várias partes do documento, apenas um chunk talvez não baste.

---

## 7. Resumo para revisão rápida

* Implementar RAG envolve cinco etapas.
* Primeiro, carregamos o documento.
* Depois, dividimos o texto em chunks.
* Geramos embeddings para cada chunk.
* Criamos um vector store.
* Armazenamos embedding + conteúdo original.
* A pergunta do usuário também vira embedding.
* Buscamos os chunks mais próximos.
* A busca retorna documentos e distâncias.
* Em cosine distance, menor valor significa maior similaridade.
* O resultado mais relevante será usado no prompt final.
* RAG converte texto em números para permitir busca semântica.

---

## 8. Perguntas simuladas

### 1. Qual é a primeira etapa da implementação de RAG apresentada?

A) Gerar a resposta final de Claude
B) Dividir o texto em chunks
C) Criar uma API key
D) Apagar o documento original

**Resposta correta: B.**

**Explicação:** O pipeline começa carregando o documento e dividindo-o em partes menores.

---

### 2. Para que serve `chunk_by_section(text)`?

A) Para dividir o documento em seções ou chunks
B) Para gerar uma API key
C) Para calcular cosine distance diretamente
D) Para enviar a resposta ao usuário

**Resposta correta: A.**

**Explicação:** Essa função divide o texto em partes menores com base em seções.

---

### 3. O que faz `generate_embedding(chunks)`?

A) Converte os chunks em embeddings
B) Remove todos os chunks irrelevantes
C) Cria um arquivo `.env`
D) Gera uma resposta final em linguagem natural

**Resposta correta: A.**

**Explicação:** A função transforma cada chunk em uma representação numérica.

---

### 4. Por que o sistema armazena o conteúdo original junto com o embedding?

A) Porque depois precisamos recuperar o texto real para colocar no prompt
B) Porque embeddings não podem ser armazenados sozinhos
C) Porque Claude só aceita números no prompt
D) Porque o texto original substitui a API key

**Resposta correta: A.**

**Explicação:** A busca retorna o chunk textual associado ao embedding, e esse texto será usado como contexto para Claude.

---

### 5. O que faz `store.add_vector(embedding, {"content": chunk})`?

A) Adiciona um embedding ao vector store junto com o chunk correspondente
B) Apaga um embedding antigo
C) Executa uma chamada para Claude
D) Calcula automaticamente a resposta final

**Resposta correta: A.**

**Explicação:** Essa linha armazena o vetor e o conteúdo textual associado.

---

### 6. O que acontece quando o usuário faz uma pergunta?

A) A pergunta também é convertida em embedding
B) O documento inteiro é sempre enviado para Claude
C) Todos os chunks são apagados
D) O vector store é ignorado

**Resposta correta: A.**

**Explicação:** Para comparar a pergunta com os chunks, o sistema precisa gerar um embedding da consulta.

---

### 7. O que significa `store.search(user_embedding, 2)`?

A) Buscar os dois chunks mais relevantes para a pergunta
B) Criar dois novos documentos
C) Dividir o texto em dois arquivos
D) Gerar duas API keys

**Resposta correta: A.**

**Explicação:** O segundo argumento indica quantos resultados relevantes retornar.

---

### 8. Em cosine distance, qual resultado é mais similar?

A) O de maior distância
B) O de menor distância
C) O que tem mais caracteres
D) O que aparece por último no documento

**Resposta correta: B.**

**Explicação:** Em cosine distance, quanto menor o valor, maior a similaridade.

---

### 9. No exemplo, por que a seção “Software Engineering” é recuperada?

A) Porque a pergunta fala sobre o departamento de engenharia de software
B) Porque ela é sempre a primeira seção do documento
C) Porque ela tem a maior distância
D) Porque o sistema ignora embeddings

**Resposta correta: A.**

**Explicação:** A pergunta é semanticamente mais próxima dessa seção.

---

### 10. Qual frase resume melhor a implementação de RAG?

A) Converter texto em embeddings, armazená-los e buscar os chunks mais similares à pergunta do usuário.
B) Enviar sempre o documento inteiro para Claude sem filtro.
C) Usar apenas palavras-chave exatas e ignorar embeddings.
D) Treinar Claude novamente para cada documento.

**Resposta correta: A.**

**Explicação:** A implementação de RAG se baseia em embeddings, armazenamento vetorial e busca por similaridade.
