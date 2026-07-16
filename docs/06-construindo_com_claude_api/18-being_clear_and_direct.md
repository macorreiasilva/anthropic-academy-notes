# Sendo claro e direto

## 1. Ideia central

O conteúdo explica uma técnica básica, mas muito importante de **prompt engineering**: ser **claro e direto**, especialmente na **primeira linha do prompt**.

A ideia central é que a primeira frase define o rumo da resposta. Se ela for vaga, Claude precisa “adivinhar” o que você quer. Se for clara e direta, Claude entende melhor a tarefa e tende a produzir uma resposta mais útil.

Em resumo:

**Comece o prompt com uma instrução direta, usando linguagem simples e um verbo de ação.**

---

## 2. Principais conceitos

### A importância da primeira linha do prompt

A primeira linha do prompt é considerada a parte mais importante da solicitação porque ela estabelece o objetivo principal.

Ela deve responder rapidamente:

* o que Claude deve fazer;
* qual saída deve produzir;
* qual é o escopo da tarefa;
* quais restrições principais devem ser respeitadas.

Uma primeira linha fraca pode deixar a tarefa ambígua. Uma primeira linha forte orienta o modelo desde o início.

---

### Clareza

Ser **claro** significa usar uma linguagem simples, objetiva e sem ambiguidade.

Um prompt claro:

* usa palavras fáceis de entender;
* declara exatamente o que deve ser feito;
* evita rodeios;
* não obriga Claude a inferir a intenção do usuário.

Exemplo ruim:

```text
I need to know about those things people put on their roofs that use sun - those solar panel things, I think they're called.
```

Esse prompt é confuso e indireto.

Versão melhor:

```text
Write three paragraphs about how solar panels work.
```

Essa versão é melhor porque diz exatamente:

* o tema: painéis solares;
* a tarefa: escrever;
* o formato: três parágrafos;
* o foco: como funcionam.

---

### Direção

Ser **direto** significa estruturar o pedido como uma instrução, não como uma pergunta vaga.

O conteúdo recomenda usar verbos de ação no início do prompt, como:

* **Write**
* **Create**
* **Generate**
* **Identify**
* **Summarize**
* **Explain**
* **Compare**

Exemplo menos direto:

```text
I was reading about renewable energy and geothermal energy sounds neat. What countries use it?
```

Versão melhor:

```text
Identify three countries that use geothermal energy. Include generation stats for each.
```

A versão melhor é mais útil porque define:

* a ação: identificar;
* a quantidade: três países;
* o tema: energia geotérmica;
* o detalhe esperado: estatísticas de geração.

---

### Usar instruções em vez de perguntas

Uma recomendação importante é transformar perguntas abertas em comandos claros.

Perguntas podem funcionar, mas instruções diretas geralmente dão mais controle.

Exemplo de pergunta:

```text
What should this person eat?
```

Versão melhor:

```text
Generate a one-day meal plan for an athlete that meets their dietary restrictions.
```

A versão melhor especifica:

* ação: gerar;
* saída esperada: plano alimentar;
* duração: um dia;
* público: atleta;
* restrição: deve respeitar limitações alimentares.

---

### Aplicação no exemplo do plano alimentar

O prompt inicial era:

```text
What should this person eat?
```

Esse prompt é fraco porque é muito genérico. Ele não deixa claro:

* que tipo de resposta é esperada;
* se deve ser um plano alimentar completo;
* se deve considerar restrições;
* se deve ser para um dia;
* se o usuário é atleta.

A versão melhorada foi:

```text
Generate a one-day meal plan for an athlete that meets their dietary restrictions.
```

Essa mudança melhora a qualidade porque Claude recebe uma tarefa mais específica desde o começo.

---

### Impacto na avaliação

O conteúdo mostra que apenas melhorar a primeira linha do prompt já aumentou a pontuação da avaliação.

A pontuação foi de:

