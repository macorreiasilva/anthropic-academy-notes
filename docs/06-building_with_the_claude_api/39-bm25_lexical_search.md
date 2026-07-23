# Busca lexical BM25

```
24-bm25_lexical_search.ipynb
```

## 1. Ideia central

O conteúdo explica por que, em pipelines de **RAG**, a busca semântica com embeddings nem sempre é suficiente. Em alguns casos, especialmente quando a pergunta contém termos exatos, IDs, códigos ou nomes específicos, é melhor combinar **busca semântica** com **busca lexical**.

A técnica apresentada para busca lexical é o **BM25**.

A ideia central é:

**Busca semântica encontra conteúdo relacionado por significado. Busca lexical encontra correspondências exatas de termos. Em RAG robusto, muitas vezes usamos as duas juntas e depois mesclamos os resultados.**

---

## 2. Principais conceitos

### O problema da busca semântica sozinha

A busca semântica usa embeddings para encontrar chunks conceitualmente parecidos com a pergunta do usuário.

Isso funciona bem para perguntas abertas, como:

```text
O que o departamento de engenharia de software fez este ano?
```

Mas pode falhar quando a pergunta contém um identificador específico, como:

```text
What happened with INC-2023-Q4-011?
```

Nesse caso, o usuário não quer apenas algo “parecido” semanticamente. Ele quer documentos que mencionem exatamente o incidente `INC-2023-Q4-011`.

A busca semântica pode retornar seções conceitualmente próximas, mas que não contêm esse ID exato.

---

### Busca semântica

A **semantic search** usa:

* embeddings;
* banco vetorial;
* similaridade vetorial;
* significado do texto.

Ela é boa para encontrar conteúdo relacionado por contexto.

Exemplo:

```text
Pergunta: What did the software engineering dept do this year?
```

Mesmo que o chunk não use exatamente as mesmas palavras, a busca semântica pode encontrar uma seção sobre engenharia de software.

---

### Busca lexical

A **lexical search** é a busca textual clássica.

Ela procura termos presentes no texto, como:

* palavras exatas;
* códigos;
* nomes;
* IDs;
* siglas;
* frases específicas.

Exemplo:

```text
INC-2023-Q4-011
```

Se esse código aparece em um chunk, a busca lexical consegue priorizar esse chunk.

Ela é especialmente útil para:

* incident IDs;
* números de ticket;
* nomes de projeto;
* nomes de clientes;
* termos técnicos raros;
* códigos de erro;
* nomes de arquivos;
* referências legais ou regulatórias.

---

### Estratégia híbrida

O conteúdo recomenda uma estratégia híbrida:

```text
Pergunta do usuário
↓
Busca semântica
↓
Resultados semânticos

Pergunta do usuário
↓
Busca lexical
↓
Resultados lexicais

Resultados semânticos + resultados lexicais
↓
Merge results
```

Ou seja, o sistema executa as duas buscas em paralelo e depois combina os resultados.

Essa abordagem aproveita o melhor dos dois mundos:

| Tipo de busca   | Melhor para                              |
| --------------- | ---------------------------------------- |
| Busca semântica | significado, contexto, perguntas abertas |
| Busca lexical   | termos exatos, IDs, nomes, códigos       |

---

### O que é BM25

**BM25**, ou **Best Match 25**, é um algoritmo popular de busca lexical.

Ele é usado para classificar documentos com base na presença e importância dos termos da consulta.

BM25 não tenta entender significado como embeddings fazem. Ele olha para os termos da pergunta e calcula quais documentos são melhores correspondências textuais.

---

### Como o BM25 funciona

O conteúdo mostra quatro etapas principais.

#### 1. Tokenizar a pergunta

A pergunta do usuário é dividida em termos.

Exemplo:

```text
a INC-2023-Q4-011
```

vira:

```text
["a", "INC-2023-Q4-011"]
```

Essa etapa separa os componentes da busca.

---

#### 2. Contar frequência dos termos

O sistema verifica quantas vezes cada termo aparece nos documentos.

Exemplo:

| Termo             | Frequência |
| ----------------- | ---------: |
| `a`               |          5 |
| `INC-2023-Q4-011` |          1 |

A palavra `a` aparece muitas vezes, então ela é pouco informativa.

Já `INC-2023-Q4-011` aparece poucas vezes, então é muito mais importante.

---

#### 3. Dar mais peso a termos raros

Termos menos frequentes recebem mais peso.

Exemplo:

| Termo             | Frequência | Importância |
| ----------------- | ---------: | ----------- |
| `a`               |          5 | Baixa       |
| `INC-2023-Q4-011` |          1 | Alta        |

Isso é essencial porque palavras comuns aparecem em muitos documentos e não ajudam muito a encontrar o trecho certo.

