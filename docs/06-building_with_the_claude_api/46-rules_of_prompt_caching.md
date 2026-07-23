# Regras de cache de prompts

## 1. Ideia central

O conteúdo aprofunda as **regras do Prompt Caching no Claude**, mostrando que o cache não acontece automaticamente. Para usar esse recurso, é necessário marcar explicitamente onde o cache deve ser criado usando **cache breakpoints**.

A ideia central é:

**Prompt caching só funciona bem quando você coloca breakpoints em partes estáveis e repetidas da request, como prompts longos, system prompts e definições de ferramentas.**

---

## 2. Principais conceitos

### Prompt caching não é automático

Claude não salva automaticamente todo o trabalho feito em uma request.

Para que o processamento seja cacheado, você precisa adicionar manualmente um **cache breakpoint** em um bloco da mensagem.

Exemplo conceitual:

```json
"cache_control": {
  "type": "ephemeral"
}
```

Isso indica que o conteúdo até aquele ponto pode ser salvo temporariamente no cache.

---

### Cache breakpoint

Um **cache breakpoint** é um marcador colocado em um bloco da request para dizer:

> “Cacheie todo o processamento feito até aqui.”

Tudo que vem **antes e incluindo o breakpoint** pode ser cacheado.

Tudo que vem **depois do breakpoint** não entra naquele cache.

Exemplo:

```json
{
  "type": "text",
  "text": "<Long prompt>",
  "cache_control": {
    "type": "ephemeral"
  }
}
```

Esse formato diz que o bloco de texto longo deve participar do cache.

---

### O conteúdo precisa ser idêntico

O cache só será reutilizado em uma follow-up request se o conteúdo até o breakpoint for **idêntico** ao conteúdo enviado anteriormente.

Pequenas mudanças podem invalidar o cache, por exemplo:

* adicionar uma palavra;
* mudar a ordem dos blocos;
* alterar uma ferramenta;
* modificar o system prompt;
* trocar uma parte do texto antes do breakpoint.

Isso é importante porque o cache depende de correspondência exata.

---

### Forma curta vs forma longa de text block

O conteúdo mostra duas formas de escrever uma mensagem de texto.

#### Forma curta

```json
{
  "role": "user",
  "content": "Hi there!"
}
```

Essa forma é simples, mas não permite adicionar `cache_control`.

#### Forma longa

```json
{
  "role": "user",
  "content": [
    {
      "type": "text",
      "text": "<Long prompt>",
      "cache_control": {
        "type": "ephemeral"
      }
    }
  ]
}
```

Para usar cache breakpoint, é necessário usar a **forma longa**, porque ela permite adicionar metadados ao bloco.

---

### Cache pode abranger várias mensagens

Cache breakpoints não se limitam a um único bloco isolado.

Eles podem abranger:

* mensagens de usuário;
* mensagens do assistant;
* múltiplos blocos;
* histórico de conversa até certo ponto.

Se você coloca o breakpoint em uma mensagem posterior, o cache pode incluir tudo que veio antes dela.

Isso é útil quando você quer cachear um contexto de conversa inteiro até determinado ponto.

---

### Localização dos breakpoints

Cache breakpoints não são restritos a blocos de texto.

Eles também podem ser usados em:

* system prompts;
* tool definitions;
* image blocks;
* tool use blocks;
* tool result blocks.

As melhores oportunidades geralmente são:

* prompts de sistema longos;
* listas grandes de ferramentas;
* documentos grandes;
* instruções fixas;
* contexto que muda pouco entre chamadas.

System prompts e tool definitions são bons candidatos porque normalmente permanecem iguais entre requests.

---

### Ordem do cache

Claude processa partes da request em uma ordem específica:

1. ferramentas;
2. system prompt;
3. mensagens.

Essa ordem importa porque o conteúdo precisa permanecer igual até o breakpoint.

Exemplo:

```text
Tool #1
Tool #2
Tool #3  ← Cache Breakpoint
System Prompt
User Message
Assistant Message
User Message
```

Se a request seguinte inclui as mesmas ferramentas até o breakpoint, essa parte pode ser reutilizada.

---

### Limite de breakpoints

O conteúdo menciona que é possível adicionar até **quatro cache breakpoints** no total.

Isso permite cachear diferentes partes da request, por exemplo:

* um breakpoint nas ferramentas;
* outro no system prompt;
* outro em parte do histórico;
* outro em um documento longo.

A ideia é cachear partes estáveis sem exigir que a request inteira seja idêntica.

---

### Tamanho mínimo de conteúdo

Para que o conteúdo seja cacheado, ele precisa ter pelo menos **1024 tokens**.

Esse limite considera a soma dos blocos/mensagens que você está tentando cachear.

Exemplo que não será cacheado:

```text
Hi there!
```

É curto demais.

Mas um conteúdo longo, ou repetido muitas vezes, pode ultrapassar 1024 tokens e se tornar elegível para cache.

---

## 3. Pontos importantes para certificação

