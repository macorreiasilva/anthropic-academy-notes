# Suporte de imagem

```
notebooks\27-image_support.ipynb
```

## 1. Ideia central

O conteúdo apresenta o suporte a **imagens no Claude**, mostrando como enviar imagens em mensagens da API e como aplicar técnicas de prompt engineering para melhorar a qualidade das análises visuais.

A ideia central é:

**Claude pode receber imagens junto com texto, analisar o conteúdo visual e responder em texto, mas a qualidade da resposta depende muito da forma como o prompt é estruturado.**

Em tarefas visuais simples, como “o que há na imagem?”, prompts simples podem funcionar. Já em tarefas mais delicadas, como contar objetos ou avaliar risco em uma imagem de satélite, é melhor fornecer instruções detalhadas, etapas de análise e exemplos.

---

## 2. Principais conceitos

### Suporte a imagens

Claude pode analisar imagens enviadas pelo usuário.

Exemplos de tarefas possíveis:

* descrever o conteúdo de uma imagem;
* comparar múltiplas imagens;
* contar objetos;
* analisar diagramas;
* extrair informações visuais;
* avaliar riscos visuais;
* interpretar imagens de satélite;
* responder perguntas sobre uma imagem.

A interação funciona como uma conversa comum: o usuário envia uma mensagem com um **image block** e um **text block**, e Claude responde com um **text block**.

---

### Limites para envio de imagens

O conteúdo apresenta alguns limites importantes:

| Limite                                 |                            Valor |
| -------------------------------------- | -------------------------------: |
| Máximo de imagens em uma única request |                  até 100 imagens |
| Tamanho máximo por imagem              |                             5 MB |
| Uma única imagem                       |    até 8000 px de altura/largura |
| Múltiplas imagens                      |    até 2000 px de altura/largura |
| Formas de envio                        |                    base64 ou URL |
| Cálculo aproximado de tokens           | `(largura px × altura px) / 750` |

A fórmula mostrada é:

```text
tokens = (width px × height px) / 750
```

Ou seja, imagens maiores consomem mais tokens.

---

### Estrutura da mensagem com imagem

Para enviar uma imagem em base64, primeiro o arquivo é lido em bytes e codificado:

```python
with open("image.png", "rb") as f:
    image_bytes = base64.standard_b64encode(
        f.read()
    ).decode("utf-8")
```

Depois, a mensagem do usuário inclui dois blocos:

1. um bloco de imagem;
2. um bloco de texto com a pergunta.

Exemplo:

```python
add_user_message(messages, [
    {
        "type": "image",
        "source": {
            "type": "base64",
            "media_type": "image/png",
            "data": image_bytes,
        }
    },
    {
        "type": "text",
        "text": "What do you see in this image?"
    }
])
```

Esse formato permite que Claude receba a imagem e a instrução textual na mesma mensagem.

---

### Fluxo da mensagem

O fluxo é:

```text
Servidor
↓
User Message:
  - Image Block
  - Text Block
↓
Claude
↓
Assistant Message:
  - Text Block
```

Exemplo:

```text
User:
[imagem]
"What do you see in this image?"

Claude:
"I see a tall tree..."
```

Claude não responde com uma imagem nesse caso. Ele analisa a imagem e devolve texto.

---

### Prompt engineering para imagens

Uma parte essencial do conteúdo é que **as mesmas técnicas de prompt engineering usadas para texto também se aplicam a imagens**.

Isso inclui:

* ser claro e direto;
* fornecer instruções específicas;
* dividir tarefas complexas em etapas;
* pedir verificação;
* usar exemplos one-shot ou multi-shot;
* definir critérios de avaliação;
* especificar o formato da resposta.

Prompts simples podem gerar respostas erradas em tarefas visuais difíceis.

---

### Exemplo: contagem de bolinhas de gude

O conteúdo mostra um exemplo em que a imagem tem 12 bolinhas de gude.

Prompt simples:

