# Estratégias de segmentação de texto

```
21-text_chunking-strategies.ipynb
```

## 1. Ideia central

O conteúdo explica **estratégias de text chunking** para pipelines de **RAG — Retrieval Augmented Generation**.

A ideia central é que, em RAG, não basta dividir um documento grande de qualquer jeito. A forma como o texto é quebrado em **chunks** impacta diretamente a qualidade das respostas.

Se os chunks forem ruins, Claude pode receber contexto irrelevante, incompleto ou confuso, levando a respostas incorretas.

Em resumo:

**Chunking é uma das etapas mais importantes de um pipeline RAG, porque determina quais pedaços do documento poderão ser recuperados e enviados ao modelo.**

---

## 2. Principais conceitos

### O que é text chunking

**Text chunking** é o processo de dividir um documento grande em partes menores chamadas **chunks**.

Esses chunks serão armazenados e depois recuperados conforme a pergunta do usuário.

Exemplo de fluxo:

```text
Documento original
↓
Divisão em chunks
↓
Busca pelos chunks mais relevantes
↓
Prompt com pergunta + chunks recuperados
↓
Resposta de Claude
```

No contexto de RAG, o objetivo não é mandar o documento inteiro para Claude, mas apenas os trechos mais relevantes.

---

### Por que chunking importa

A qualidade do chunking afeta diretamente a qualidade da recuperação.

Imagine um documento com seções sobre:

* pesquisa médica;
* engenharia de software.

Se o usuário perguntar:

```text
How many bugs did engineers fix this year?
```

o sistema deveria recuperar a seção de **Software Engineering**.

Mas, se o chunking for ruim, ele pode recuperar um trecho de **Medical Research** porque a palavra “bug” aparece ali em outro sentido.

Esse é um exemplo importante: palavras iguais podem ter significados diferentes dependendo do contexto.

---

### Problema de contexto insuficiente

Um chunk ruim pode:

* cortar palavras no meio;
* cortar frases no meio;
* separar títulos do conteúdo;
* separar seções relacionadas;
* remover contexto necessário para entender o trecho.

Por exemplo, se um chunk contém apenas:

```text
ant strides in our understanding of XDR-47...
```

Claude não sabe o começo da frase nem o contexto da seção.

Isso prejudica a resposta.

---

## 3. Estratégias de chunking

### 1. Size-based chunking

**Size-based chunking** divide o texto em blocos de tamanho fixo.

Por exemplo, se um documento tem **325 caracteres**, podemos dividi-lo em três chunks de aproximadamente **108 caracteres**.

Essa é a abordagem mais simples.

Exemplo conceitual:

```text
Chunk 1: caracteres 0–108
Chunk 2: caracteres 109–216
Chunk 3: caracteres 217–325
```

Código apresentado:

```python
def chunk_by_char(text, chunk_size=150, chunk_overlap=20):
    chunks = []
    start_idx = 0
    
    while start_idx < len(text):
        end_idx = min(start_idx + chunk_size, len(text))
        chunk_text = text[start_idx:end_idx]
        chunks.append(chunk_text)
        
        start_idx = (
            end_idx - chunk_overlap if end_idx < len(text) else len(text)
        )
    
    return chunks
```

Essa função:

* recebe um texto;
* define um tamanho de chunk;
* cria blocos de caracteres;
* usa overlap para reaproveitar parte do chunk anterior.

---

### Vantagens do size-based chunking

Essa estratégia é:

* fácil de implementar;
* previsível;
* funciona com praticamente qualquer tipo de texto;
* funciona inclusive com documentos sem estrutura clara;
* pode ser usada como fallback em produção.

Por isso, mesmo não sendo perfeita, é muito comum.

---

### Desvantagens do size-based chunking

A principal desvantagem é que ele não entende o conteúdo.

Ele pode cortar:

* palavras;
* frases;
* parágrafos;
* seções;
* títulos.

Problemas mostrados nos slides:

```text
Each chunk has some cutoff text
Each chunk lacks context
```

Ou seja, cada chunk pode ficar incompleto e perder contexto.

---

### Overlap entre chunks

