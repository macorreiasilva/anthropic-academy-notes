# Incorporação de texto

```
22-text_embeddings.ipynb
```

## 1. Ideia central

O conteúdo explica **text embeddings**, uma etapa essencial em pipelines de **RAG — Retrieval Augmented Generation**.

Depois de dividir um documento em **chunks**, o próximo desafio é descobrir quais chunks são mais relevantes para a pergunta do usuário.

Esse problema é tratado como uma tarefa de busca.

A ideia central é:

**Text embeddings transformam textos em listas de números que representam significado, permitindo comparar semanticamente a pergunta do usuário com os chunks do documento.**

Em vez de procurar apenas palavras iguais, como em busca por palavra-chave, usamos **busca semântica** para encontrar trechos que falam sobre o mesmo assunto, mesmo que usem palavras diferentes.

---

## 2. Principais conceitos

### Encontrar chunks relevantes

Depois que um documento é dividido em chunks, o sistema precisa responder:

```text
Quais desses chunks estão mais relacionados à pergunta do usuário?
```

Exemplo do conteúdo:

```text
User’s Question:
How many bugs did engineers fix this year?
```

O documento contém chunks sobre:

* pesquisa médica;
* engenharia de software;
* outras áreas de pesquisa.

A palavra **bug** pode aparecer tanto em contexto médico quanto em contexto de software. Por isso, uma busca simples por palavra-chave pode recuperar o chunk errado.

Esse é o problema que a busca semântica tenta resolver.

---

### Busca semântica

**Semantic search**, ou busca semântica, é uma técnica que tenta encontrar textos relacionados pelo **significado**, não apenas por palavras exatas.

Busca por palavra-chave pergunta:

```text
Este chunk contém a palavra “bug”?
```

Busca semântica pergunta:

```text
Este chunk fala sobre o mesmo assunto da pergunta do usuário?
```

Isso é muito importante em RAG, porque o sistema precisa recuperar o contexto certo antes de chamar Claude.

Se o chunk recuperado for errado, Claude pode gerar uma resposta errada mesmo que o modelo seja bom.

---

### Text embeddings

Um **text embedding** é uma representação numérica do significado de um texto.

Na prática, o texto é enviado para um **embedding model**, que retorna uma lista de números.

Exemplo conceitual:

```text
"I'm very happy today!"
```

vira algo como:

```text
[0.279, -0.05, -0.009, 0.143, ...]
```

Cada texto recebe uma lista numérica diferente.

Esses vetores permitem comparar textos matematicamente.

---

### Embedding model

Um **embedding model** é o modelo responsável por converter texto em vetor.

O fluxo é:

```text
Texto
↓
Embedding model
↓
Lista de números
```

Exemplos de entrada:

```text
I'm very happy today!
That movie wasn't great
abcd
```

Cada uma dessas frases é transformada em uma lista de números diferente.

Esses números representam características aprendidas pelo modelo durante o treinamento.

---

### O que os números representam

Cada número em um embedding pode ser entendido como uma espécie de “pontuação” para alguma característica do texto.

Conceitualmente, podemos imaginar dimensões como:

* o quanto o texto expressa felicidade;
* o quanto fala sobre oceanos;
* o quanto fala sobre frutas;
* o quanto é formal;
* o quanto se refere a dirigir.

Mas há um cuidado importante:

**não sabemos exatamente o que cada número representa.**

Essas dimensões são aprendidas pelo modelo e não são diretamente interpretáveis por humanos.

Então, exemplos como “este número representa felicidade” servem apenas para intuição, não como uma explicação literal.

---

### Intervalo dos valores

O conteúdo explica que os números do embedding costumam estar em uma faixa como:

```text
-1 a +1
```

Na prática, cada texto vira um vetor com muitos valores decimais.

Exemplo:

```text
[-0.0543, 0.0146, -0.0170, 0.0005, 0.0215, ...]
```

Esse vetor pode depois ser comparado com outros vetores.

---

### VoyageAI para embeddings

O conteúdo destaca que, nesse curso, a geração de embeddings é feita com **VoyageAI**.

Para usar, é necessário:

1. criar uma conta na VoyageAI;
2. obter uma API key;
3. adicionar a chave ao arquivo `.env`.

Exemplo:

```env
VOYAGE_API_KEY="your_key_here"
```

Também aparece no ambiente:

```env
ANTHROPIC_API_KEY="MY_KEY"
VOYAGE_API_KEY="MY_KEY"
```

Cuidado importante: chaves de API devem ficar em arquivos `.env` e não devem ser expostas em repositórios públicos.

---

### Instalação da biblioteca

Para usar VoyageAI em Python, instala-se a biblioteca:

```python
%pip install voyageai
```

Depois, o cliente é configurado:

