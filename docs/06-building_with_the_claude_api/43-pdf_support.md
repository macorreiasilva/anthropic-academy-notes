# Suporte a PDF

```
notebooks\28-pdf_support.ipynb
```

## 1. Ideia central

O conteúdo apresenta o **suporte a PDFs no Claude**.

A ideia central é que Claude pode receber arquivos PDF diretamente pela API, de forma parecida com o envio de imagens, e analisar o conteúdo do documento.

Em resumo:

**Claude pode processar PDFs como documentos, extrair informações textuais e visuais, e responder perguntas sobre o conteúdo.**

Isso torna Claude útil para tarefas como resumo de documentos, extração de dados, análise de tabelas, interpretação de gráficos e leitura de documentos longos.

---

## 2. Principais conceitos

### Suporte a PDFs

Claude consegue ler e analisar arquivos PDF enviados como parte da mensagem do usuário.

O funcionamento é parecido com o envio de imagens, mas o bloco usado é diferente.

Para imagens, usamos:

```json
"type": "image"
```

Para PDFs, usamos:

```json
"type": "document"
```

O arquivo PDF é enviado junto com uma instrução textual, como:

```text
Summarize the document in one sentence
```

Claude então analisa o documento e retorna uma resposta em texto.

---

### Estrutura básica do envio de PDF

O PDF é aberto como arquivo binário:

```python
with open("earth.pdf", "rb") as f:
    file_bytes = base64.standard_b64encode(f.read()).decode("utf-8")
```

Depois, ele é enviado dentro de uma mensagem do usuário:

```python
add_user_message(
    messages,
    [
        {
            "type": "document",
            "source": {
                "type": "base64",
                "media_type": "application/pdf",
                "data": file_bytes,
            },
        },
        {"type": "text", "text": "Summarize the document in one sentence"},
    ],
)
```

Por fim, a chamada é feita normalmente:

```python
chat(messages)
```

---

### Diferenças em relação ao envio de imagens

O código é muito parecido com o envio de imagens, mas algumas partes mudam.

| Para imagem                 | Para PDF                          |
| --------------------------- | --------------------------------- |
| `image.png`                 | `earth.pdf`                       |
| `image_bytes`               | `file_bytes`                      |
| `"type": "image"`           | `"type": "document"`              |
| `"media_type": "image/png"` | `"media_type": "application/pdf"` |

A mudança mais importante é que PDFs são enviados como **document blocks**, não como **image blocks**.

---

### O que Claude pode extrair de PDFs

Claude não faz apenas extração simples de texto. Ele pode analisar diferentes elementos dentro do PDF, incluindo:

* texto;
* imagens;
* gráficos;
* tabelas;
* estrutura do documento;
* formatação;
* relações entre dados.

Isso é importante porque muitos documentos reais misturam texto, tabelas e elementos visuais.

Por exemplo, um relatório financeiro pode conter:

* texto explicativo;
* tabelas de resultados;
* gráficos de desempenho;
* imagens;
* notas de rodapé;
* seções e subtítulos.

Claude pode ajudar a interpretar esses elementos em conjunto.

---

### Exemplo usado no conteúdo

O exemplo mostra um PDF chamado:

```text
earth.pdf
```

Esse PDF contém uma página da Wikipédia sobre a Terra.

A instrução enviada a Claude é:

```text
Summarize the document in one sentence
```

Claude processa o PDF e gera um resumo curto sobre o conteúdo do documento.

Esse exemplo demonstra que Claude pode receber um documento PDF e responder perguntas sobre ele sem que o usuário precise extrair manualmente o texto antes.

---

## 3. Pontos importantes para certificação

* Claude pode processar PDFs diretamente.
* PDFs são enviados como blocos do tipo `"document"`.
* O `media_type` para PDF é `"application/pdf"`.
* O arquivo pode ser codificado em base64.
* A mensagem do usuário pode conter um bloco de documento e um bloco de texto.
* O bloco de texto contém a instrução ou pergunta sobre o PDF.
* O código para PDF é parecido com o código para imagem.
* Para PDF, use `file_bytes` ou nome equivalente, em vez de `image_bytes`.
* Claude pode analisar texto, imagens, gráficos, tabelas e estrutura do PDF.
* Esse recurso é útil para resumo, extração e análise de documentos.
* PDFs podem ser tratados como fonte de contexto para perguntas do usuário.
* Mesmo com suporte direto, boas instruções continuam sendo importantes para obter bons resultados.

---

## 4. Termos-chave

**PDF support**
Capacidade de Claude de receber e analisar arquivos PDF diretamente.

**Document block**
Bloco usado para enviar documentos, como PDFs, na mensagem do usuário.

**Base64 encoding**
Forma de transformar um arquivo binário em texto para envio pela API.

**`application/pdf`**
Media type usado para indicar que o arquivo enviado é um PDF.

**`file_bytes`**
Variável usada para armazenar o conteúdo do PDF codificado em base64.

**Text block**
Bloco de texto que acompanha o documento e contém a pergunta ou instrução.

**Document processing**
Processamento de documentos para extrair, resumir ou analisar informações.

**Tables**
Tabelas dentro do PDF que Claude pode interpretar.

**Charts**
Gráficos no PDF que podem ser analisados visualmente.

**Document structure**
Organização do documento, como títulos, seções, colunas e formatação.

---

## 5. Boas práticas

Use um bloco de texto claro junto com o PDF.

Exemplo simples:

```python
{"type": "text", "text": "Summarize the document in one sentence"}
```

Para tarefas mais complexas, seja mais específico:

```text
Analyze this PDF and extract:
1. The main topic
2. The key facts
3. Any tables or figures mentioned
4. A concise summary
```

