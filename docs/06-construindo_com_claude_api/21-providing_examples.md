# Fornecendo exemplos

## 1. Ideia central

O conteúdo explica a técnica de **fornecer exemplos no prompt** para orientar Claude sobre como responder.

Essa técnica é chamada de:

* **One-shot prompting**: quando você fornece um único exemplo.
* **Multi-shot prompting**: quando você fornece vários exemplos.

A ideia central é:

**Em vez de apenas dizer o que você quer, mostre exemplos de entrada e saída esperada.**

Isso ajuda Claude a entender padrões, formatos, tom, estilo e principalmente **casos difíceis**, como sarcasmo, ambiguidade ou formatos complexos.

---

## 2. Principais conceitos

### Provide examples

Fornecer exemplos significa incluir no prompt pares de:

```text
input → output ideal
```

Ou seja, você mostra para Claude uma entrada parecida com a que ele receberá e qual seria a resposta correta.

Exemplo conceitual:

```xml
<sample_input>
Great game tonight!
</sample_input>

<ideal_output>
Positive
</ideal_output>
```

Isso ensina o padrão desejado de resposta.

---

### One-shot prompting

**One-shot prompting** ocorre quando você fornece **um único exemplo**.

É útil quando o formato é simples e você quer apenas demonstrar o padrão básico.

Exemplo:

```xml
<sample_input>
Great game tonight!
</sample_input>

<ideal_output>
Positive
</ideal_output>
```

Aqui, Claude aprende que deve classificar sentimentos usando rótulos como `Positive` ou `Negative`.

---

### Multi-shot prompting

**Multi-shot prompting** ocorre quando você fornece **múltiplos exemplos**.

É útil quando há:

* diferentes tipos de entrada;
* casos ambíguos;
* corner cases;
* formatos complexos;
* variações de saída aceitáveis.

No exemplo das imagens, o caso difícil é sarcasmo.

Tweet:

```text
Yeah, sure, that was the best movie I’ve seen since ‘Plan 9 from Outer Space’
```

Na superfície, parece positivo por usar expressões como “best movie”. Mas o sentido real é negativo, porque a frase é sarcástica.

---

### Exemplos para corner cases

Um dos usos mais importantes de exemplos é ensinar Claude a lidar com **casos extremos ou difíceis**.

No exemplo do sentimento de tweets, o prompt inicial apenas diz:

```text
Categorize the sentiment of the below tweet.
If the tweet has a positive sentiment, respond with “Positive”.
If it is negative, respond with “Negative”.
```

Isso pode falhar em tweets sarcásticos.

A versão melhor adiciona exemplos:

```xml
<sample_input>
Great game tonight!
</sample_input>

<ideal_output>
Positive
</ideal_output>
```

E também um exemplo sarcástico:

```xml
<sample_input>
Oh yeah, I really needed a flight delay tonight! Excellent!
</sample_input>

<ideal_output>
Negative
</ideal_output>
```

Esse segundo exemplo mostra a Claude que uma frase aparentemente positiva pode expressar sentimento negativo quando há sarcasmo.

---

### Combinação com XML tags

O conteúdo recomenda fortemente combinar exemplos com **XML tags**.

As tags deixam claro o papel de cada bloco:

```xml
<sample_input>
...
</sample_input>

<ideal_output>
...
</ideal_output>
```

Isso evita confusão entre:

* a entrada real do usuário;
* os exemplos;
* a saída ideal;
* as instruções do prompt.

Essa estrutura é especialmente útil quando há vários exemplos ou formatos mais complexos.

---

### Encontrar bons exemplos em avaliações

A terceira imagem mostra um relatório de avaliação de prompt, com respostas, critérios e notas.

A ideia é: ao rodar avaliações, procure os exemplos que tiveram nota alta, como **10/10**, e use esses pares de entrada e saída como exemplos no prompt.

Isso cria um ciclo de melhoria:

1. Rodar avaliação.
2. Identificar respostas excelentes.
3. Transformar essas respostas em exemplos.
4. Inserir exemplos no prompt.
5. Rodar nova avaliação.
6. Medir se o desempenho melhorou.

Esse processo ajuda Claude a entender o que uma “resposta ideal” significa no seu caso de uso específico.

---

### Explicar por que o exemplo é bom