Uma solução parcial para os problemas do size-based chunking é usar **overlap**.

Overlap significa que cada chunk compartilha um pedaço de texto com o chunk anterior ou posterior.

Exemplo:

```text
Chunk 1: A B C D
Chunk 2: C D E F
Chunk 3: E F G H
```

Isso ajuda a preservar contexto nas bordas dos chunks.

No código, o overlap aparece aqui:

```python
start_idx = (
    end_idx - chunk_overlap if end_idx < len(text) else len(text)
)
```

Em vez de começar o próximo chunk exatamente onde o anterior terminou, ele volta alguns caracteres.

---

### 2. Structure-based chunking

**Structure-based chunking** divide o documento com base na estrutura natural do texto.

Exemplos de estrutura:

* títulos;
* subtítulos;
* parágrafos;
* seções;
* cabeçalhos Markdown;
* capítulos.

Exemplo com Markdown:

```python
def chunk_by_section(document_text):
    pattern = r"\n## "
    return re.split(pattern, document_text)
```

Esse código divide o documento sempre que encontra um cabeçalho Markdown de nível 2:

```markdown
## Section
```

---

### Vantagens do structure-based chunking

Essa abordagem costuma gerar chunks mais significativos.

Por exemplo:

```text
Chunk 1: Introdução
Chunk 2: Medical Research
Chunk 3: Software Engineering
Chunk 4: Risk Factors
```

Isso evita separar uma seção do seu conteúdo.

É uma boa estratégia quando você controla o formato dos documentos, como:

* relatórios internos padronizados;
* arquivos Markdown;
* documentação técnica;
* documentos com títulos bem definidos.

---

### Desvantagens do structure-based chunking

A limitação é que nem todo documento tem estrutura clara.

Muitos documentos reais vêm como:

* PDFs mal extraídos;
* texto plano;
* imagens digitalizadas;
* documentos com formatação inconsistente;
* arquivos sem cabeçalhos confiáveis.

Essa estratégia exige conhecer previamente a estrutura do documento.

---

### 3. Semantic-based chunking

**Semantic-based chunking** divide o texto com base no significado.

Em vez de dividir por tamanho ou por título, o sistema tenta agrupar frases ou seções relacionadas semanticamente.

A ideia é:

```text
Frases sobre o mesmo assunto → mesmo chunk
Frases sobre outro assunto → outro chunk
```

Essa abordagem tende a produzir chunks mais relevantes.

---

### Vantagens do semantic-based chunking

Ela pode gerar chunks de maior qualidade porque leva em conta o significado do texto.

É útil quando:

* o documento não tem estrutura clara;
* os temas mudam gradualmente;
* a divisão por tamanho cortaria ideias relacionadas;
* a qualidade da recuperação é muito importante.

---

### Desvantagens do semantic-based chunking

A desvantagem é que essa abordagem é mais complexa e cara.

Ela pode exigir:

* embeddings;
* modelos de NLP;
* comparação semântica entre frases;
* mais processamento;
* mais custo computacional.

O conteúdo resume assim:

```text
Computationally expensive, but more relevant chunks
```

---

### 4. Sentence-based chunking

O conteúdo também apresenta uma estratégia intermediária: **sentence-based chunking**.

Ela divide o texto em frases e depois agrupa algumas frases por chunk.

Código apresentado:

```python
def chunk_by_sentence(text, max_sentences_per_chunk=5, overlap_sentences=1):
    sentences = re.split(r"(?<=[.!?])\s+", text)
    
    chunks = []
    start_idx = 0
    
    while start_idx < len(sentences):
        end_idx = min(start_idx + max_sentences_per_chunk, len(sentences))
        current_chunk = sentences[start_idx:end_idx]
        chunks.append(" ".join(current_chunk))
        
        start_idx += max_sentences_per_chunk - overlap_sentences
        
        if start_idx < 0:
            start_idx = 0
    
    return chunks
```

Essa função:

* divide o texto por frases;
* agrupa um número máximo de frases por chunk;
* permite overlap de frases entre chunks.

É um meio-termo entre simplicidade e preservação de contexto.

---

## 4. Pontos importantes para certificação