```text
2.32 → 3.92
```

Isso demonstra que pequenas mudanças estruturais no prompt podem gerar melhorias mensuráveis.

O ponto importante é que prompt engineering deve ser avaliado com dados, não apenas por impressão subjetiva.

---

## 3. Pontos importantes para certificação

* A primeira linha do prompt é uma das partes mais importantes.
* Prompts claros e diretos tendem a gerar respostas melhores.
* Clareza significa usar linguagem simples e sem ambiguidade.
* Direção significa usar instruções objetivas, não perguntas vagas.
* Bons prompts começam com verbos de ação.
* Exemplos de verbos úteis: **Write, Create, Generate, Identify, Explain, Summarize**.
* Um prompt deve dizer exatamente o que Claude deve produzir.
* Instruções específicas reduzem a necessidade de inferência pelo modelo.
* Trocar uma pergunta vaga por uma instrução direta pode melhorar a pontuação em avaliações.
* No exemplo, a pontuação subiu de **2.32 para 3.92** apenas com uma primeira linha melhor.
* Claude funciona melhor quando recebe direção clara, como um assistente capaz que precisa de instruções objetivas.

---

## 4. Termos-chave

**Clear prompt**
Prompt claro, escrito com linguagem simples e objetivo bem definido.

**Direct prompt**
Prompt direto, estruturado como uma instrução explícita.

**First line**
Primeira linha do prompt, responsável por estabelecer a tarefa principal.

**Action verb**
Verbo de ação usado para começar uma instrução, como “Generate”, “Write” ou “Identify”.

**Ambiguity**
Ambiguidade; quando uma instrução pode ser interpretada de várias formas.

**Instruction**
Comando direto que diz ao modelo o que fazer.

**Prompt refinement**
Processo de melhorar um prompt para gerar respostas mais confiáveis.

**Evaluation score**
Pontuação usada para medir a qualidade do prompt em um processo de avaliação.

**Task framing**
Forma como a tarefa é apresentada ao modelo.

---

## 5. Boas práticas

* Comece o prompt com uma instrução clara.
* Use verbos de ação logo no início.
* Evite começar com contexto longo antes de dizer a tarefa.
* Troque perguntas vagas por comandos objetivos.
* Especifique o formato ou tipo de saída desejado.
* Informe restrições importantes logo no começo.
* Use linguagem simples e direta.
* Remova palavras desnecessárias.
* Evite fazer Claude adivinhar sua intenção.
* Após mudar o prompt, rode uma avaliação para medir se houve melhora.

Exemplo de estrutura recomendada:

```text
Generate a [tipo de saída] for [público/contexto] that [restrição principal].
```

Exemplo:

```text
Generate a one-day meal plan for an athlete that meets their dietary restrictions.
```

---

## 6. Limitações, riscos e cuidados

Ser claro e direto melhora muito o prompt, mas não resolve todos os problemas sozinho.

Um prompt pode ter uma primeira linha boa e ainda assim precisar de:

* exemplos;
* critérios de qualidade;
* formato de saída;
* restrições adicionais;
* system prompt;
* avaliação com dataset;
* refinamento iterativo.

Outro cuidado é não ser direto demais a ponto de omitir contexto importante. O ideal é começar com uma instrução clara e depois adicionar detalhes necessários.

Também é importante evitar instruções vagas como:

```text
Help me with this.
```

ou:

```text
Tell me about it.
```

Essas frases exigem que Claude deduza a intenção, o que aumenta o risco de respostas genéricas.

---

## 7. Resumo para revisão rápida

* A primeira linha do prompt é crucial.
* Comece com uma instrução clara.
* Use linguagem simples.
* Use verbos de ação.
* Prefira comandos a perguntas vagas.
* Diga exatamente o que Claude deve produzir.
* Inclua o escopo principal logo no início.
* Evite rodeios e contexto desnecessário antes da tarefa.
* Prompt ruim: “What should this person eat?”
* Prompt melhor: “Generate a one-day meal plan for an athlete that meets their dietary restrictions.”
* A mudança clara e direta melhorou a pontuação de **2.32 para 3.92**.
* Clareza e direção são fundamentos de prompt engineering.