Além de mostrar input e output, o conteúdo recomenda explicar **por que a saída é ideal**.

Exemplo:

```xml
<ideal_output>
[resposta ideal aqui]
</ideal_output>

This example is well-structured, provides detailed information on food choices and quantities, and aligns with the athlete's goals and restrictions.
```

Isso ajuda Claude a aprender não apenas o formato, mas também os critérios de qualidade por trás da resposta.

---

## 3. Pontos importantes para certificação

* Fornecer exemplos é uma técnica forte de prompt engineering.
* Exemplos mostram a Claude o padrão desejado de entrada e saída.
* **One-shot** significa fornecer um único exemplo.
* **Multi-shot** significa fornecer vários exemplos.
* Exemplos são especialmente úteis para **corner cases**.
* Exemplos ajudam em formatos complexos de saída.
* Exemplos podem mostrar tom, estilo, estrutura e nível de detalhe.
* XML tags ajudam a organizar exemplos.
* Tags como `<sample_input>` e `<ideal_output>` deixam claro o que é entrada e o que é resposta esperada.
* Exemplos podem ser extraídos de outputs com alta pontuação em avaliações.
* Explicar por que um exemplo é ideal ajuda Claude a entender critérios de qualidade.
* Exemplos são uma forma de “mostrar em vez de apenas dizer”.
* Multi-shot é melhor quando há vários cenários ou exceções importantes.

---

## 4. Termos-chave

**Example prompting**
Técnica de incluir exemplos no prompt para orientar o comportamento do modelo.

**One-shot prompting**
Uso de um único exemplo de entrada e saída.

**Multi-shot prompting**
Uso de vários exemplos para cobrir diferentes cenários.

**Input/output pair**
Par composto por uma entrada de exemplo e a saída ideal correspondente.

**Sample input**
Entrada de exemplo mostrada ao modelo.

**Ideal output**
Resposta considerada correta ou desejada para aquela entrada.

**Corner case**
Caso difícil, incomum ou extremo que pode causar erro.

**Sarcasm**
Uso de linguagem aparentemente positiva para expressar sentido negativo ou irônico.

**XML tags**
Marcadores usados para estruturar partes do prompt, como `<sample_input>` e `<ideal_output>`.

**Evaluation report**
Relatório que mostra outputs, notas e justificativas de uma avaliação.

**High-scoring output**
Resposta que recebeu nota alta em uma avaliação e pode ser usada como exemplo.

---

## 5. Boas práticas

Use exemplos quando quiser controlar melhor o comportamento de Claude.

Inclua exemplos para:

* casos ambíguos;
* formatos difíceis;
* tom específico;
* estilo de escrita;
* respostas com estrutura rígida;
* casos em que o modelo costuma errar.

Use XML tags para organizar os exemplos:

```xml
<sample_input>
...
</sample_input>

<ideal_output>
...
</ideal_output>
```

Prefira exemplos relevantes ao seu caso real de uso.

Use exemplos que passaram bem em avaliações. Se uma resposta recebeu nota máxima ou muito alta, ela pode servir como modelo de resposta ideal.

Quando possível, explique por que aquele exemplo é bom. Isso ajuda Claude a entender os critérios de qualidade, não apenas copiar o formato.

---

## 6. Limitações, riscos e cuidados

Exemplos ajudam muito, mas precisam ser bem escolhidos.

Exemplos ruins podem ensinar o comportamento errado.

Exemplos pouco representativos podem fazer Claude funcionar bem em um cenário, mas falhar em outros.

Também existe o risco de usar exemplos demais e deixar o prompt longo, aumentando custo e consumo de contexto.

Outro cuidado é garantir que os exemplos não entrem em conflito com as instruções. Se as instruções dizem uma coisa e o exemplo mostra outra, Claude pode ficar inconsistente.

Em tarefas com muitos corner cases, um único exemplo pode não ser suficiente. Nesse caso, multi-shot prompting é mais adequado.

---

## 7. Resumo para revisão rápida

* Fornecer exemplos melhora prompts.
* Exemplos mostram o padrão desejado.
* One-shot = um exemplo.
* Multi-shot = vários exemplos.
* Exemplos são úteis para corner cases.
* Sarcasmo é um bom exemplo de caso difícil.
* Use `<sample_input>` e `<ideal_output>` para estruturar exemplos.
* Combine exemplos com XML tags.
* Use outputs bem avaliados como exemplos futuros.
* Explique por que o exemplo é ideal.
* Exemplos ajudam Claude a entender formato, tom, estilo e critérios de qualidade.
* A técnica funciona porque mostra, em vez de apenas descrever.