* **Text chunking é uma etapa crítica em RAG.**
* A estratégia de chunking impacta diretamente a qualidade da resposta.
* Chunking ruim pode recuperar contexto errado.
* Chunking ruim pode cortar frases, palavras ou seções importantes.
* Existem três estratégias principais: **size-based, structure-based e semantic-based**.
* **Size-based chunking** divide texto por tamanho fixo.
* Size-based é simples e robusto, mas pode cortar conteúdo relacionado.
* **Overlap** ajuda a reduzir perda de contexto entre chunks.
* **Structure-based chunking** divide por seções, parágrafos ou cabeçalhos.
* Structure-based funciona bem quando o documento tem estrutura confiável.
* **Semantic-based chunking** divide por significado.
* Semantic-based pode gerar chunks mais relevantes, mas é mais caro e complexo.
* **Sentence-based chunking** é uma alternativa intermediária.
* Não existe uma única melhor estratégia para todos os casos.
* A escolha depende do tipo de documento, caso de uso e trade-off entre simplicidade e qualidade.

---

## 5. Termos-chave

**Text chunking**
Processo de dividir documentos em partes menores para uso em RAG.

**Chunk**
Pedaço menor de texto extraído de um documento.

**RAG**
Técnica que recupera trechos relevantes e os usa para gerar respostas.

**Size-based chunking**
Divisão do texto por tamanho fixo, como número de caracteres ou tokens.

**Structure-based chunking**
Divisão baseada na estrutura do documento, como títulos, seções ou parágrafos.

**Semantic-based chunking**
Divisão baseada no significado e na relação entre frases ou trechos.

**Sentence-based chunking**
Divisão em frases, agrupando várias frases por chunk.

**Overlap**
Sobreposição entre chunks vizinhos para preservar contexto.

**Context**
Informação ao redor de um trecho que ajuda Claude a interpretá-lo corretamente.

**Retrieval**
Etapa de busca dos chunks mais relevantes para a pergunta do usuário.

**Preprocessing**
Etapa anterior à consulta em que o documento é preparado e dividido em chunks.

**Fallback strategy**
Estratégia alternativa usada quando outras opções mais sofisticadas não são possíveis.

---

## 6. Boas práticas

Use **size-based chunking com overlap** quando precisar de uma estratégia simples, robusta e aplicável a qualquer documento.

Use **structure-based chunking** quando o documento tiver estrutura clara, como Markdown, relatórios bem formatados ou documentação técnica.

Use **semantic-based chunking** quando a qualidade da recuperação for mais importante que simplicidade e custo.

Use **sentence-based chunking** como alternativa intermediária para preservar frases inteiras sem depender de cabeçalhos.

Sempre avalie se os chunks:

* preservam contexto suficiente;
* não cortam frases importantes;
* mantêm títulos próximos do conteúdo;
* não misturam assuntos diferentes;
* são relevantes para perguntas reais dos usuários.

Também é importante testar a estratégia com perguntas típicas do usuário. A melhor estratégia é aquela que recupera os chunks certos para o seu caso de uso.

---

## 7. Limitações, riscos e cuidados

Não existe uma estratégia universalmente melhor.

O **size-based chunking** é simples, mas pode cortar conteúdo no meio e gerar chunks sem contexto.

O **structure-based chunking** gera chunks melhores quando há estrutura clara, mas falha quando o documento não tem marcações confiáveis.

O **semantic-based chunking** pode ser mais preciso, mas adiciona custo, complexidade e processamento.

O **overlap** ajuda, mas também aumenta redundância. Se o overlap for grande demais, você pode armazenar e enviar muito texto repetido, aumentando custo.

Outro risco importante é recuperar chunks com palavras semelhantes, mas significado diferente. O exemplo da palavra “bug” mostra isso bem: em uma seção médica, “bug” pode significar microrganismo; em engenharia de software, significa erro de software.

Por isso, chunking deve ser avaliado junto com o mecanismo de busca/retrieval.

---

## 8. Resumo para revisão rápida