```text
How many marbles are in this image?
```

Claude pode responder incorretamente:

```text
I count 13 marbles in this image
```

Isso demonstra que visão computacional com LLMs pode cometer erros em contagem visual, especialmente quando objetos se sobrepõem ou são parecidos.

---

### Melhorando com metodologia

Um prompt melhor dá instruções passo a passo:

```text
Analyze this image of marbles and determine the exact count using this methodology:

1. Begin by identifying each unique marble one at a time. Assign each a number as you identify it.
2. Verify your result by counting with a different method. Start from the bottom-left corner and work row by row, from left to right.

What is the exact, verified number of marbles in this image?
```

Com essa abordagem, Claude chega ao resultado correto:

```text
I count 12 marbles in this image
```

O conceito demonstrado é que **instruções de análise e verificação aumentam a precisão em tarefas visuais**.

---

### Exemplo one-shot com imagem

O conteúdo também mostra uma abordagem com exemplo.

Primeiro, o prompt apresenta uma imagem de referência:

```text
The image above has 11 marbles in it.
```

Depois, apresenta a imagem-alvo e pergunta:

```text
How many marbles are in this image?
```

Claude responde corretamente:

```text
I count 12 marbles in this image
```

Esse é um exemplo de **one-shot prompting visual**: fornecer um exemplo de entrada e resposta correta antes da tarefa principal.

---

### Caso prático: avaliação de risco de incêndio

O conteúdo apresenta um caso realista: **Fire Risk Assessments**.

Em algumas regiões dos EUA, seguradoras podem exigir que moradores cortem ou removam árvores ao redor da residência para reduzir risco de incêndio.

Problema:

* enviar um inspetor físico para cada imóvel é caro;
* é possível usar imagens de satélite recentes e de alta resolução;
* Claude pode analisar visualmente fatores de risco.

Elementos analisados na imagem:

* árvores densas e muito próximas;
* acesso difícil à residência;
* galhos sobre o telhado;
* vegetação criando caminho contínuo para o fogo.

---

### Prompt detalhado para risco de incêndio

Em vez de pedir apenas:

```text
Provide a fire risk score.
```

o conteúdo recomenda um prompt estruturado com etapas.

A análise deve incluir:

1. identificação da residência principal;
2. análise de galhos sobre o telhado;
3. avaliação de vulnerabilidade ao fogo;
4. identificação de espaço defensável;
5. atribuição de nota de risco de 1 a 4.

Exemplo de escala:

| Rating | Significado    |
| -----: | -------------- |
|      1 | Baixo risco    |
|      2 | Risco moderado |
|      3 | Alto risco     |
|      4 | Risco severo   |

Esse exemplo mostra que tarefas visuais complexas precisam de critérios explícitos e uma metodologia clara.

---

## 3. Pontos importantes para certificação

* Claude aceita imagens em mensagens via **image blocks**.
* Uma mensagem pode conter imagem e texto juntos.
* Claude responde com texto analisando a imagem.
* As imagens podem ser enviadas em base64 ou por URL.
* O limite citado é de até 100 imagens por request.
* Cada imagem pode ter até 5 MB.
* Para uma imagem, o limite de altura/largura é 8000 px.
* Para múltiplas imagens, o limite de altura/largura é 2000 px.
* Imagens contam como tokens.
* Fórmula apresentada: `(width px × height px) / 750`.
* Prompts simples podem falhar em tarefas visuais.
* Prompt engineering também se aplica a imagens.
* Diretrizes, passos de análise e exemplos melhoram a precisão.
* One-shot e multi-shot prompting podem ser usados com imagens.
* Para contagem de objetos, pedir verificação com outro método pode melhorar resultados.
* Para tarefas de risco, forneça critérios explícitos de avaliação.
* Claude pode analisar imagens de satélite, mas a resposta deve ser orientada por instruções claras.

---

## 4. Termos-chave

**Image block**
Bloco de conteúdo usado para enviar uma imagem a Claude.

