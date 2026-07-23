# Citações

```
notebooks\29-citations.ipynb
```

## 1. Ideia central

O conteúdo explica o recurso de **citations** no Claude.

A ideia central é que, quando Claude responde com base em documentos fornecidos pelo usuário, ele pode indicar **exatamente de onde tirou cada informação**.

Em resumo:

**Citations tornam as respostas de Claude mais transparentes, verificáveis e confiáveis, conectando trechos da resposta aos documentos de origem.**

Isso é especialmente útil quando Claude analisa PDFs, textos longos ou documentos técnicos, porque o usuário consegue verificar se a resposta está realmente fundamentada no material enviado.

---

## 2. Principais conceitos

### Por que citações importam

Quando Claude responde uma pergunta sobre um documento, o usuário pode não saber se a resposta veio:

* do conhecimento geral do modelo;
* do documento enviado;
* de uma inferência do modelo;
* ou de uma mistura dessas fontes.

As **citations** resolvem esse problema ao mostrar o trecho específico do documento que sustenta uma afirmação.

Exemplo de pergunta:

```text
How did Earth's atmosphere form?
```

Sem citações, Claude poderia responder corretamente, mas o usuário não teria como verificar a fonte.

Com citações, Claude pode indicar que determinada frase foi baseada em um trecho específico do PDF `earth.pdf`.

---

### Como ativar citations

Para ativar citações em um documento enviado à API, é necessário adicionar dois campos ao bloco de documento:

```json
{
  "type": "document",
  "source": {
    "type": "base64",
    "media_type": "application/pdf",
    "data": file_bytes
  },
  "title": "earth.pdf",
  "citations": { "enabled": true }
}
```

Os campos importantes são:

* `title`: nome legível do documento;
* `citations`: configuração que ativa o rastreamento das fontes.

O campo:

```json
"citations": { "enabled": true }
```

informa a Claude que ele deve rastrear de onde as informações foram extraídas.

---

### Estrutura de uma citação

Quando citations estão ativadas, a resposta de Claude pode incluir metadados sobre a origem de cada trecho.

A estrutura apresentada contém:

| Campo               | Função                          |
| ------------------- | ------------------------------- |
| `cited_text`        | Texto exato citado do documento |
| `document_index`    | Índice do documento citado      |
| `document_title`    | Título do documento citado      |
| `start_page_number` | Página inicial do trecho citado |
| `end_page_number`   | Página final do trecho citado   |

Exemplo:

```json
{
  "cited_text": "Earth's atmosphere and oceans were formed by volcanic activity and outgassing.",
  "document_index": 0,
  "document_title": "earth.pdf",
  "start_page_number": 4,
  "end_page_number": 5
}
```

Isso mostra não apenas o texto citado, mas também **qual documento** e **quais páginas** sustentam a afirmação.

---

### `cited_text`

O campo `cited_text` contém o trecho do documento que Claude está usando como evidência.

Exemplo:

```text
Earth's atmosphere and oceans were formed by volcanic activity and outgassing.
```

Esse trecho é útil para mostrar ao usuário o suporte direto para determinada parte da resposta.

---

### `document_index`

O campo `document_index` indica qual documento foi citado.

Isso é importante quando a request contém múltiplos documentos.

Exemplo:

```json
"document_index": 0
```

Significa que Claude está citando o primeiro documento enviado.

Se houvesse vários PDFs, esse campo ajudaria a identificar qual deles serviu como fonte.

---

### `document_title`

O campo `document_title` indica o nome do documento citado.

Exemplo:

```json
"document_title": "earth.pdf"
```

Esse título é definido no bloco do documento enviado à API.

Por isso, é uma boa prática dar nomes claros aos documentos.

---

### `start_page_number` e `end_page_number`

Esses campos indicam em quais páginas o trecho citado começa e termina.

Exemplo:

```json
"start_page_number": 4,
"end_page_number": 5
```

Isso permite que o usuário abra o PDF e confira a informação diretamente no local correto.

---

### Citações em interface de usuário

O conteúdo mostra que citations podem ser exibidas em uma interface como marcadores numéricos:

```text
Earth's atmosphere and oceans were formed by volcanic activity and outgassing. [1]
```

Esses marcadores podem ser interativos.

Por exemplo, o usuário pode clicar ou passar o mouse sobre `[1]` para ver:

* o texto citado;
* o documento de origem;
* a página;
* o contexto da citação.

Isso transforma Claude em um assistente mais transparente, porque ele “mostra o trabalho” e não apenas entrega uma resposta final.

---

### Citações com texto puro

Citations não funcionam apenas com PDFs.

Também é possível usar documentos em texto puro.

Exemplo:

```json
{
  "type": "document",
  "source": {
    "type": "text",
    "media_type": "text/plain",
    "data": article_text
  },
  "title": "earth_article",
  "citations": { "enabled": true }
}
```

Nesse caso, em vez de páginas, a citação pode usar posições de caracteres para indicar onde o trecho aparece no texto.

---

### Quando usar citations

Citations são especialmente úteis quando:

* o usuário precisa verificar a fonte;
* a resposta envolve documentos oficiais;
* a aplicação exige transparência;
* há risco de erro ou alucinação;
* o usuário pode querer consultar o contexto original;
* o domínio é sensível, como jurídico, financeiro, médico, governança ou compliance.

Em aplicações profissionais, citations ajudam a aumentar confiança e auditabilidade.

---

## 3. Pontos importantes para certificação

* Citations permitem que Claude mostre de onde tirou informações em documentos fornecidos.
* Elas aumentam transparência e verificabilidade.
* Para PDFs, citations podem indicar páginas específicas.
* Para texto puro, citations podem indicar posições no texto.
* Para ativar citations, use `"citations": { "enabled": true }`.
* O documento deve ter um `title`.
* O campo `cited_text` contém o trecho do documento usado como evidência.
* `document_index` identifica qual documento foi citado.
* `document_title` mostra o título do documento citado.
* `start_page_number` indica a página inicial da citação.
* `end_page_number` indica a página final da citação.
* Citations são úteis quando há múltiplos documentos.
* Citations ajudam a diferenciar resposta baseada no documento de resposta baseada no conhecimento geral do modelo.
* Interfaces podem exibir citações como marcadores clicáveis ou interativos.
* Citations são importantes para confiança, auditoria e aplicações de alto risco.

---

## 4. Termos-chave

**Citations**
Recurso que permite a Claude indicar a fonte exata de uma informação usada na resposta.

**`cited_text`**
Texto exato do documento que sustenta uma afirmação.

**`document_index`**
Número que identifica qual documento foi citado quando múltiplos documentos foram enviados.

**`document_title`**
Nome ou título do documento citado.

**`start_page_number`**
Página onde o trecho citado começa.

**`end_page_number`**
Página onde o trecho citado termina.

**Document block**
Bloco usado para enviar documentos, como PDFs ou textos, para Claude.

**Source document**
Documento fornecido pelo usuário como fonte de informação.

**Grounding**
Ato de fundamentar a resposta do modelo em uma fonte específica.

**Verifiability**
Capacidade de verificar se uma informação está correta consultando a fonte original.

**Transparency**
Clareza sobre a origem das informações usadas na resposta.

**Plain text citations**
Citações feitas a partir de documentos em texto puro, geralmente com posições de caracteres em vez de páginas.

---

## 5. Boas práticas

Sempre dê títulos claros aos documentos.

Exemplo:

```json
"title": "earth.pdf"
```

Evite nomes genéricos como:

```json
"title": "document.pdf"
```

Use citations quando a aplicação envolver:

* documentos oficiais;
* relatórios financeiros;
* contratos;
* artigos técnicos;
* políticas internas;
* PDFs regulatórios;
* documentos de compliance;
* decisões com impacto relevante.

Também é uma boa prática construir interfaces que mostrem as citações de forma acessível, por exemplo:

```text
A atmosfera da Terra foi formada por atividade vulcânica e liberação de gases. [1]
```

E permitir que `[1]` revele:

* trecho citado;
* nome do documento;
* página inicial;
* página final.

Além disso, peça respostas baseadas no documento quando necessário:

```text
Answer using only the provided document and include citations for key claims.
```

Isso reduz o risco de Claude misturar informações externas ou conhecimento geral com o conteúdo do documento.

---

## 6. Limitações, riscos e cuidados

Citations aumentam transparência, mas não eliminam todos os riscos.

Cuidados importantes:

* Claude pode citar um trecho correto, mas interpretá-lo de forma incompleta;
* a citação pode sustentar apenas parte da afirmação;
* documentos longos podem ter múltiplos trechos relevantes;
* a resposta ainda deve ser revisada em contextos críticos;
* citações não substituem validação humana em domínios sensíveis;
* se o documento estiver mal formatado ou escaneado com baixa qualidade, a extração pode ser prejudicada.

Outro cuidado: não confundir “ter citação” com “estar automaticamente correto”.

Uma resposta citada é mais verificável, mas ainda pode conter:

* erro de interpretação;
* omissão de contexto;
* conclusão além do que o documento permite;
* uso seletivo de evidências.

Por isso, em aplicações profissionais, citations devem ser combinadas com bons prompts, validação e revisão.

---

## 7. Resumo para revisão rápida

