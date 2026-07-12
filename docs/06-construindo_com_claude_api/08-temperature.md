# Temperatura

```
004_temperatures.ipynb
```

## 1. Ideia central

O conteúdo explica o parâmetro **temperature**, usado para controlar o quanto as respostas de Claude serão **previsíveis** ou **criativas**.

A ideia principal é:

**Temperature ajusta o grau de aleatoriedade na escolha dos próximos tokens gerados pelo modelo.**

Temperaturas baixas deixam a resposta mais consistente e determinística. Temperaturas altas tornam a resposta mais variada, criativa e menos previsível.

---

## 2. Principais conceitos

### Como Claude gera texto

Antes de entender temperature, é importante lembrar que Claude gera texto em etapas:

1. **Tokenization**: quebra o texto de entrada em partes menores chamadas tokens.
2. **Prediction**: calcula probabilidades para os próximos tokens possíveis.
3. **Sampling**: escolhe um token com base nessas probabilidades.

Por exemplo, diante de um prompt como:

```text
What do you think?
```

Claude pode calcular probabilidades para o próximo token:

| Token possível | Probabilidade |
| -------------- | ------------: |
| `about`        |           30% |
| `would`        |           20% |
| `of`           |           10% |

Depois, ele escolhe um token e repete o processo até formar a resposta completa.

---

### O que é temperature

**Temperature** é um valor decimal entre **0 e 1** que influencia como Claude escolhe o próximo token.

Ela funciona como um “controle de criatividade”:

* **temperature baixa**: respostas mais previsíveis;
* **temperature alta**: respostas mais variadas e criativas.

Em temperatura próxima de **0**, Claude tende a escolher quase sempre o token mais provável.

Em temperatura próxima de **1**, Claude distribui melhor as chances entre diferentes tokens possíveis, aumentando a variedade da resposta.

---

### Temperature baixa

Com **temperature baixa**, Claude se torna mais determinístico.

Isso significa que ele tende a escolher a opção mais provável com maior frequência.

Exemplo:

```python
answer = chat(messages, temperature=0.0)
```

Esse tipo de configuração é útil quando você quer respostas estáveis, consistentes e menos criativas.

Casos de uso:

* respostas factuais;
* assistência com código;
* extração de dados;
* moderação de conteúdo.

---

### Temperature média

Com **temperature média**, Claude mantém certo equilíbrio entre previsibilidade e variação.

Faixa sugerida:

```text
0.4 - 0.7
```

Essa configuração costuma funcionar bem para tarefas gerais, especialmente quando você quer alguma flexibilidade, mas sem perder muito controle.

Casos de uso:

* resumo;
* conteúdo educacional;
* resolução de problemas;
* escrita criativa com restrições.

---

### Temperature alta

Com **temperature alta**, Claude fica mais criativo e menos previsível.

Exemplo:

```python
answer = chat(messages, temperature=1.0)
```

Essa configuração é útil quando o objetivo é gerar variedade, ideias novas ou estilos mais livres.

Casos de uso:

* brainstorming;
* escrita criativa;
* conteúdo de marketing;
* geração de piadas.

---

### Implementando temperature no código

A função `chat()` pode ser modificada para aceitar o parâmetro `temperature`:

```python
def chat(messages, system=None, temperature=1.0):
    params = {
        "model": model,
        "max_tokens": 1000,
        "messages": messages,
        "temperature": temperature
    }
    
    if system:
        params["system"] = system
    
    message = client.messages.create(**params)
    return message.content[0].text
```

A mudança principal é incluir:

```python
temperature=1.0
```

como parâmetro da função e adicionar:

```python
"temperature": temperature
```

ao dicionário `params`.

---

### Testando os efeitos da temperature

Para observar a diferença, o curso sugere gerar ideias de filmes com diferentes temperaturas.

Com temperature baixa:

```python
answer = chat(messages, temperature=0.0)
```

Claude tende a gerar respostas mais parecidas e previsíveis.

Com temperature alta:

```python
answer = chat(messages, temperature=1.0)
```

Claude tende a gerar mais variedade em temas, personagens e enredos.

Um ponto importante: **temperature alta não garante respostas diferentes sempre**. Ela apenas aumenta a probabilidade de variação.

---

## 3. Pontos importantes para certificação

* **Temperature controla a aleatoriedade da geração.**
* O valor fica entre **0 e 1**.
* Temperature baixa gera respostas mais previsíveis e determinísticas.
* Temperature alta gera respostas mais criativas e variadas.
* Temperature não muda o conhecimento do modelo; muda a forma como ele escolhe tokens.
* Claude gera texto por **tokenization, prediction e sampling**.
* Temperature atua principalmente na etapa de **sampling**.
* Para respostas factuais, código e extração de dados, prefira temperature baixa.
* Para brainstorming, escrita criativa e marketing, prefira temperature alta.
* Para tarefas gerais, uma temperature média pode ser adequada.
* Temperature alta **não garante** que a saída será sempre diferente.
* A escolha da temperature deve estar alinhada ao caso de uso.

---

## 4. Termos-chave

**Temperature**
Parâmetro que controla o grau de aleatoriedade ou criatividade na geração de texto.

**Deterministic / Determinístico**
Comportamento mais previsível, em que o modelo tende a escolher sempre as opções mais prováveis.

**Tokenization**
Processo de dividir o texto em tokens.

**Token**
Unidade de texto processada pelo modelo, como uma palavra, parte de palavra ou símbolo.

**Prediction**
Etapa em que o modelo calcula probabilidades para os próximos tokens possíveis.

**Sampling**
Etapa em que o modelo escolhe o próximo token com base nas probabilidades.

**Probability distribution**
Distribuição de probabilidades entre os tokens possíveis.

**Low temperature**
Configuração que favorece respostas consistentes e previsíveis.

**High temperature**
Configuração que aumenta variedade, criatividade e aleatoriedade.

**Creativity dial**
Metáfora usada para entender temperature como um controle de criatividade.

---

## 5. Boas práticas

Use **temperature baixa**, entre **0.0 e 0.3**, quando precisar de precisão, consistência e previsibilidade.

Exemplos:

* código;
* extração de informações;
* respostas factuais;
* classificação;
* moderação.

Use **temperature média**, entre **0.4 e 0.7**, quando quiser equilíbrio entre estabilidade e flexibilidade.

Exemplos:

* resumos;
* explicações educacionais;
* resolução de problemas;
* escrita com alguma criatividade, mas ainda controlada.

Use **temperature alta**, entre **0.8 e 1.0**, quando quiser variedade e criatividade.

Exemplos:

* ideias de campanha;
* brainstorming;
* histórias;
* slogans;
* piadas;
* conceitos criativos.

Sempre teste diferentes valores para o mesmo prompt, porque o melhor valor depende do caso de uso.

---

## 6. Limitações, riscos e cuidados

Temperature não garante controle absoluto sobre a resposta.

Mesmo com temperature alta, Claude pode gerar respostas parecidas. E mesmo com temperature baixa, pequenas variações ainda podem acontecer dependendo do contexto e da configuração da chamada.

Outro cuidado importante: usar temperature alta em tarefas que exigem precisão pode aumentar o risco de respostas inconsistentes, imprecisas ou menos confiáveis.

Por exemplo, em tarefas como extração de dados, moderação, análise factual ou geração de código, uma temperature alta pode ser inadequada.

Também é importante não confundir criatividade com qualidade. Uma resposta mais variada não é necessariamente melhor. O valor ideal de temperature depende do objetivo da aplicação.

---

## 7. Resumo para revisão rápida

* Temperature controla previsibilidade versus criatividade.
* O valor vai de **0 a 1**.
* Baixa temperature = respostas mais consistentes.
* Alta temperature = respostas mais variadas.
* Claude gera texto por tokenization, prediction e sampling.
* Temperature afeta a escolha dos tokens durante o sampling.
* Use temperature baixa para fatos, código, extração e moderação.
* Use temperature média para resumo, ensino e resolução de problemas.
* Use temperature alta para brainstorming, marketing e escrita criativa.
* Temperature alta não garante respostas sempre diferentes.
* Escolha a temperature de acordo com o caso de uso.