* Prompt caching não acontece automaticamente.
* É necessário adicionar manualmente um cache breakpoint.
* O breakpoint usa `cache_control`.
* O tipo apresentado é `"ephemeral"`.
* Tudo antes e incluindo o breakpoint pode ser cacheado.
* Conteúdo depois do breakpoint não entra naquele cache.
* Para reutilizar cache, o conteúdo até o breakpoint precisa ser idêntico.
* A forma curta de text block não permite `cache_control`.
* Para usar cache, utilize a forma longa do bloco.
* Breakpoints podem abranger múltiplas mensagens.
* Mensagens do assistant também podem ser cacheadas.
* Breakpoints podem ser usados em system prompts e tool definitions.
* System prompts e tool definitions são boas oportunidades de cache porque mudam pouco.
* A ordem de processamento é: tools, system prompt e depois messages.
* É possível usar até quatro cache breakpoints.
* O conteúdo precisa ter pelo menos 1024 tokens para ser cacheado.
* Prompt caching é mais útil quando há conteúdo longo, estável e repetido.

---

## 4. Termos-chave

**Prompt caching**
Recurso que reutiliza processamento de partes repetidas do prompt para reduzir custo e latência.

**Cache breakpoint**
Marcador que indica até onde o conteúdo deve ser cacheado.

**`cache_control`**
Campo usado para configurar o comportamento de cache em um bloco.

**`ephemeral`**
Tipo de cache temporário usado no exemplo.

**Text block**
Bloco de conteúdo textual dentro de uma mensagem.

**Shorthand text block**
Forma curta de escrever uma mensagem textual, sem espaço para `cache_control`.

**Longhand text block**
Forma expandida de escrever um bloco de texto, permitindo campos adicionais como `cache_control`.

**System prompt**
Instrução de alto nível que define comportamento, papel ou estilo do modelo.

**Tool definitions**
Definições de ferramentas disponíveis para Claude usar.

**Cache ordering**
Ordem em que Claude considera partes da request para caching.

**Minimum content length**
Tamanho mínimo necessário para que o conteúdo seja elegível ao cache.

---

## 5. Boas práticas

Use cache breakpoints em partes da request que são:

* longas;
* repetidas;
* estáveis;
* caras de processar;
* pouco prováveis de mudar.

Bons candidatos:

```text
System prompt longo
Lista grande de ferramentas
Documento longo
Contexto fixo de uma conversa
Instruções detalhadas reutilizadas
```

Prefira colocar o breakpoint depois de uma parte grande e estável.

Exemplo:

```json
{
  "type": "text",
  "text": "Você é um assistente especializado em análise de documentos...",
  "cache_control": {
    "type": "ephemeral"
  }
}
```

Também é uma boa prática manter a ordem dos blocos consistente entre requests. Se você muda a ordem das ferramentas ou altera o system prompt, pode perder o benefício do cache.

---

## 6. Limitações, riscos e cuidados

Prompt caching tem regras rígidas.

Principais cuidados:

* o cache não é automático;
* conteúdo precisa ser idêntico até o breakpoint;
* pequenas alterações podem invalidar o cache;
* há limite de até quatro breakpoints;
* conteúdo curto não é cacheado;
* o mínimo é 1024 tokens;
* cache é temporário, não memória permanente;
* cache mal posicionado pode gerar pouco ou nenhum benefício.

Um erro comum é colocar o breakpoint depois de conteúdo que muda com frequência.

Exemplo ruim:

```text
User-specific question
Timestamp atual
Dados dinâmicos
Cache breakpoint
```

Se essas partes mudam a cada request, o cache dificilmente será reutilizado.

Outro erro é tentar cachear prompts muito curtos. Mesmo com `cache_control`, se o conteúdo não atingir o mínimo necessário, ele não será salvo.

---

## 7. Resumo para revisão rápida

* Prompt caching exige configuração manual.
* Use `cache_control` para criar cache breakpoints.
* O exemplo usa `"type": "ephemeral"`.
* Tudo até o breakpoint pode ser cacheado.
* O conteúdo após o breakpoint não entra no cache.
* Para reutilizar cache, o conteúdo precisa ser idêntico.
* Use a forma longa de text block.
* A forma curta não permite cache breakpoint.
* Breakpoints podem atravessar múltiplas mensagens.
* Assistant messages também podem ser cacheadas.
* System prompts e tool definitions são ótimos candidatos.
* A ordem é: tools, system prompt, messages.
* Até quatro breakpoints podem ser usados.
* O conteúdo precisa ter pelo menos 1024 tokens.
* Cache é útil para conteúdo longo, repetido e estável.

---

## 8. Perguntas simuladas

### 1. O prompt caching em Claude acontece automaticamente?

A) Sim, Claude sempre cacheia todas as mensagens
B) Não, é necessário adicionar manualmente um cache breakpoint
C) Sim, mas apenas para imagens
D) Não, porque Claude não suporta caching

**Resposta correta: B.**

