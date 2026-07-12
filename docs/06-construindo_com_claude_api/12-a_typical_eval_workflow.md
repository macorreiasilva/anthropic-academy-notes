# Um fluxo de trabalho de avaliação típico

```
006_controlling_output.ipynb
```

## 1. Ideia central

O conteúdo explica um **fluxo típico de avaliação de prompts**, chamado de **eval workflow**.

A ideia central é que prompts devem ser melhorados de forma **sistemática e mensurável**, não apenas por tentativa e erro. Para isso, você cria um prompt inicial, testa com um conjunto de exemplos, avalia as respostas, ajusta o prompt e repete o processo.

O objetivo é transformar prompt engineering em um processo mais confiável, baseado em dados.

---

## 2. Principais conceitos

### 1. Criar um prompt inicial

O primeiro passo é escrever uma versão inicial do prompt.

Exemplo:

```python
prompt = f"""
Please answer the user's question:

{question}
"""
```

Esse prompt funciona como **baseline**, ou seja, uma versão de referência para comparação.

A partir dele, você poderá medir se uma nova versão do prompt ficou melhor ou pior.

---

### 2. Criar um eval dataset

O **eval dataset** é um conjunto de exemplos usados para testar o prompt.

Ele deve conter entradas parecidas com as que o sistema receberá em produção.

Exemplo do conteúdo:

```text
What's 2+2?
How do I make oatmeal?
How far away is the Moon?
```

Essas perguntas serão inseridas no template do prompt no lugar de `{question}`.

Em aplicações reais, esse dataset pode conter dezenas, centenas ou milhares de exemplos.

---

### 3. Enviar os exemplos para Claude

Depois de criar o dataset, cada pergunta é combinada com o prompt.

Por exemplo, a pergunta:

```text
What's 2+2?
```

vira:

```text
Please answer the user's question:

What's 2+2?
```

Claude então responde a cada entrada do dataset.

O resultado dessa etapa é uma coleção de respostas geradas pelo modelo.

---

### 4. Avaliar com um grader

O **grader** é o avaliador responsável por analisar a qualidade das respostas.

Ele recebe:

* a pergunta original;
* a resposta gerada por Claude;
* os critérios de avaliação.

Depois, atribui uma nota, geralmente em uma escala de **1 a 10**.

Exemplo:

| Pergunta                    | Nota |
| --------------------------- | ---: |
| “What's 2+2?”               |   10 |
| “How do I make oatmeal?”    |    4 |
| “How far away is the Moon?” |    9 |

A média dessas notas fornece uma medida objetiva do desempenho do prompt:

```text
(10 + 4 + 9) ÷ 3 = 7.66
```

---

### 5. Alterar o prompt e repetir

Depois de obter uma pontuação inicial, você modifica o prompt e roda a avaliação novamente.

Exemplo de melhoria:

```python
prompt = f"""
Please answer the user's question:

{question}

Answer the question with ample detail
"""
```

Se a nova versão obtiver uma média maior, por exemplo **8.7**, isso indica que a alteração melhorou o desempenho no conjunto de avaliação.

Esse processo é chamado de **iteração**.

---

### Prompt scoring

**Prompt scoring** é a prática de atribuir pontuações ao desempenho de um prompt.

Isso permite:

* comparar versões diferentes numericamente;
* escolher a versão com melhor resultado;
* continuar melhorando o prompt com base em evidências;
* reduzir decisões subjetivas.

A grande vantagem é remover o “achismo” do prompt engineering.

---

## 3. Pontos importantes para certificação

* Um workflow típico de avaliação de prompts tem **cinco etapas**.
* As etapas são: criar prompt, criar dataset, enviar para Claude, avaliar com grader, alterar e repetir.
* O primeiro prompt serve como **baseline**.
* O eval dataset deve representar entradas reais ou prováveis de produção.
* Cada entrada do dataset é interpolada no template do prompt.
* Claude gera uma resposta para cada caso de teste.
* O grader avalia a resposta com base em critérios definidos.
* A nota pode ser dada em escala de **1 a 10**.
* A média das notas mede o desempenho geral do prompt.
* Alterações no prompt devem ser comparadas usando o mesmo dataset.
* Prompt scoring permite comparar versões de forma objetiva.
* A avaliação sistemática reduz riscos em produção.

---

## 4. Termos-chave

**Eval workflow**
Fluxo estruturado para testar, medir e melhorar prompts.

**Prompt evaluation**
Processo de avaliação da qualidade de um prompt com base em testes e métricas.

**Prompt engineering**
Prática de criar e ajustar prompts para obter melhores respostas.

**Baseline**
Versão inicial usada como referência de comparação.

**Eval dataset**
Conjunto de exemplos usados para testar o prompt.

**Prompt template**
Modelo de prompt com variáveis, como `{question}`.

**Interpolation**
Inserção de valores reais, como perguntas, dentro de um template.

**Grader**
Avaliador que pontua a qualidade da resposta gerada.

**Prompt scoring**
Atribuição de notas ao desempenho de um prompt.

**Average score**
Média das pontuações obtidas nos casos de teste.

**Iteration**
Ciclo de melhoria: testar, medir, ajustar e repetir.

---