---

## 8. Perguntas simuladas

### 1. O que significa fornecer exemplos em um prompt?

A) Dar pares de entrada e saída esperada para orientar Claude
B) Enviar apenas uma API key junto com o prompt
C) Pedir para Claude ignorar o usuário
D) Usar sempre respostas aleatórias

**Resposta correta: A.**

**Explicação:** Exemplos mostram ao modelo como transformar uma entrada em uma saída ideal.

---

### 2. O que é one-shot prompting?

A) Usar vários exemplos diferentes
B) Usar um único exemplo no prompt
C) Não usar exemplos
D) Usar apenas XML tags sem conteúdo

**Resposta correta: B.**

**Explicação:** One-shot significa fornecer um exemplo para demonstrar o padrão desejado.

---

### 3. O que é multi-shot prompting?

A) Usar múltiplos exemplos no prompt
B) Usar apenas uma pergunta vaga
C) Rodar várias APIs ao mesmo tempo
D) Aumentar a temperature para 1

**Resposta correta: A.**

**Explicação:** Multi-shot prompting usa vários exemplos para cobrir diferentes cenários e casos difíceis.

---

### 4. Por que exemplos são úteis em casos de sarcasmo?

A) Porque sarcasmo pode parecer positivo na superfície, mas ter sentido negativo
B) Porque sarcasmo sempre deve ser ignorado
C) Porque sarcasmo não pode ser classificado
D) Porque sarcasmo só aparece em JSON

**Resposta correta: A.**

**Explicação:** Exemplos ajudam Claude a entender padrões sutis, como frases aparentemente positivas que expressam sentimento negativo.

---

### 5. No exemplo do tweet “Oh yeah, I really needed a flight delay tonight! Excellent!”, qual é a classificação ideal?

A) Positive
B) Negative
C) Neutral
D) JSON

**Resposta correta: B.**

**Explicação:** Apesar das palavras positivas, a frase é sarcástica e expressa sentimento negativo.

---

### 6. Por que usar XML tags em exemplos?

A) Para deixar claro o que é entrada de exemplo e o que é saída ideal
B) Para executar código automaticamente
C) Para esconder informações de Claude
D) Para substituir a avaliação do prompt

**Resposta correta: A.**

**Explicação:** Tags como `<sample_input>` e `<ideal_output>` estruturam o prompt e reduzem ambiguidade.

---

### 7. Quando multi-shot prompting é mais indicado?

A) Quando há vários casos de borda ou tipos diferentes de resposta
B) Quando a tarefa é sempre trivial
C) Quando não existe formato esperado
D) Quando você quer remover contexto do prompt

**Resposta correta: A.**

**Explicação:** Múltiplos exemplos ajudam o modelo a lidar com variações e exceções.

---

### 8. Como avaliações podem ajudar a escolher exemplos?

A) Usando respostas com alta pontuação como exemplos de saída ideal
B) Apagando todos os outputs bons
C) Escolhendo apenas respostas com erro
D) Evitando comparar versões de prompt

**Resposta correta: A.**

**Explicação:** Outputs bem avaliados mostram o que uma resposta ideal deve parecer.

---

### 9. Por que explicar o motivo de um exemplo ser bom pode ajudar?

A) Porque ajuda Claude a entender os critérios de qualidade por trás da resposta
B) Porque impede Claude de ler o exemplo
C) Porque transforma o exemplo em API key
D) Porque remove a necessidade de XML tags

**Resposta correta: A.**

**Explicação:** A explicação mostra não apenas o formato, mas também a lógica de qualidade esperada.

---

### 10. Qual é a principal vantagem de fornecer exemplos?

A) Exemplos mostram diretamente o comportamento desejado, reduzindo ambiguidades
B) Exemplos garantem que o modelo nunca errará
C) Exemplos substituem qualquer necessidade de instrução
D) Exemplos só servem para classificação de sentimento

**Resposta correta: A.**

**Explicação:** Exemplos funcionam como demonstrações práticas do padrão esperado, tornando o prompt mais confiável.