```python
from dotenv import load_dotenv
import voyageai

load_dotenv()
client = voyageai.Client()
```

Aqui:

* `load_dotenv()` carrega variáveis do arquivo `.env`;
* `voyageai.Client()` cria o cliente para chamar a API da VoyageAI.

---

### Função para gerar embeddings

A função apresentada é:

```python
def generate_embedding(text, model="voyage-3-large", input_type="query"):
    result = client.embed([text], model=model, input_type=input_type)
    return result.embeddings[0]
```

Ela recebe:

* `text`: o texto a ser convertido em embedding;
* `model`: o modelo de embedding;
* `input_type`: o tipo de entrada, como `"query"`.

Ela retorna:

```text
uma lista de números
```

Essa lista representa semanticamente o texto enviado.

---

### Exemplo com chunks

Depois de carregar um documento:

```python
with open("./report.md", "r") as f:
    text = f.read()
```

O texto é dividido em chunks:

```python
chunks = chunk_by_section(text)
```

Depois, gera-se o embedding de um chunk:

```python
generate_embedding(chunks[0])
```

A saída é uma lista longa de números, como:

```text
[-0.054362788796424866,
 0.014613168314099312,
 -0.017022375017404556,
 ...]
```

Esse embedding será usado posteriormente para comparar o chunk com a pergunta do usuário.

---

## 3. Pontos importantes para certificação

* Depois do chunking, o próximo passo em RAG é encontrar os chunks mais relevantes.
* Encontrar chunks relevantes é um problema de busca.
* Busca semântica usa significado, não apenas palavras exatas.
* Text embeddings representam textos como listas de números.
* Embeddings permitem comparar textos matematicamente.
* Um embedding model transforma texto em vetor numérico.
* Cada número representa alguma característica aprendida do texto.
* Não sabemos exatamente o que cada dimensão do embedding representa.
* Embeddings ajudam a comparar perguntas com chunks.
* Busca semântica reduz problemas de ambiguidade de palavras.
* A palavra “bug” pode ter significados diferentes dependendo do contexto.
* No curso, VoyageAI é usado para gerar embeddings.
* É necessário configurar `VOYAGE_API_KEY` no `.env`.
* A função `generate_embedding()` chama `client.embed()`.
* O próximo passo depois de gerar embeddings é comparar vetores para medir similaridade.

---

## 4. Termos-chave

**Text embedding**
Representação numérica do significado de um texto.

**Embedding model**
Modelo que transforma texto em uma lista de números.

**Vector**
Lista numérica que representa um texto em espaço matemático.

**Semantic search**
Busca baseada no significado dos textos, não apenas em palavras iguais.

**Chunk**
Trecho menor de um documento usado em RAG.

**Relevant chunk**
Chunk mais relacionado à pergunta do usuário.

**RAG**
Técnica que recupera trechos relevantes e usa esses trechos para gerar uma resposta.

**Query**
Pergunta ou consulta do usuário.

**VoyageAI**
Serviço usado no curso para gerar embeddings.

**`VOYAGE_API_KEY`**
Chave de API usada para autenticar chamadas à VoyageAI.

**`.env`**
Arquivo usado para armazenar variáveis sensíveis, como API keys.

**`client.embed()`**
Método usado para gerar embeddings com o cliente da VoyageAI.

**Similarity**
Medida de quão próximos dois embeddings estão semanticamente.

---

## 5. Boas práticas

Use embeddings para buscar chunks por significado, especialmente quando palavras podem ter múltiplos sentidos.

Armazene chaves de API no `.env`:

```env
VOYAGE_API_KEY="your_key_here"
```

Não coloque chaves reais em notebooks, arquivos versionados ou prints públicos.

Gere embeddings para:

* a pergunta do usuário;
* cada chunk do documento.

Depois, compare esses embeddings para encontrar os chunks semanticamente mais próximos.

Também é importante escolher corretamente o `input_type`. Em muitos sistemas de embedding, há diferença entre embeddings gerados para consultas e embeddings gerados para documentos.

Em pipelines RAG reais, uma prática comum é:

```text
embedding da pergunta do usuário
comparado com
embeddings dos chunks armazenados
```

Os chunks mais similares são enviados para Claude no prompt.

---

## 6. Limitações, riscos e cuidados

Embeddings são poderosos, mas não são perfeitamente interpretáveis.

Não devemos assumir que uma dimensão específica sempre representa algo claro, como “felicidade” ou “formalidade”. Essa explicação é apenas uma analogia.

Outro risco é confiar cegamente na busca semântica. O sistema pode recuperar chunks parecidos, mas ainda insuficientes ou errados.

Também há custos e dependências externas. Usar VoyageAI exige:

* conta separada;
* API key;
* chamadas adicionais;
* controle de custos;
* cuidado com privacidade dos dados enviados.