**Text block**
Bloco de texto usado para fazer uma pergunta ou fornecer instruções junto da imagem.

**Base64 encoding**
Forma de codificar arquivos binários, como imagens, em texto para envio via API.

**Media type**
Tipo MIME da imagem, como `image/png`.

**Vision capabilities**
Capacidade de Claude de analisar informações visuais.

**Image tokens**
Tokens consumidos por uma imagem, estimados com base em suas dimensões.

**One-shot prompting**
Uso de um único exemplo para orientar a resposta do modelo.

**Multi-shot prompting**
Uso de múltiplos exemplos para ensinar padrões ou casos difíceis.

**Step-by-step analysis**
Instrução para o modelo analisar a imagem seguindo etapas.

**Visual counting**
Tarefa de contar objetos presentes em uma imagem.

**Fire risk assessment**
Avaliação de risco de incêndio com base em fatores visuais, como árvores próximas à residência.

**Defensible space**
Área ao redor de uma residência com menor carga de vegetação, ajudando a reduzir risco de propagação do fogo.

---

## 5. Boas práticas

Use prompts estruturados para tarefas visuais complexas.

Em vez de perguntar apenas:

```text
How many marbles are in this image?
```

prefira orientar o processo:

```text
Identify each object one by one, assign numbers, then verify the count using a second method.
```

Para análises de risco ou decisão, forneça:

* critérios claros;
* escala de classificação;
* etapas de análise;
* formato esperado da resposta;
* instruções para justificar a avaliação.

Também é recomendado:

* reduzir o tamanho da imagem quando possível;
* evitar imagens desnecessariamente grandes;
* considerar o custo em tokens;
* usar exemplos quando houver casos ambíguos;
* testar prompts com imagens variadas;
* validar resultados em tarefas sensíveis.

---

## 6. Limitações, riscos e cuidados

Claude pode errar em tarefas visuais, especialmente quando há:

* objetos sobrepostos;
* baixa resolução;
* sombras;
* ângulos difíceis;
* imagens comprimidas;
* objetos muito pequenos;
* muitos elementos parecidos;
* necessidade de contagem exata.

O exemplo das bolinhas de gude mostra isso: um prompt simples levou Claude a contar 13 quando havia 12.

Em casos sensíveis, como avaliação de risco de incêndio para seguros, a resposta de Claude deve ser tratada como apoio à decisão, não como verdade absoluta sem validação.

Cuidados importantes:

* não depender apenas de prompt simples;
* usar critérios explícitos;
* pedir análise verificável;
* considerar revisão humana em decisões de alto impacto;
* proteger imagens sensíveis ou privadas;
* respeitar limites de tamanho e quantidade;
* lembrar que imagens grandes aumentam custo.

---

## 7. Resumo para revisão rápida

* Claude pode analisar imagens.
* Imagens são enviadas em `image blocks`.
* A pergunta entra em um `text block`.
* A resposta vem como texto.
* Imagens podem ser base64 ou URL.
* Máximo citado: 100 imagens por request.
* Tamanho máximo: 5 MB por imagem.
* Uma imagem: até 8000 px.
* Múltiplas imagens: até 2000 px.
* Tokens da imagem dependem de largura e altura.
* Fórmula: `(width × height) / 750`.
* Prompt engineering também vale para imagens.
* Prompts simples podem gerar erros.
* Etapas de análise melhoram precisão.
* One-shot e multi-shot ajudam em tarefas visuais.
* Contagem de objetos deve incluir verificação.
* Avaliações de risco devem ter critérios explícitos.

---

## 8. Perguntas simuladas

### 1. Como uma imagem é enviada para Claude na API?

A) Como um `image block` dentro da mensagem do usuário
B) Apenas como texto descritivo
C) Como uma API key
D) Como parâmetro de temperature

**Resposta correta: A.**

**Explicação:** Imagens são enviadas como blocos de imagem, geralmente junto com um bloco de texto.