* Citations mostram a origem das informações usadas por Claude.
* Elas são ativadas no document block.
* Use `"citations": { "enabled": true }`.
* Use também um campo `title` claro.
* `cited_text` mostra o trecho citado.
* `document_index` indica qual documento foi usado.
* `document_title` mostra o nome do documento.
* `start_page_number` e `end_page_number` indicam páginas no PDF.
* Em texto puro, citations podem usar posições de caracteres.
* Citations ajudam em transparência, confiança e auditoria.
* São úteis em PDFs, documentos oficiais e aplicações de alto risco.
* Ter citação melhora verificabilidade, mas não garante interpretação perfeita.

---

## 8. Perguntas simuladas

### 1. Para que servem citations no Claude?

A) Para mostrar de onde Claude tirou informações em documentos fornecidos
B) Para aumentar a temperature da resposta
C) Para gerar embeddings automaticamente
D) Para substituir o uso de PDFs

**Resposta correta: A.**

**Explicação:** Citations conectam afirmações da resposta a trechos específicos dos documentos de origem.

---

### 2. Qual campo ativa citations em um document block?

A) `"citations": { "enabled": true }`
B) `"temperature": 1.0`
C) `"media_type": "image/png"`
D) `"thinking": false`

**Resposta correta: A.**

**Explicação:** Esse campo informa a Claude que ele deve rastrear e retornar citações.

---

### 3. Qual campo dá um nome legível ao documento citado?

A) `title`
B) `max_tokens`
C) `temperature`
D) `rank`

**Resposta correta: A.**

**Explicação:** O campo `title` define o nome do documento, como `earth.pdf`.

---

### 4. O que representa `cited_text`?

A) O trecho exato do documento citado por Claude
B) A resposta final inteira
C) O nome do modelo usado
D) O número total de tokens da imagem

**Resposta correta: A.**

**Explicação:** `cited_text` contém o texto do documento que sustenta uma afirmação.

---

### 5. Para que serve `document_index`?

A) Para identificar qual documento foi citado entre múltiplos documentos
B) Para calcular cosine similarity
C) Para definir o tamanho do chunk
D) Para alterar o idioma da resposta

**Resposta correta: A.**

**Explicação:** Quando há mais de um documento, `document_index` indica qual deles foi usado.

---

### 6. O que indica `document_title`?

A) O título do documento que Claude está citando
B) O nome da função Python
C) A pontuação BM25
D) O tipo de embedding

**Resposta correta: A.**

**Explicação:** Esse campo mostra o nome ou título do documento citado.

---

### 7. Em PDFs, o que indicam `start_page_number` e `end_page_number`?

A) As páginas inicial e final do trecho citado
B) O número de tokens da resposta
C) O tamanho da imagem em pixels
D) A versão do modelo Claude

**Resposta correta: A.**

**Explicação:** Esses campos ajudam o usuário a localizar a evidência no PDF.

---

### 8. Citations funcionam apenas com PDFs?

A) Não, também podem funcionar com texto puro
B) Sim, apenas com PDFs
C) Sim, apenas com imagens
D) Não, mas só funcionam com áudio

**Resposta correta: A.**

**Explicação:** O conteúdo mostra que citations também podem ser usadas com fontes em texto puro.

---

### 9. Em texto puro, o que pode substituir números de página?

A) Posições de caracteres no texto
B) Coordenadas GPS
C) Tamanho da API key
D) Temperature da resposta

**Resposta correta: A.**

**Explicação:** Como texto puro não tem páginas, a localização pode ser indicada por posições de caracteres.

---

### 10. Qual é uma boa prática ao usar citations?

A) Dar títulos claros aos documentos e exibir as fontes ao usuário
B) Omitir o documento de origem
C) Remover todas as citações da interface
D) Usar citations apenas em prompts criativos

**Resposta correta: A.**

**Explicação:** Títulos claros e exibição das fontes aumentam transparência e confiança.

---

### 11. Por que citations são importantes em aplicações profissionais?

A) Porque aumentam transparência, verificabilidade e auditabilidade
B) Porque eliminam completamente a necessidade de revisão
C) Porque reduzem automaticamente todos os custos
D) Porque substituem bancos vetoriais

**Resposta correta: A.**

**Explicação:** Citations ajudam o usuário a verificar a fonte e entender de onde veio cada informação.

---

### 12. Qual cuidado é correto sobre citations?

A) Uma resposta com citação ainda pode exigir revisão humana em contextos críticos
B) Uma citação garante que toda interpretação está correta
C) Citations impedem qualquer erro do modelo
D) Citations só servem para respostas sem documentos

**Resposta correta: A.**

**Explicação:** Citations melhoram verificabilidade, mas não garantem interpretação perfeita nem substituem revisão em casos sensíveis.