## 5. Boas práticas

* Começar com um prompt simples como baseline.
* Criar um dataset representativo do uso real.
* Usar o mesmo dataset para comparar versões diferentes.
* Incluir exemplos variados, não apenas casos fáceis.
* Definir critérios claros para o grader.
* Avaliar com pontuações numéricas quando possível.
* Fazer uma mudança por vez no prompt para entender o impacto.
* Repetir o processo até obter uma melhoria consistente.
* Usar datasets maiores conforme a aplicação cresce.
* Comparar prompts com base em métricas, não apenas preferência pessoal.

---

## 6. Limitações, riscos e cuidados

Um dataset pequeno pode não representar bem o comportamento real dos usuários.

No exemplo, há apenas três perguntas. Isso é útil para entender o conceito, mas seria pouco para uma aplicação séria em produção.

Outro cuidado é que a média pode esconder falhas importantes. Um prompt pode ter boa média geral, mas falhar em um tipo crítico de pergunta.

Também é importante que o grader tenha critérios bem definidos. Se o avaliador for inconsistente ou mal instruído, a pontuação pode ser enganosa.

Além disso, uma melhoria no dataset de avaliação não garante perfeição no mundo real. O dataset deve ser atualizado com novos casos encontrados ao longo do uso.

---

## 7. Resumo para revisão rápida

* Avaliar prompts é tão importante quanto escrevê-los.
* O fluxo típico tem cinco etapas.
* Primeiro, escreva um prompt inicial.
* Depois, crie um eval dataset.
* Envie cada exemplo para Claude.
* Avalie as respostas com um grader.
* Calcule uma média de pontuação.
* Altere o prompt e rode a avaliação novamente.
* Compare versões numericamente.
* Use a versão com melhor desempenho.
* A avaliação reduz achismo e aumenta confiabilidade.
* O processo é iterativo e baseado em dados.

---

## 8. Perguntas simuladas

### 1. Qual é o objetivo principal de um eval workflow?

A) Criar automaticamente uma API key

B) Melhorar prompts de forma sistemática usando medições objetivas

C) Substituir Claude por código tradicional

D) Fazer streaming de respostas

**Resposta correta: B.**

**Explicação:** O eval workflow permite testar, pontuar e comparar prompts com base em dados.

---

### 2. O que é o prompt inicial em um processo de avaliação?

A) Uma versão usada como baseline

B) Uma resposta final do usuário

C) Um arquivo `.env`

D) Um modelo treinado do zero

**Resposta correta: A.**

**Explicação:** O prompt inicial serve como referência para comparar versões futuras.

---

### 3. Para que serve um eval dataset?

A) Para armazenar API keys

B) Para reunir exemplos usados no teste do prompt

C) Para escolher automaticamente a temperature

D) Para impedir Claude de responder

**Resposta correta: B.**

**Explicação:** O eval dataset contém entradas representativas que serão usadas para avaliar o prompt.

---

### 4. No exemplo, qual variável é inserida dentro do prompt template?

A) `{model}`

B) `{question}`

C) `{temperature}`

D) `{score}`

**Resposta correta: B.**

**Explicação:** As perguntas do dataset são interpoladas no lugar de `{question}`.

---

### 5. Qual é a função do grader?

A) Criar o prompt inicial

B) Avaliar a resposta de Claude e atribuir uma pontuação

C) Gerar uma API key

D) Fazer deploy da aplicação

**Resposta correta: B.**

**Explicação:** O grader examina a pergunta e a resposta para medir a qualidade da saída.

---

### 6. Se as notas forem 10, 4 e 9, qual é a média?

A) 6.33

B) 7.66

C) 8.7

D) 10

**Resposta correta: B.**

**Explicação:** A média é `(10 + 4 + 9) ÷ 3 = 7.66`.

---

### 7. Por que repetir o processo após mudar o prompt?

A) Para verificar se a alteração melhorou a performance

B) Para trocar a API key

C) Para apagar o dataset

D) Para impedir novas respostas

**Resposta correta: A.**

**Explicação:** Reavaliar permite comparar a nova versão com a baseline.

---

### 8. O que é prompt scoring?

A) Escolher o modelo mais barato

B) Atribuir notas ao desempenho de um prompt

C) Transformar prompts em JSON

D) Fazer Claude responder em tempo real

**Resposta correta: B.**

**Explicação:** Prompt scoring mede a qualidade das respostas para permitir comparação entre versões.

---

### 9. Qual é uma vantagem de comparar prompts numericamente?

A) Reduz o achismo na escolha da melhor versão

B) Elimina a necessidade de testes

C) Garante que o prompt nunca falhará

D) Remove o uso de datasets

**Resposta correta: A.**

**Explicação:** Métricas objetivas ajudam a decidir com mais confiança qual versão é melhor.

---

### 10. Qual é um cuidado importante ao usar eval datasets?

A) Usar apenas um exemplo simples

B) Garantir que o dataset represente bem os casos reais de uso

C) Evitar qualquer tipo de pontuação

D) Usar sempre perguntas idênticas

**Resposta correta: B.**

**Explicação:** Um dataset pouco representativo pode gerar avaliações enganosas e deixar falhas passarem para produção.