---

## 8. Perguntas simuladas

### 1. O que o parâmetro temperature controla?

A) O número máximo de tokens da resposta
B) O modelo Claude que será usado
C) O grau de previsibilidade ou criatividade da resposta
D) A API key usada na requisição

**Resposta correta: C.**

**Explicação:** Temperature ajusta a aleatoriedade na escolha dos próximos tokens, influenciando se a resposta será mais previsível ou criativa.

---

### 2. Qual faixa de temperature é mais indicada para respostas factuais?

A) 0.0 - 0.3
B) 0.4 - 0.7
C) 0.8 - 1.0
D) Apenas 1.0

**Resposta correta: A.**

**Explicação:** Temperaturas baixas são melhores para tarefas que exigem consistência e precisão.

---

### 3. Qual faixa de temperature é mais adequada para brainstorming?

A) 0.0 - 0.1
B) 0.2 - 0.3
C) 0.8 - 1.0
D) Sempre 0.0

**Resposta correta: C.**

**Explicação:** Brainstorming se beneficia de maior variedade e criatividade, características de temperaturas altas.

---

### 4. Em qual etapa a temperature tem efeito mais direto?

A) Na criação da API key
B) Na escolha do próximo token durante o sampling
C) Na instalação do pacote `anthropic`
D) No armazenamento do arquivo `.env`

**Resposta correta: B.**

**Explicação:** Temperature influencia a forma como o modelo escolhe entre tokens possíveis com base nas probabilidades.

---

### 5. O que acontece em temperature próxima de 0?

A) Claude escolhe tokens de forma totalmente aleatória
B) Claude tende a escolher o token mais provável
C) Claude para de gerar texto
D) Claude ignora o prompt do usuário

**Resposta correta: B.**

**Explicação:** Com temperature baixa, a geração fica mais determinística e previsível.

---

### 6. O que acontece em temperature próxima de 1?

A) As respostas tendem a ficar mais variadas e criativas
B) Claude sempre gera a mesma resposta
C) A API key deixa de ser necessária
D) O limite de contexto aumenta automaticamente

**Resposta correta: A.**

**Explicação:** Temperaturas altas distribuem mais as probabilidades entre tokens possíveis, aumentando a variação.

---

### 7. Qual configuração seria mais indicada para extração de dados?

A) Temperature alta
B) Temperature baixa
C) Sem parâmetro `messages`
D) Apenas prompts vazios

**Resposta correta: B.**

**Explicação:** Extração de dados exige consistência e precisão, por isso combina melhor com temperature baixa.

---

### 8. Qual configuração seria mais indicada para gerar ideias de marketing?

A) Temperature alta
B) Temperature sempre igual a 0
C) Nenhum uso de sampling
D) Remover `max_tokens`

**Resposta correta: A.**

**Explicação:** Ideias de marketing geralmente se beneficiam de criatividade e variedade.

---

### 9. O que significa dizer que temperature alta não garante respostas diferentes?

A) Que temperature não tem nenhum efeito
B) Que ela apenas aumenta a probabilidade de variação, mas não obriga o modelo a variar
C) Que Claude para de responder em temperature alta
D) Que ela só funciona em prompts de código

**Resposta correta: B.**

**Explicação:** Temperature altera probabilidades, mas não garante diversidade absoluta em todas as chamadas.

---

### 10. Como adicionar temperature à função `chat()`?

A) Adicionando `"temperature": temperature` aos parâmetros da chamada
B) Colocando a temperature dentro da API key
C) Criando uma mensagem com `role: "temperature"`
D) Substituindo `messages` por `temperature`

**Resposta correta: A.**

**Explicação:** O parâmetro deve ser incluído no dicionário enviado para `client.messages.create()`.