Também é uma boa prática:

* nomear variáveis de forma clara, como `file_bytes`;
* usar o media type correto: `application/pdf`;
* separar o documento e a instrução em blocos diferentes;
* pedir formatos específicos de saída quando necessário;
* validar respostas importantes, especialmente em documentos técnicos, financeiros ou legais.

---

## 6. Limitações, riscos e cuidados

Embora Claude possa analisar PDFs diretamente, ainda é importante ter cuidado.

Possíveis riscos:

* PDFs muito longos podem consumir muitos tokens;
* documentos escaneados ou com baixa qualidade podem ser mais difíceis de interpretar;
* tabelas complexas podem exigir instruções específicas;
* gráficos podem ser interpretados incorretamente se estiverem pequenos ou ilegíveis;
* documentos sensíveis exigem cuidados de privacidade;
* respostas sobre documentos críticos devem ser revisadas.

Também é importante não assumir que Claude sempre extrairá todos os detalhes automaticamente. Para melhores resultados, diga exatamente o que deve ser analisado.

Por exemplo, em vez de:

```text
Analyze this document.
```

prefira:

```text
Extract the executive summary, list the financial risks, and identify any tables containing revenue data.
```

---

## 7. Resumo para revisão rápida

* Claude pode analisar PDFs.
* PDFs são enviados como `"type": "document"`.
* O media type correto é `"application/pdf"`.
* O PDF pode ser enviado em base64.
* A mensagem combina documento + instrução textual.
* O código é parecido com envio de imagens.
* Para PDFs, trocamos `image` por `document`.
* Claude pode analisar texto, imagens, tabelas e gráficos dentro do PDF.
* O exemplo usa um PDF da Wikipédia sobre a Terra.
* Boas instruções melhoram a qualidade da análise.
* Documentos sensíveis ou críticos exigem validação e cuidado.

---

## 8. Perguntas simuladas

### 1. Como um PDF deve ser enviado para Claude na API?

A) Como um bloco do tipo `"document"`
B) Como um bloco do tipo `"temperature"`
C) Apenas como texto copiado manualmente
D) Como uma API key

**Resposta correta: A.**

**Explicação:** PDFs são enviados como `document blocks`, diferentemente de imagens, que usam `image blocks`.

---

### 2. Qual é o `media_type` correto para um PDF?

A) `"image/png"`
B) `"application/pdf"`
C) `"text/html"`
D) `"audio/mp3"`

**Resposta correta: B.**

**Explicação:** O media type usado para arquivos PDF é `"application/pdf"`.

---

### 3. Qual é uma diferença entre envio de imagem e envio de PDF?

A) PDF usa `"type": "document"`, enquanto imagem usa `"type": "image"`
B) PDF não pode ser enviado em base64
C) Imagem usa `"application/pdf"`
D) PDF não pode ter instrução textual junto

**Resposta correta: A.**

**Explicação:** A principal mudança é o tipo do bloco: imagens usam `image`, PDFs usam `document`.

---

### 4. O que o código faz ao usar `base64.standard_b64encode(f.read())`?

A) Codifica o arquivo PDF em base64
B) Remove todas as imagens do PDF
C) Converte Claude em um banco vetorial
D) Divide o documento em chunks automaticamente

**Resposta correta: A.**

**Explicação:** O arquivo binário é convertido em uma string base64 para envio pela API.

---

### 5. Qual variável é usada no exemplo para armazenar o PDF codificado?

A) `file_bytes`
B) `image_tokens`
C) `max_temperature`
D) `vector_store`

**Resposta correta: A.**

**Explicação:** Para clareza, o exemplo usa `file_bytes`, já que o arquivo é um documento, não uma imagem.

---

### 6. O que deve acompanhar o bloco de documento na mensagem?

A) Um bloco de texto com a pergunta ou instrução
B) Uma chave privada exposta
C) Um valor de cosine similarity
D) Um índice BM25 obrigatório

**Resposta correta: A.**

**Explicação:** O text block informa a Claude o que fazer com o PDF.

---

### 7. O que Claude pode analisar dentro de PDFs?

A) Texto, imagens, gráficos, tabelas e estrutura do documento
B) Apenas o nome do arquivo
C) Apenas a primeira palavra
D) Apenas arquivos sem formatação

**Resposta correta: A.**

**Explicação:** O suporte a PDF permite análise de múltiplos elementos, não apenas texto bruto.

---

### 8. Qual foi o exemplo de PDF usado no conteúdo?

A) Um PDF sobre a Terra
B) Um contrato de aluguel
C) Um arquivo de áudio
D) Uma planilha de vendas

**Resposta correta: A.**

**Explicação:** O exemplo usa um PDF chamado `earth.pdf`, com conteúdo da Wikipédia sobre a Terra.

---

### 9. Qual é uma boa prática ao pedir análise de PDF?

A) Ser específico sobre o que extrair ou resumir
B) Nunca enviar uma pergunta junto com o documento
C) Usar sempre prompts vagos
D) Alterar o media type para `image/png`

**Resposta correta: A.**

**Explicação:** Instruções específicas ajudam Claude a focar nos elementos mais importantes do documento.

---

### 10. Qual frase resume melhor o suporte a PDFs no Claude?

A) Claude pode receber PDFs como documentos, analisar seu conteúdo e responder perguntas sobre texto, imagens, tabelas e estrutura.
B) Claude só aceita PDFs se eles forem convertidos manualmente em texto.
C) Claude não consegue analisar tabelas em PDFs.
D) PDFs devem ser enviados como blocos de imagem.

**Resposta correta: A.**

**Explicação:** Essa é a principal funcionalidade apresentada: análise direta de PDFs pela API.