Se os documentos forem sensíveis, é importante avaliar políticas de privacidade e segurança antes de enviar texto para um provedor externo de embeddings.

Além disso, embeddings precisam ser combinados com boa estratégia de chunking. Se os chunks forem ruins, os embeddings representarão trechos ruins.

---

## 7. Resumo para revisão rápida

* Depois de chunking, precisamos encontrar chunks relevantes.
* Isso é um problema de busca.
* Busca semântica usa significado, não apenas palavras-chave.
* Embeddings transformam texto em listas de números.
* Esses números representam características aprendidas do texto.
* Não sabemos exatamente o significado de cada dimensão.
* Embeddings permitem comparar pergunta e chunks.
* A palavra “bug” mostra por que contexto importa.
* VoyageAI é usado para gerar embeddings no curso.
* A API key deve ficar no `.env`.
* `generate_embedding()` retorna uma lista numérica.
* O próximo passo é comparar embeddings para medir similaridade.

---

## 8. Perguntas simuladas

### 1. Qual é o próximo passo em RAG depois de dividir o documento em chunks?

A) Apagar todos os chunks
B) Encontrar quais chunks são mais relevantes para a pergunta do usuário
C) Treinar Claude novamente
D) Transformar o documento em imagem

**Resposta correta: B.**

**Explicação:** Depois do chunking, o sistema precisa recuperar os chunks mais relacionados à consulta do usuário.

---

### 2. O que é semantic search?

A) Busca baseada apenas em palavras idênticas
B) Busca baseada no significado e contexto dos textos
C) Busca que ignora a pergunta do usuário
D) Busca feita apenas em arquivos `.env`

**Resposta correta: B.**

**Explicação:** Busca semântica tenta encontrar textos relacionados pelo significado, mesmo que usem palavras diferentes.

---

### 3. O que é um text embedding?

A) Uma API key criptografada
B) Uma representação numérica do significado de um texto
C) Um tipo de markdown
D) Um arquivo gerado pelo navegador

**Resposta correta: B.**

**Explicação:** Embeddings transformam texto em vetores numéricos que podem ser comparados matematicamente.

---

### 4. O que um embedding model retorna?

A) Uma lista de números
B) Um arquivo PDF
C) Uma imagem
D) Uma senha temporária

**Resposta correta: A.**

**Explicação:** O embedding model converte texto em um vetor, ou seja, uma lista numérica.

---

### 5. Sabemos exatamente o que cada número em um embedding representa?

A) Sim, cada número sempre tem um rótulo humano claro
B) Não, as dimensões são aprendidas pelo modelo e não são diretamente interpretáveis
C) Sim, o primeiro número sempre representa felicidade
D) Não, porque embeddings não têm significado algum

**Resposta correta: B.**

**Explicação:** Podemos usar analogias, mas não sabemos precisamente o significado individual de cada dimensão.

---

### 6. Por que a palavra “bug” é um bom exemplo nesse conteúdo?

A) Porque pode ter significados diferentes em medicina e engenharia de software
B) Porque sempre significa erro de software
C) Porque nunca aparece em documentos reais
D) Porque não pode ser representada por embeddings

**Resposta correta: A.**

**Explicação:** “Bug” pode se referir a um microrganismo ou a um erro de software, dependendo do contexto.

---

### 7. Qual serviço é usado no curso para gerar embeddings?

A) VoyageAI
B) GitHub Actions
C) Google Calendar
D) Markdown Parser

**Resposta correta: A.**

**Explicação:** O conteúdo usa VoyageAI como provedor de embeddings.

---

### 8. Onde a chave `VOYAGE_API_KEY` deve ser armazenada?

A) Em um arquivo `.env`
B) Dentro do prompt enviado ao usuário
C) Em uma imagem pública
D) No README público do GitHub

**Resposta correta: A.**

**Explicação:** Chaves de API devem ser armazenadas com segurança em variáveis de ambiente, geralmente via `.env`.

---

### 9. O que faz a função `generate_embedding()`?

A) Gera uma lista de números representando semanticamente um texto
B) Divide um documento em páginas
C) Remove todos os chunks
D) Cria uma resposta final de Claude sem contexto

**Resposta correta: A.**

**Explicação:** A função chama o modelo de embedding e retorna o vetor numérico do texto.

---

### 10. Qual é o próximo passo depois de gerar embeddings para pergunta e chunks?

A) Comparar os embeddings para encontrar os chunks mais similares
B) Apagar a pergunta do usuário
C) Ignorar os chunks recuperados
D) Colocar todas as API keys no prompt

**Resposta correta: A.**

**Explicação:** A comparação de embeddings permite identificar quais chunks têm maior similaridade semântica com a pergunta.
