# Resumo do conteúdo

```
25-a_multi_index_rag_pipeline.ipynb
```

## 1. Ideia central

O conteúdo explica como construir um pipeline de **RAG multi-index**, combinando dois tipos de busca:

* **Semantic Search**, usando embeddings e banco vetorial;
* **Lexical Search**, usando BM25 e busca textual clássica.

A ideia central é:

**Um sistema RAG mais robusto pode consultar múltiplos índices de busca, coletar resultados de cada um e mesclar esses resultados para encontrar os chunks mais relevantes.**

Para isso, o conteúdo apresenta a técnica **Reciprocal Rank Fusion**, ou **RRF**, usada para combinar rankings vindos de diferentes mecanismos de busca.

---

## 2. Principais conceitos

### Multi-Index RAG

Um pipeline **multi-index** usa mais de um índice de busca ao mesmo tempo.

No exemplo:

```text
Pergunta do usuário:
What happened with INC-2023-Q4-011?
```

A pergunta é enviada para:

```text
VectorIndex → busca semântica
BM25Index → busca lexical
```

Depois, os resultados dos dois índices são combinados.

Esse tipo de arquitetura melhora o retrieval porque cada busca tem uma força diferente:

| Índice        | Tipo de busca | Melhor para                                    |
| ------------- | ------------- | ---------------------------------------------- |
| `VectorIndex` | Semântica     | significado, contexto, similaridade conceitual |
| `BM25Index`   | Lexical       | termos exatos, IDs, códigos, nomes específicos |

---

### Por que combinar VectorIndex e BM25Index

A busca semântica pode encontrar conteúdo relacionado mesmo sem termos iguais, mas pode falhar com identificadores específicos.

A busca lexical encontra termos exatos, mas pode não entender significado ou contexto.

Exemplo:

```text
INC-2023-Q4-011
```

Esse é um ID específico. BM25 tende a ser muito bom para encontrar chunks que mencionam exatamente esse ID.

Já o VectorIndex pode encontrar chunks conceitualmente relacionados ao incidente, mesmo que não usem exatamente as mesmas palavras.

A combinação das duas buscas aumenta a chance de recuperar os melhores trechos.

---

### O papel do Retriever

O conteúdo propõe criar uma classe chamada **Retriever**.

Ela atua como uma camada coordenadora que envolve os índices de busca.

Em vez de a aplicação chamar diretamente:

```text
VectorIndex
BM25Index
```

ela chama:

```text
Retriever
```

O Retriever:

1. recebe a pergunta do usuário;
2. envia a pergunta para cada índice;
3. coleta os resultados;
4. mescla os rankings;
5. retorna uma lista final unificada.

Esse padrão deixa o sistema modular e mais fácil de expandir.

---

### Interfaces consistentes

Um detalhe importante é que `VectorIndex` e `BM25Index` têm APIs parecidas.

Ambos possuem métodos como:

```python
add_document()
search()
```

Isso facilita criar um wrapper genérico.

Exemplo conceitual:

```python
class Retriever:
    def __init__(self, *indexes):
        self._indexes = list(indexes)

    def add_document(self, document):
        for index in self._indexes:
            index.add_document(document)

    def search(self, query_text, k=1):
        ...
```

A vantagem é que a aplicação não precisa saber os detalhes internos de cada índice.

---

### Reciprocal Rank Fusion

**Reciprocal Rank Fusion**, ou **RRF**, é uma técnica para combinar resultados vindos de diferentes sistemas de busca.

O problema é que cada índice pode usar pontuações diferentes.

Por exemplo:

* VectorIndex pode retornar distância vetorial;
* BM25Index pode retornar score lexical;
* outros índices poderiam usar métricas próprias.

Comparar diretamente esses scores pode ser injusto.

Por isso, RRF usa a **posição no ranking**, não o score original.

---

### Exemplo de rankings

O conteúdo mostra dois rankings.

#### VectorIndex

| Rank | Chunk                           |
| ---: | ------------------------------- |
|    1 | Section 2: Software Engineering |
|    2 | Section 7: Historical Research  |
|    3 | Section 6: Product Engineering  |