---

### 2. Qual bloco contém a pergunta sobre a imagem?

A) `text block`
B) `signature block`
C) `thinking block`
D) `stop block`

**Resposta correta: A.**

**Explicação:** A imagem vai no `image block`, e a pergunta ou instrução vai no `text block`.

---

### 3. Quais são duas formas citadas de incluir imagens?

A) Base64 ou URL
B) Apenas XML
C) Apenas JSON Schema
D) Apenas Markdown

**Resposta correta: A.**

**Explicação:** O conteúdo informa que imagens podem ser enviadas por codificação base64 ou por URL.

---

### 4. Qual é o tamanho máximo citado por imagem?

A) 5 MB
B) 50 MB
C) 500 KB
D) 100 MB

**Resposta correta: A.**

**Explicação:** O conteúdo indica limite máximo de 5 MB por imagem.

---

### 5. Quantas imagens podem ser enviadas em uma única request, segundo o conteúdo?

A) Até 100 imagens
B) Apenas 1 imagem
C) Até 5 imagens
D) Ilimitadas

**Resposta correta: A.**

**Explicação:** O limite citado é de até 100 imagens entre todas as mensagens da request.

---

### 6. Qual é a fórmula aproximada de tokens para imagem apresentada?

A) `(width px × height px) / 750`
B) `max_tokens / temperature`
C) `width px + API key`
D) `height px × 100`

**Resposta correta: A.**

**Explicação:** O consumo de tokens da imagem depende das dimensões em pixels.

---

### 7. Por que um prompt simples pode ser ruim para contar objetos em imagem?

A) Porque Claude pode cometer erros sem metodologia de análise
B) Porque Claude não aceita imagens
C) Porque imagens não contam como tokens
D) Porque o text block é proibido

**Resposta correta: A.**

**Explicação:** O exemplo das bolinhas mostra que Claude pode errar a contagem sem instruções detalhadas.

---

### 8. Qual técnica melhorou a contagem das bolinhas de gude?

A) Pedir identificação individual e verificação por outro método
B) Remover a imagem
C) Aumentar a temperature
D) Enviar apenas a URL sem pergunta

**Resposta correta: A.**

**Explicação:** O prompt estruturado orienta Claude a contar cuidadosamente e verificar o resultado.

---

### 9. O que é one-shot prompting visual?

A) Fornecer um exemplo com imagem e resposta correta antes da tarefa principal
B) Enviar uma imagem sem texto
C) Usar apenas uma palavra no prompt
D) Apagar exemplos anteriores

**Resposta correta: A.**

**Explicação:** O exemplo com uma imagem que tinha 11 bolinhas ajuda Claude a entender a tarefa antes de contar a imagem-alvo.

---

### 10. No exemplo de avaliação de risco de incêndio, o que Claude deve analisar?

A) Árvores próximas, galhos sobre a residência, acesso e espaço defensável
B) Apenas a cor do telhado
C) Apenas o tamanho do arquivo
D) Apenas o nome da cidade

**Resposta correta: A.**

**Explicação:** A avaliação considera fatores visuais que podem aumentar risco de incêndio.

---

### 11. Qual escala de risco de incêndio foi apresentada?

A) De 1 a 4
B) De 0 a 100
C) De A a Z
D) Apenas baixo ou alto

**Resposta correta: A.**

**Explicação:** O prompt detalhado pede uma classificação de Fire Risk Rating de 1 a 4.

---

### 12. Qual frase resume melhor o suporte a imagens no Claude?

A) Claude pode analisar imagens, mas bons resultados dependem de prompts claros, estruturados e com critérios.
B) Claude sempre conta objetos perfeitamente com prompts simples.
C) Imagens não têm impacto em tokens.
D) Não é possível enviar imagem e texto na mesma mensagem.

**Resposta correta: A.**

**Explicação:** O conteúdo enfatiza que as capacidades visuais funcionam melhor com boas técnicas de prompt engineering.