---

## 8. Perguntas simuladas

### 1. Qual parte do prompt o conteúdo destaca como especialmente importante?

A) A última palavra
B) A primeira linha
C) A quantidade de emojis
D) O nome do arquivo

**Resposta correta: B.**

**Explicação:** A primeira linha define a tarefa principal e orienta tudo que vem depois.

---

### 2. O que significa ser claro em um prompt?

A) Usar linguagem simples e dizer exatamente o que se deseja
B) Usar frases longas e indiretas
C) Esconder a tarefa até o final
D) Fazer várias perguntas sem contexto

**Resposta correta: A.**

**Explicação:** Clareza significa reduzir ambiguidade e comunicar a tarefa de forma simples.

---

### 3. O que significa ser direto em um prompt?

A) Escrever o prompt como uma instrução objetiva
B) Evitar dizer o que se quer
C) Usar apenas perguntas abertas
D) Pedir para Claude adivinhar o objetivo

**Resposta correta: A.**

**Explicação:** Ser direto envolve usar comandos claros, normalmente iniciados por verbos de ação.

---

### 4. Qual é um bom exemplo de verbo de ação para iniciar um prompt?

A) Talvez
B) Generate
C) Coisa
D) Interessante

**Resposta correta: B.**

**Explicação:** “Generate” é um verbo de ação que indica claramente o que Claude deve fazer.

---

### 5. Qual prompt é mais claro?

A) “I need to know about those roof sun things.”
B) “Write three paragraphs about how solar panels work.”
C) “Can you maybe talk about energy stuff?”
D) “Those things, explain.”

**Resposta correta: B.**

**Explicação:** Ele especifica o tema, a tarefa e o formato esperado.

---

### 6. Qual prompt é mais direto?

A) “I was reading about renewable energy and geothermal energy sounds neat. What countries use it?”
B) “Identify three countries that use geothermal energy. Include generation stats for each.”
C) “Tell me some things about heat from Earth maybe.”
D) “What about geothermal?”

**Resposta correta: B.**

**Explicação:** Ele começa com uma ação clara, define quantidade e pede dados específicos.

---

### 7. Qual era o problema do prompt “What should this person eat?”

A) Ele era muito específico
B) Ele era vago e não definia o tipo de resposta esperada
C) Ele continha JSON inválido
D) Ele usava muitos critérios técnicos

**Resposta correta: B.**

**Explicação:** O prompt não especificava que a resposta deveria ser um plano alimentar de um dia para atleta com restrições.

---

### 8. Qual foi a versão melhorada do prompt de alimentação?

A) “Food?”
B) “Generate a one-day meal plan for an athlete that meets their dietary restrictions.”
C) “Tell me anything about nutrition.”
D) “What do athletes eat sometimes?”

**Resposta correta: B.**

**Explicação:** Essa versão define ação, saída, duração, público e restrição.

---

### 9. Qual foi o impacto da melhoria na avaliação do exemplo?

A) A pontuação caiu de 10 para 1
B) A pontuação subiu de 2.32 para 3.92
C) A pontuação ficou exatamente igual
D) A avaliação deixou de funcionar

**Resposta correta: B.**

**Explicação:** Apenas tornar a primeira linha mais clara e direta já melhorou a pontuação.

---

### 10. Qual é a principal lição dessa técnica?

A) Claude deve adivinhar o que o usuário quer
B) Quanto mais vago o prompt, melhor
C) Começar com uma instrução clara e direta melhora os resultados
D) Prompts não precisam ser avaliados

**Resposta correta: C.**

**Explicação:** Claude responde melhor quando recebe direção clara desde o início.