#### BM25Index

| Rank | Chunk                           |
| ---: | ------------------------------- |
|    1 | Section 6: Product Engineering  |
|    2 | Section 2: Software Engineering |
|    3 | Section 7: Historical Research  |

O objetivo é combinar essas duas listas em um ranking final.

---

### Fórmula do RRF

A fórmula apresentada é:

```text
RRF_score(d) = Σ 1 / (k + rank_i(d))
```

Onde:

* `d` é o documento ou chunk;
* `rank_i(d)` é a posição do documento no ranking do índice `i`;
* `k` é uma constante;
* a soma considera todos os índices usados.

Normalmente, `k = 60` é comum, mas no exemplo foi usado `k = 1` para facilitar a visualização dos resultados.

---

### Cálculo do exemplo

#### Section 2: Software Engineering

Rank no VectorIndex: `1`
Rank no BM25Index: `2`

```text
1 / (1 + 1) + 1 / (1 + 2) = 0.833
```

#### Section 7: Historical Research

Rank no VectorIndex: `2`
Rank no BM25Index: `3`

```text
1 / (1 + 2) + 1 / (1 + 3) = 0.583
```

#### Section 6: Product Engineering

Rank no VectorIndex: `3`
Rank no BM25Index: `1`

```text
1 / (1 + 3) + 1 / (1 + 1) = 0.75
```

Resultado final:

| Final Rank | Chunk                           | RRF Score |
| ---------: | ------------------------------- | --------: |
|          1 | Section 2: Software Engineering |     0.833 |
|          2 | Section 6: Product Engineering  |      0.75 |
|          3 | Section 7: Historical Research  |     0.583 |

Quanto maior o score RRF, mais relevante o chunk.

---

### Por que Section 2 fica em primeiro

A Section 2 fica em primeiro porque teve bom desempenho nos dois índices:

```text
VectorIndex: rank 1
BM25Index: rank 2
```

Mesmo que a Section 6 tenha ficado em primeiro no BM25, ela ficou apenas em terceiro no VectorIndex.

O RRF favorece documentos que aparecem bem colocados em múltiplos rankings, não apenas em um único mecanismo.

---

### Resultado prático do pipeline híbrido

O conteúdo mostra que, ao usar apenas busca vetorial, o sistema podia retornar resultados inesperados.

Com o Retriever híbrido, os resultados melhoram:

1. **Section 10: Cybersecurity Analysis — Incident Response Report**
2. **Section 2: Software Engineering — Project Phoenix Stability Enhancements**
3. **Section 5: Legal Developments**

Isso demonstra que combinar busca semântica e lexical pode recuperar contexto mais relevante do que usar apenas uma delas.

---

### Extensibilidade

A arquitetura é extensível.

Se todos os índices implementarem a mesma interface, é possível adicionar novos mecanismos de busca.

Exemplos:

```text
VectorIndex
BM25Index
KeywordIndex
GraphIndex
DomainSpecificIndex
```

O Retriever pode combinar todos eles usando a mesma lógica de fusão de rankings.

Esse é um bom padrão de engenharia: cada índice fica focado em sua própria lógica, enquanto o Retriever coordena a busca e a fusão dos resultados.

---

## 3. Pontos importantes para certificação

* Um pipeline **Multi-Index RAG** usa mais de um índice de busca.
* O objetivo é combinar pontos fortes de diferentes mecanismos de retrieval.
* `VectorIndex` faz busca semântica usando embeddings.
* `BM25Index` faz busca lexical usando termos exatos.
* Busca semântica é boa para significado e contexto.
* Busca lexical é boa para IDs, nomes, códigos e termos exatos.
* O **Retriever** coordena múltiplos índices.
* O Retriever envia a query para cada índice e mescla os resultados.
* **Reciprocal Rank Fusion** combina rankings de diferentes mecanismos.
* RRF usa a posição no ranking, não necessariamente o score bruto.
* A fórmula é `Σ 1 / (k + rank_i(d))`.
* Quanto maior o score RRF, mais relevante é o chunk.
* Um chunk que aparece bem colocado em vários índices tende a subir no ranking final.
* A constante `k` controla o impacto das posições no ranking.
* `k = 60` é comum, mas o exemplo usa `k = 1` para facilitar o entendimento.
* A arquitetura é modular e permite adicionar novos índices no futuro.