Já termos raros, como IDs de incidente, são muito úteis.

---

#### 4. Encontrar melhores correspondências

O BM25 retorna os chunks que contêm mais ocorrências dos termos importantes.

No exemplo, um chunk que contém `INC-2023-Q4-011` será priorizado sobre outro que contém apenas palavras comuns.

---

### Exemplo em código

A implementação apresentada segue esta ideia:

```python
chunks = chunk_by_section(text)

store = BM25Index()

for chunk in chunks:
    store.add_document({"content": chunk})

results = store.search("What happened with INC-2023-Q4-011?", 3)

for doc, distance in results:
    print(distance, "\n", doc["content"][:200], "\n----\n")
```

Esse código:

1. divide o documento em chunks;
2. cria um índice BM25;
3. adiciona cada chunk ao índice;
4. busca os três resultados mais relevantes;
5. imprime os resultados e suas pontuações/distâncias.

---

### Exemplo dos resultados

No exemplo, a busca por:

```text
What happened with INC-2023-Q4-011?
```

retorna resultados como:

1. seção de **Software Engineering**;
2. seção de **Cybersecurity Analysis — Incident Response Report: INC-2023-Q4-011**;
3. seção de **Methodology**.

A seção de cibersegurança é importante porque contém diretamente o ID do incidente.

O ponto principal é que o BM25 consegue priorizar chunks que mencionam termos raros e específicos.

---

### Por que BM25 melhora o RAG

BM25 melhora o retrieval porque:

* dá peso maior a termos raros;
* reduz a importância de palavras comuns;
* encontra correspondências exatas;
* funciona bem com identificadores específicos;
* complementa a busca semântica.

Em RAG real, a busca híbrida costuma ser mais robusta do que usar apenas embeddings.

---

## 3. Pontos importantes para certificação

* Busca semântica sozinha pode falhar em consultas com termos exatos.
* IDs, códigos e nomes específicos são casos fortes para busca lexical.
* BM25 é um algoritmo popular de busca lexical.
* BM25 significa **Best Match 25**.
* Busca lexical considera termos presentes no texto.
* Busca semântica considera significado e contexto.
* Uma estratégia robusta combina semantic search e lexical search.
* BM25 tokeniza a query do usuário.
* BM25 conta a frequência dos termos nos documentos.
* Termos raros recebem mais peso.
* Termos comuns recebem menos peso.
* BM25 prioriza documentos que contêm termos raros importantes.
* Busca híbrida melhora RAG porque combina significado com correspondência exata.
* O próximo passo natural é mesclar os resultados das duas buscas.

---

## 4. Termos-chave

**BM25**
Algoritmo de busca lexical que ranqueia documentos com base na relevância dos termos da consulta.

**Best Match 25**
Nome completo do BM25.

**Lexical search**
Busca baseada em termos exatos presentes no texto.

**Semantic search**
Busca baseada no significado dos textos usando embeddings.

**Hybrid search**
Combinação de busca semântica e busca lexical.

**Tokenization**
Processo de dividir a consulta em termos menores.

**Term frequency**
Frequência com que um termo aparece nos documentos.

**Term weighting**
Atribuição de pesos diferentes para termos com base em sua importância.

**Rare term**
Termo que aparece poucas vezes e, por isso, costuma ser mais informativo.

**Common term**
Termo frequente, como artigos e preposições, que geralmente tem menor importância.

**Incident ID**
Identificador específico de um incidente, como `INC-2023-Q4-011`.

**Merge results**
Processo de combinar resultados vindos de diferentes métodos de busca.

**RAG**
Técnica que recupera trechos relevantes e os usa para gerar uma resposta com o modelo.

---

## 5. Boas práticas

Use busca semântica para perguntas conceituais e abertas.

Use busca lexical quando a pergunta envolver:

* IDs;
* códigos;
* números de incidente;
* nomes próprios;
* termos técnicos específicos;
* frases exatas;
* nomes de arquivos;
* referências regulatórias.

Para sistemas RAG mais robustos, combine as duas abordagens:

```text
semantic search + lexical search → merge results
```

Também é uma boa prática recuperar múltiplos resultados e avaliar se eles realmente contêm as informações necessárias.

Quando trabalhar com identificadores como:

```text
INC-2023-Q4-011
```

não dependa apenas de embeddings. Use BM25 ou outro mecanismo lexical para garantir que o termo exato seja encontrado.

---

## 6. Limitações, riscos e cuidados

BM25 é poderoso para termos exatos, mas não entende significado profundo.

Por exemplo, se o usuário perguntar:

```text
What did the engineering team improve?
```

e o documento disser:

```text
The software division enhanced system stability.
```

BM25 pode não perceber a relação se os termos exatos forem diferentes.