**Explicação:** O conteúdo deixa claro que o trabalho feito nas mensagens não é cacheado automaticamente. É preciso adicionar um breakpoint.

---

### 2. Qual campo é usado para marcar um cache breakpoint?

A) `temperature`
B) `cache_control`
C) `max_tokens`
D) `stop_reason`

**Resposta correta: B.**

**Explicação:** O campo `cache_control` é usado para indicar que aquele bloco deve atuar como ponto de cache.

---

### 3. O que é cacheado quando um breakpoint é colocado em um bloco?

A) Apenas o conteúdo depois do breakpoint
B) Tudo antes e incluindo o breakpoint
C) Apenas a resposta final do assistant
D) Apenas a API key

**Resposta correta: B.**

**Explicação:** O cache cobre o processamento feito até o ponto do breakpoint, incluindo o próprio bloco marcado.

---

### 4. O que precisa acontecer para uma follow-up request reutilizar o cache?

A) O conteúdo até o breakpoint precisa ser idêntico
B) A temperature precisa ser 1.0
C) O usuário precisa enviar uma imagem
D) O modelo precisa usar RAG

**Resposta correta: A.**

**Explicação:** O cache só é reutilizado se o conteúdo até e incluindo o breakpoint for igual ao da request anterior.

---

### 5. Por que a forma curta de text block não serve para cache breakpoint?

A) Porque não permite adicionar `cache_control`
B) Porque só funciona com PDFs
C) Porque não aceita mensagens de usuário
D) Porque sempre ultrapassa 1024 tokens

**Resposta correta: A.**

**Explicação:** A forma curta é apenas uma string simples e não permite adicionar metadados ao bloco.

---

### 6. Qual formato deve ser usado para adicionar `cache_control` a um bloco de texto?

A) Forma longa, com `content` como lista de blocos
B) Forma curta, com `content` como string
C) Apenas Markdown
D) Apenas CSV

**Resposta correta: A.**

**Explicação:** A forma longa permite definir o bloco como `{"type": "text", "text": "...", "cache_control": ...}`.

---

### 7. Cache breakpoints podem atravessar várias mensagens?

A) Sim
B) Não, só funcionam em uma única mensagem
C) Não, só funcionam em system prompts
D) Apenas quando não há assistant message

**Resposta correta: A.**

**Explicação:** O conteúdo mostra que breakpoints podem abranger user messages, assistant messages e múltiplos blocos anteriores.

---

### 8. Quais partes são bons candidatos para caching?

A) System prompts longos e tool definitions
B) Apenas mensagens curtas como “Hi there!”
C) Apenas respostas criativas
D) Apenas valores de temperature

**Resposta correta: A.**

**Explicação:** System prompts e definições de ferramentas raramente mudam entre requests, tornando-se bons candidatos para cache.

---

### 9. Qual é a ordem de processamento relevante para cache ordering?

A) Tools, system prompt, messages
B) Messages, tools, system prompt
C) Temperature, max tokens, response
D) PDF, image, text

**Resposta correta: A.**

**Explicação:** O conteúdo mostra que Claude considera primeiro ferramentas, depois system prompt e depois mensagens.

---

### 10. Quantos cache breakpoints podem ser usados, segundo o conteúdo?

A) Até quatro
B) Apenas um
C) Ilimitados
D) Nenhum

**Resposta correta: A.**

**Explicação:** O material menciona que é possível adicionar até quatro breakpoints.

---

### 11. Qual é o tamanho mínimo para que um conteúdo seja cacheado?

A) 1024 tokens
B) 10 tokens
C) 100 caracteres
D) 1 milhão de tokens

**Resposta correta: A.**

**Explicação:** O conteúdo precisa ter pelo menos 1024 tokens para ser elegível ao cache.

---

### 12. Uma mensagem como “Hi there!” seria cacheada sozinha?

A) Não, porque é curta demais
B) Sim, porque qualquer texto é cacheado
C) Sim, se estiver em português
D) Sim, se usar temperature baixa

**Resposta correta: A.**

**Explicação:** “Hi there!” não atinge o mínimo de 1024 tokens necessário para caching.

---

### 13. Qual é um erro comum ao usar prompt caching?

A) Colocar o breakpoint depois de conteúdo que muda com frequência
B) Usar conteúdo longo e estável
C) Cachear tool definitions fixas
D) Cachear um system prompt grande

**Resposta correta: A.**

**Explicação:** Se o conteúdo anterior ao breakpoint muda, o cache não será reutilizado.

---

### 14. Qual frase resume melhor as regras de prompt caching?

A) Prompt caching exige breakpoints manuais, conteúdo idêntico até o breakpoint e tamanho mínimo de 1024 tokens.
B) Prompt caching salva automaticamente qualquer mensagem curta.
C) Prompt caching só funciona com imagens e PDFs.
D) Prompt caching substitui system prompts e ferramentas.

**Resposta correta: A.**

**Explicação:** Essa alternativa reúne os pontos centrais: configuração manual, igualdade do conteúdo e tamanho mínimo.