---

## 4. Termos-chave

**Multi-Index RAG**
Pipeline RAG que usa múltiplos índices de busca para recuperar contexto.

**Retriever**
Classe ou componente que coordena diferentes mecanismos de busca e mescla seus resultados.

**VectorIndex**
Índice de busca semântica baseado em embeddings e similaridade vetorial.

**BM25Index**
Índice de busca lexical baseado no algoritmo BM25.

**Semantic Search**
Busca por significado e contexto usando embeddings.

**Lexical Search**
Busca por termos exatos presentes no texto.

**Hybrid Search**
Combinação de busca semântica e lexical.

**Reciprocal Rank Fusion**
Técnica para combinar rankings de diferentes sistemas de busca.

**RRF Score**
Pontuação calculada com base nas posições de um chunk em múltiplos rankings.

**Rank**
Posição de um documento ou chunk em uma lista de resultados.

**`k` constant**
Constante usada na fórmula RRF para suavizar o impacto dos rankings.

**SearchIndex protocol**
Interface comum esperada dos índices, como `add_document()` e `search()`.

**Merge results**
Processo de combinar resultados vindos de múltiplas buscas.

---

## 5. Boas práticas

Use busca híbrida quando o sistema precisar lidar tanto com perguntas conceituais quanto com termos específicos.

Exemplo de pergunta que se beneficia da busca híbrida:

```text
What happened with INC-2023-Q4-011?
```

Essa pergunta contém um ID exato, mas também exige contexto semântico sobre o que aconteceu.

Use uma interface comum para os índices:

```python
add_document()
search()
```

Isso facilita criar um `Retriever` genérico e extensível.

Use RRF para combinar resultados quando os scores dos mecanismos forem diferentes ou difíceis de comparar diretamente.

Também é uma boa prática remover duplicatas antes de montar o prompt final, já que o mesmo chunk pode aparecer em mais de um índice.

---

## 6. Limitações, riscos e cuidados

A fusão de rankings melhora o retrieval, mas não garante perfeição.

Alguns cuidados importantes:

* resultados duplicados precisam ser tratados;
* rankings muito ruins de um índice podem afetar a fusão;
* o valor de `k` pode influenciar o resultado final;
* buscar muitos chunks pode aumentar custo e ruído no prompt;
* buscar poucos chunks pode perder contexto importante;
* diferentes índices podem retornar formatos de score incompatíveis.

Outro ponto importante: RRF usa posição no ranking, não o conteúdo em si. Então, se os índices retornarem rankings ruins, a fusão também pode ser prejudicada.

Além disso, a implementação do `Retriever` deve ser testada com perguntas reais. A melhor combinação de índices depende do tipo de documento, do domínio e das perguntas dos usuários.

---

## 7. Resumo para revisão rápida

* Multi-Index RAG combina múltiplos sistemas de busca.
* VectorIndex busca por significado.
* BM25Index busca por termos exatos.
* O Retriever coordena os índices.
* A query é enviada para todos os índices.
* Cada índice retorna seu próprio ranking.
* Os rankings são mesclados.
* Reciprocal Rank Fusion combina rankings de forma justa.
* RRF usa posição no ranking, não score bruto.
* Fórmula: `Σ 1 / (k + rank_i(d))`.
* Score maior significa chunk mais relevante.
* Chunks bem colocados em vários índices sobem no ranking final.
* A arquitetura é extensível para novos tipos de índice.
* Busca híbrida melhora a robustez do RAG.

---

## 8. Perguntas simuladas

### 1. O que é um pipeline Multi-Index RAG?

A) Um pipeline que usa apenas um prompt grande
B) Um pipeline que combina múltiplos índices de busca para recuperar contexto
C) Um pipeline que não usa embeddings
D) Um pipeline usado apenas para gerar imagens

**Resposta correta: B.**

**Explicação:** Multi-Index RAG usa mais de um mecanismo de busca, como VectorIndex e BM25Index.

---