Nesse caso, a busca semântica pode funcionar melhor.

Por outro lado, embeddings podem falhar quando a query contém identificadores específicos. Eles podem recuperar algo semanticamente parecido, mas sem o ID exato.

Por isso, a melhor abordagem em muitos sistemas reais é híbrida.

Também é importante lembrar que mesclar resultados exige cuidado. O sistema precisa decidir:

* como combinar rankings;
* como remover duplicatas;
* quantos resultados manter;
* como lidar com conflitos;
* como balancear busca semântica e lexical.

---

## 7. Resumo para revisão rápida

* BM25 é usado para busca lexical.
* Busca lexical encontra termos exatos.
* Busca semântica encontra significado parecido.
* Semantic search sozinha pode falhar com IDs específicos.
* BM25 funciona bem com códigos, nomes e incident IDs.
* BM25 tokeniza a pergunta.
* Conta a frequência dos termos nos documentos.
* Termos raros recebem mais importância.
* Termos comuns recebem menos importância.
* Chunks com termos raros importantes sobem no ranking.
* Busca híbrida combina semantic search e lexical search.
* Essa combinação melhora a robustez do RAG.
* O próximo passo é mesclar resultados das duas buscas.

---

## 8. Perguntas simuladas

### 1. Qual problema o BM25 ajuda a resolver em RAG?

A) Falhas da busca semântica ao lidar com termos exatos
B) Falta de API key da Anthropic
C) Erros de formatação em XML
D) Limite de temperature do modelo

**Resposta correta: A.**

**Explicação:** BM25 ajuda a encontrar chunks que contêm termos específicos, como IDs e códigos.

---

### 2. O que é busca lexical?

A) Busca baseada em correspondência de termos no texto
B) Busca baseada apenas em embeddings
C) Busca feita por geração de imagem
D) Busca que ignora palavras exatas

**Resposta correta: A.**

**Explicação:** Busca lexical procura termos presentes nos documentos.

---

### 3. O que é busca semântica?

A) Busca baseada no significado e contexto usando embeddings
B) Busca baseada apenas na frequência de letras
C) Busca que só encontra palavras idênticas
D) Busca manual feita pelo usuário

**Resposta correta: A.**

**Explicação:** Semantic search compara representações vetoriais do significado dos textos.

---

### 4. Qual tipo de pergunta se beneficia muito de busca lexical?

A) Pergunta com identificador específico, como `INC-2023-Q4-011`
B) Pergunta vaga sobre criatividade
C) Pedido para escrever um poema
D) Pedido para resumir um texto já fornecido inteiro

**Resposta correta: A.**

**Explicação:** IDs específicos exigem correspondência exata, algo em que busca lexical é forte.

---

### 5. O que significa BM25?

A) Best Match 25
B) Basic Model 25
C) Binary Matrix 25
D) Base Memory 25

**Resposta correta: A.**

**Explicação:** BM25 significa Best Match 25.

---

### 6. Qual é a primeira etapa do BM25 no exemplo?

A) Tokenizar a query do usuário
B) Gerar imagens dos documentos
C) Criar uma API key
D) Apagar termos raros

**Resposta correta: A.**

**Explicação:** A pergunta é dividida em termos, como `["a", "INC-2023-Q4-011"]`.

---

### 7. No BM25, quais termos recebem maior importância?

A) Termos raros
B) Termos comuns
C) Artigos como “a”
D) Espaços em branco

**Resposta correta: A.**

**Explicação:** Termos raros ajudam mais a identificar documentos relevantes.

---

### 8. Por que a palavra `a` recebe baixa importância no exemplo?

A) Porque aparece com frequência e é pouco informativa
B) Porque é um ID de incidente
C) Porque não pode ser tokenizada
D) Porque só aparece em código Python

**Resposta correta: A.**

**Explicação:** Palavras comuns aparecem em muitos documentos e não ajudam muito na busca.

---

### 9. Qual é a principal vantagem da busca híbrida?

A) Combinar entendimento semântico com correspondência exata de termos
B) Substituir completamente o modelo Claude
C) Eliminar a necessidade de documentos
D) Impedir o uso de embeddings

**Resposta correta: A.**

**Explicação:** Busca híbrida usa semantic search e lexical search para melhorar a recuperação.

---

### 10. Qual frase resume melhor BM25 em RAG?

A) BM25 prioriza documentos que contêm termos importantes da query, especialmente termos raros e específicos.
B) BM25 gera embeddings para todos os chunks.
C) BM25 substitui completamente a necessidade de chunking.
D) BM25 serve apenas para calcular cosine similarity.

**Resposta correta: A.**

**Explicação:** BM25 é um algoritmo lexical que favorece documentos com termos relevantes e raros da consulta.