* Chunking divide documentos em partes menores.
* É uma etapa essencial em RAG.
* Chunking ruim gera respostas ruins.
* Size-based divide por tamanho fixo.
* Size-based é simples, mas pode cortar contexto.
* Overlap ajuda a preservar contexto.
* Structure-based divide por títulos, seções ou parágrafos.
* Structure-based é bom para documentos bem formatados.
* Semantic-based divide por significado.
* Semantic-based gera chunks mais relevantes, mas custa mais.
* Sentence-based é um meio-termo prático.
* Não há melhor estratégia universal.
* A escolha depende do documento e do caso de uso.
* Avalie se os chunks recuperados realmente respondem à pergunta.

---

## 9. Perguntas simuladas

### 1. O que é text chunking em um pipeline RAG?

A) O processo de treinar Claude novamente
B) O processo de dividir documentos em partes menores
C) O processo de criar uma API key
D) O processo de transformar JSON em imagem

**Resposta correta: B.**

**Explicação:** Text chunking divide documentos grandes em chunks menores que podem ser recuperados e usados no prompt.

---

### 2. Por que chunking é importante?

A) Porque determina a qualidade do contexto enviado ao modelo
B) Porque elimina a necessidade de busca
C) Porque aumenta sempre o custo sem benefício
D) Porque substitui completamente Claude

**Resposta correta: A.**

**Explicação:** Se os chunks forem ruins, Claude pode receber contexto errado ou insuficiente.

---

### 3. Qual é a estratégia mais simples de chunking?

A) Semantic-based chunking
B) Size-based chunking
C) Human-based chunking
D) Model training

**Resposta correta: B.**

**Explicação:** Size-based chunking divide o texto em blocos de tamanho fixo.

---

### 4. Qual é uma desvantagem do size-based chunking?

A) Pode cortar palavras, frases ou seções no meio
B) Exige sempre embeddings avançados
C) Só funciona com Markdown
D) Nunca funciona em produção

**Resposta correta: A.**

**Explicação:** Como ele divide por tamanho, pode quebrar conteúdo relacionado.

---

### 5. Para que serve overlap entre chunks?

A) Para preservar um pouco de contexto entre chunks vizinhos
B) Para apagar conteúdo repetido
C) Para impedir Claude de ler os chunks
D) Para transformar texto em JSON

**Resposta correta: A.**

**Explicação:** Overlap adiciona parte do chunk anterior ou posterior para reduzir perda de contexto.

---

### 6. Quando structure-based chunking é mais adequado?

A) Quando o documento tem estrutura clara, como títulos e seções
B) Quando o documento não tem nenhuma organização
C) Apenas para imagens
D) Quando não há texto disponível

**Resposta correta: A.**

**Explicação:** Essa estratégia usa a estrutura do documento para criar chunks mais significativos.

---

### 7. Qual estratégia tende a gerar chunks semanticamente mais relevantes, mas é mais cara?

A) Size-based

B) Semantic-based

C) Random chunking

D) API-key chunking

**Resposta correta: B.**

**Explicação:** Semantic-based chunking considera o significado das frases, mas exige mais processamento.

---

### 8. O que é sentence-based chunking?

A) Dividir o texto em frases e agrupá-las em chunks

B) Dividir o texto por senha

C) Dividir o texto apenas por imagens

D) Remover todas as frases do documento

**Resposta correta: A.**

**Explicação:** Essa estratégia divide por sentenças e agrupa algumas frases por chunk, com possível overlap.

---

### 9. O que o exemplo da palavra “bug” demonstra?

A) Que palavras podem ter significados diferentes dependendo do contexto

B) Que RAG não usa texto

C) Que Claude não entende nenhuma palavra

D) Que chunking sempre é desnecessário

**Resposta correta: A.**

**Explicação:** “Bug” pode significar microrganismo em pesquisa médica ou erro em software, então contexto é essencial.

---

### 10. Qual frase resume melhor a escolha de estratégia de chunking?

A) Existe uma única estratégia perfeita para todos os documentos

B) A melhor estratégia depende do documento, caso de uso e trade-offs

C) Size-based nunca deve ser usado

D) Semantic-based é sempre obrigatório

**Resposta correta: B.**

**Explicação:** Cada estratégia tem vantagens e limitações; a escolha depende do cenário.