### 2. Qual é a função do Retriever?

A) Coordenar múltiplos índices de busca e mesclar seus resultados
B) Substituir Claude na geração final
C) Criar API keys automaticamente
D) Editar arquivos locais sem permissão

**Resposta correta: A.**

**Explicação:** O Retriever recebe a query, consulta os índices e combina os resultados.

---

### 3. Qual índice é usado para busca semântica?

A) VectorIndex
B) BM25Index
C) MarkdownIndex
D) APIKeyIndex

**Resposta correta: A.**

**Explicação:** VectorIndex usa embeddings para encontrar similaridade semântica.

---

### 4. Qual índice é usado para busca lexical?

A) VectorIndex
B) BM25Index
C) ImageIndex
D) CalendarIndex

**Resposta correta: B.**

**Explicação:** BM25Index usa busca textual clássica e prioriza termos relevantes.

---

### 5. Por que não basta concatenar os resultados de VectorIndex e BM25Index?

A) Porque os scores vêm de sistemas diferentes e não são diretamente comparáveis
B) Porque Claude não aceita texto de múltiplos chunks
C) Porque BM25 não retorna resultados
D) Porque embeddings são sempre inválidos

**Resposta correta: A.**

**Explicação:** Cada índice usa métricas diferentes; por isso é melhor combinar rankings com uma técnica como RRF.

---

### 6. O que é Reciprocal Rank Fusion?

A) Uma técnica para combinar rankings de diferentes sistemas de busca
B) Uma forma de calcular embeddings
C) Um algoritmo para criar arquivos JSON
D) Um método para aumentar temperature

**Resposta correta: A.**

**Explicação:** RRF combina resultados usando a posição dos documentos em cada ranking.

---

### 7. Qual informação o RRF usa principalmente?

A) A posição do chunk no ranking
B) A cor do slide
C) O nome do arquivo `.env`
D) A quantidade de imagens no documento

**Resposta correta: A.**

**Explicação:** RRF calcula pontuação com base no rank de cada documento em cada índice.

---

### 8. Qual é a fórmula geral do RRF?

A) `RRF_score(d) = Σ 1 / (k + rank_i(d))`
B) `RRF_score(d) = temperature * max_tokens`
C) `RRF_score(d) = chunk_size - overlap`
D) `RRF_score(d) = API_KEY + query`

**Resposta correta: A.**

**Explicação:** Essa fórmula soma contribuições baseadas no ranking de cada índice.

---

### 9. No RRF, o que significa score maior?

A) O chunk é considerado mais relevante
B) O chunk deve ser descartado
C) O chunk é mais antigo
D) O chunk tem menos texto

**Resposta correta: A.**

**Explicação:** Quanto maior o score RRF, melhor o ranking final do chunk.

---

### 10. Por que a Section 2 ficou em primeiro no exemplo?

A) Porque teve boa posição nos dois índices
B) Porque apareceu apenas no BM25Index
C) Porque não apareceu em nenhum ranking
D) Porque tinha o maior número de caracteres

**Resposta correta: A.**

**Explicação:** A Section 2 foi rank 1 no VectorIndex e rank 2 no BM25Index, então acumulou o maior score RRF.

---

### 11. Qual é uma vantagem da arquitetura com Retriever?

A) Permite adicionar novos índices mantendo uma interface comum
B) Impede o uso de busca lexical
C) Remove a necessidade de chunking
D) Faz Claude ignorar o contexto recuperado

**Resposta correta: A.**

**Explicação:** Com métodos padronizados como `add_document()` e `search()`, novos índices podem ser integrados facilmente.

---

### 12. Qual frase resume melhor Multi-Index RAG?

A) Multi-Index RAG combina diferentes métodos de busca, como semântica e lexical, e usa fusão de rankings para recuperar melhores chunks.
B) Multi-Index RAG envia sempre o documento inteiro para Claude.
C) Multi-Index RAG substitui embeddings por API keys.
D) Multi-Index RAG serve apenas para streaming de resposta.

**Resposta correta: A.**

**Explicação:** O objetivo é aumentar a robustez do retrieval combinando diferentes mecanismos de busca.
