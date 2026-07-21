# Classificação baseada em modelo

```
notebooks\09_model_based_grading.ipynb
```

## 1. Ideia central

O conteúdo explica como usar **model based grading** em workflows de avaliação de prompts.

A ideia central é que, depois de Claude gerar uma resposta para um caso de teste, precisamos de um **grader** para avaliar a qualidade dessa resposta de forma mensurável.

Um **grader** recebe a saída do modelo e retorna algum sinal de qualidade, normalmente uma nota de **1 a 10**.

Existem três tipos principais de graders:

1. **Code graders**
2. **Model graders**
3. **Human graders**

O foco desta aula é o **model grader**, ou seja, usar outro modelo de IA para avaliar a resposta gerada.

---

## 2. Principais conceitos

### O que é um grader

Um **grader** é um sistema de avaliação.

Ele recebe uma resposta gerada por Claude e retorna uma avaliação estruturada, como:

* nota numérica;
* pontos fortes;
* pontos fracos;
* raciocínio da avaliação;
* indicação de problemas.

Exemplo de escala:

| Nota | Significado             |
| ---- | ----------------------- |
| 10   | Alta qualidade          |
| 5    | Qualidade intermediária |
| 1    | Baixa qualidade         |

O objetivo é transformar qualidade, que pode ser subjetiva, em um sinal mais fácil de comparar.

---

### Tipos de graders

O conteúdo apresenta três abordagens principais.

### 1. Code graders

**Code graders** avaliam a saída usando lógica programática.

Eles são úteis quando é possível verificar algo com código.

Exemplos:

* verificar tamanho da resposta;
* checar se certas palavras aparecem ou não aparecem;
* validar sintaxe de JSON;
* validar sintaxe de Python;
* testar se uma regex compila;
* calcular métricas de legibilidade.

Exemplo conceitual:

```python id="vkg9q1"
def grade_json(output):
    try:
        json.loads(output)
        return 10
    except:
        return 1
```

Code graders são bons para critérios objetivos e verificáveis.

---

### 2. Model graders

**Model graders** usam outro modelo de IA para avaliar a saída.

Nesse caso, você faz uma nova chamada à API com um prompt de avaliação.

O modelo avaliador analisa:

* a tarefa original;
* a resposta gerada;
* os critérios de qualidade.

Depois, ele retorna uma avaliação.

Model graders são úteis para critérios mais subjetivos ou semânticos, como:

* qualidade geral;
* completude;
* utilidade;
* segurança;
* aderência à instrução;
* relevância;
* precisão em relação à tarefa.

---

### 3. Human graders

**Human graders** são avaliadores humanos.

Eles oferecem muita flexibilidade, porque humanos conseguem julgar nuances complexas.

São úteis para avaliar:

* qualidade geral;
* profundidade;
* concisão;
* relevância;
* completude;
* clareza.

A desvantagem é que avaliações humanas são mais lentas, caras e trabalhosas.

---

### Critérios de avaliação

Antes de implementar um grader, é necessário definir **critérios claros**.

Para um prompt de geração de código, a aula sugere três critérios:

| Critério           | O que avalia                                                      | Melhor tipo de grader |
| ------------------ | ----------------------------------------------------------------- | --------------------- |
| **Format**         | Se a resposta contém apenas Python, JSON ou Regex, sem explicação | Code grader           |
| **Valid Syntax**   | Se o código, JSON ou regex tem sintaxe válida                     | Code grader           |
| **Task Following** | Se a resposta resolve corretamente a tarefa do usuário            | Model grader          |

O ponto importante é que nem todo critério deve ser avaliado do mesmo jeito.

Critérios objetivos combinam melhor com código.
Critérios semânticos combinam melhor com model graders ou humanos.

---

### Implementando um model grader

O exemplo cria uma função chamada:

```python id="d8nd1e"
grade_by_model(test_case, output)
```

Essa função recebe:

* `test_case`: o caso de teste original;
* `output`: a resposta gerada por Claude.

Ela monta um prompt de avaliação pedindo para o modelo agir como um **especialista em revisão de código**.

Exemplo apresentado:

````python id="voyxcd"
def grade_by_model(test_case, output):
    # Create evaluation prompt
    eval_prompt = """
    You are an expert code reviewer. Evaluate this AI-generated solution.
    
    Task: {task}
    Solution: {solution}
    
    Provide your evaluation as a structured JSON object with:
    - "strengths": An array of 1-3 key strengths
    - "weaknesses": An array of 1-3 key areas for improvement  
    - "reasoning": A concise explanation of your assessment
    - "score": A number between 1-10
    """
    
    messages = []
    add_user_message(messages, eval_prompt)
    add_assistant_message(messages, "```json")
    
    eval_text = chat(messages, stop_sequences=["```"])
    return json.loads(eval_text)
````

A saída esperada do grader é um JSON com:

* `strengths`;
* `weaknesses`;
* `reasoning`;
* `score`.

---

### Importância de pedir reasoning, strengths e weaknesses

Um ponto importante da aula é que não basta pedir apenas uma nota.

Se você pedir apenas:

```text id="bs965b"
Give this output a score from 1 to 10.
```

o modelo avaliador pode tender a dar notas medianas, como 6, sem muita justificativa.

Por isso, é melhor pedir também:

* pontos fortes;
* pontos fracos;
* raciocínio;
* nota final.

Isso força o avaliador a justificar melhor a pontuação.

---

### Integração com `run_test_case`

Depois de criar o grader, a função `run_test_case()` é atualizada.

Antes, o score era fixo:

```python id="y1bhl5"
score = 10
```

Agora, o score vem do model grader:

```python id="uhz8u8"
def run_test_case(test_case):
    output = run_prompt(test_case)
    
    # Grade the output
    model_grade = grade_by_model(test_case, output)
    score = model_grade["score"]
    reasoning = model_grade["reasoning"]
    
    return {
        "output": output, 
        "test_case": test_case, 
        "score": score,
        "reasoning": reasoning
    }
```

Agora cada resultado inclui:

* saída gerada;
* caso de teste;
* nota;
* justificativa.

Isso torna a avaliação muito mais útil.

---

### Calculando média dos scores

A função `run_eval()` também é atualizada para calcular a média dos resultados:

```python id="obz86a"
from statistics import mean

def run_eval(dataset):
    results = []
    
    for test_case in dataset:
        result = run_test_case(test_case)
        results.append(result)
    
    average_score = mean([result["score"] for result in results])
    print(f"Average score: {average_score}")
    
    return results
```

A média permite comparar diferentes versões do prompt.

Por exemplo:

| Prompt   | Média |
| -------- | ----: |
| Versão 1 |   6.8 |
| Versão 2 |   8.1 |
| Versão 3 |   8.7 |

Assim, você pode acompanhar se as mudanças no prompt realmente melhoraram o desempenho.

---

### Observação técnica importante

No exemplo da aula, o prompt de avaliação contém:

```python id="midcqn"
Task: {task}
Solution: {solution}
```

Mas, para isso funcionar em Python, seria necessário interpolar os valores usando `f-string` ou `.format()`.

Uma versão mais correta seria:

```python id="t8rz68"
eval_prompt = f"""
You are an expert code reviewer. Evaluate this AI-generated solution.

Task: {test_case["task"]}
Solution: {output}

Provide your evaluation as a structured JSON object with:
- "strengths": An array of 1-3 key strengths
- "weaknesses": An array of 1-3 key areas for improvement
- "reasoning": A concise explanation of your assessment
- "score": A number between 1-10
"""
```

Esse detalhe é importante na prática, porque sem interpolação o grader avaliaria literalmente os textos `{task}` e `{solution}`, não os dados reais.

---

## 3. Pontos importantes para certificação

* Um **grader** avalia a qualidade da saída do modelo.
* Graders retornam um sinal mensurável, geralmente uma nota de **1 a 10**.
* Existem três tipos principais: **code graders, model graders e human graders**.
* Code graders são bons para verificações objetivas e programáticas.
* Model graders são bons para critérios mais subjetivos e semânticos.
* Human graders são flexíveis, mas caros e demorados.
* Antes de criar um grader, é necessário definir critérios claros.
* Para geração de código, critérios comuns incluem formato, sintaxe válida e aderência à tarefa.
* **Format** e **Valid Syntax** podem ser avaliados com code graders.
* **Task Following** costuma ser melhor avaliado por model graders.
* Model graders fazem uma nova chamada à API para avaliar a resposta.
* Pedir apenas uma nota pode gerar avaliações medianas e pouco úteis.
* Pedir `strengths`, `weaknesses`, `reasoning` e `score` melhora a qualidade da avaliação.
* A média dos scores permite comparar versões de prompts.
* Model graders são úteis, mas podem ser inconsistentes ou “caprichosos”.

---

## 4. Termos-chave

**Grader**
Sistema que avalia a saída do modelo e retorna uma medida de qualidade.

**Code grader**
Avaliador baseado em código, usado para checagens programáticas.

**Model grader**
Avaliador que usa outro modelo de IA para julgar a resposta.

**Human grader**
Pessoa que avalia manualmente a saída do modelo.

**Score**
Nota numérica atribuída à resposta, normalmente de 1 a 10.

**Evaluation criteria**
Critérios usados para decidir se uma resposta é boa ou ruim.

**Format**
Critério que verifica se a resposta está no formato esperado.

**Valid Syntax**
Critério que verifica se o código, JSON ou regex tem sintaxe válida.

**Task Following**
Critério que avalia se a resposta seguiu corretamente a tarefa solicitada.

**Strengths**
Pontos fortes da resposta avaliada.

**Weaknesses**
Pontos fracos ou áreas de melhoria da resposta.

**Reasoning**
Justificativa da avaliação atribuída pelo grader.

**Average score**
Média das notas obtidas nos casos de teste.

**Capricious**
Termo usado para indicar que model graders podem variar ou ser inconsistentes.

---

## 5. Boas práticas

* Definir critérios de avaliação antes de criar o grader.
* Usar code graders para validações objetivas.
* Usar model graders para critérios semânticos ou difíceis de programar.
* Usar human graders quando for necessário julgamento mais refinado.
* Pedir que o model grader retorne JSON estruturado.
* Incluir `strengths`, `weaknesses`, `reasoning` e `score` na avaliação.
* Usar prefilling com `"```json"` e stop sequence `"```"` para obter JSON limpo.
* Validar o JSON retornado pelo grader com `json.loads()`.
* Calcular a média dos scores para comparar versões de prompt.
* Não confiar cegamente em um único score; revisar também o reasoning.
* Combinar diferentes tipos de graders quando possível.

---

## 6. Limitações, riscos e cuidados

Model graders são úteis, mas não são perfeitos.

Eles podem:

* variar entre execuções;
* dar notas inconsistentes;
* favorecer respostas longas;
* ignorar algum critério;
* ser enganados por respostas bem escritas, mas incorretas;
* atribuir notas medianas se o prompt de avaliação for fraco.

Por isso, é importante pedir justificativas e estruturar bem o prompt do grader.

Também é importante lembrar que model grading gera custo adicional, porque cada avaliação exige uma nova chamada ao modelo.

Outro cuidado é não usar apenas média como métrica final. Uma média alta pode esconder falhas críticas em alguns casos específicos.

Em aplicações sérias, é comum combinar:

* code graders para formato e sintaxe;
* model graders para qualidade e aderência;
* human graders para auditoria e validação final.

---

## 7. Resumo para revisão rápida

* Graders medem a qualidade das respostas.
* A nota geralmente vai de 1 a 10.
* Existem code graders, model graders e human graders.
* Code graders são bons para checagens objetivas.
* Model graders são bons para avaliação semântica.
* Human graders são flexíveis, mas lentos e caros.
* Critérios devem ser definidos antes da avaliação.
* Para código: avalie formato, sintaxe e aderência à tarefa.
* Model grader faz outra chamada à API para avaliar a resposta.
* Peça strengths, weaknesses, reasoning e score.
* Integre o grader em `run_test_case()`.
* Calcule a média dos scores em `run_eval()`.
* Use os scores para comparar versões do prompt.
* Model graders ajudam, mas podem ser inconsistentes.

---

## 8. Perguntas simuladas

### 1. Qual é a função de um grader em um workflow de avaliação?

A) Gerar uma nova API key
B) Avaliar a saída do modelo e retornar um sinal mensurável
C) Substituir o prompt original
D) Aumentar automaticamente o limite de tokens

**Resposta correta: B.**

**Explicação:** Um grader analisa a saída e retorna uma avaliação, geralmente uma nota.

---

### 2. Quais são os três principais tipos de graders?

A) Prompt, token e embedding graders
B) Code, model e human graders
C) JSON, Python e Regex graders
D) Server, client e API graders

**Resposta correta: B.**

**Explicação:** O conteúdo apresenta code graders, model graders e human graders.

---

### 3. Qual tipo de grader é mais adequado para validar se um JSON tem sintaxe correta?

A) Code grader
B) Human grader exclusivamente
C) Model grader exclusivamente
D) Nenhum tipo de grader consegue fazer isso

**Resposta correta: A.**

**Explicação:** Validação de sintaxe JSON pode ser feita programaticamente com código.

---

### 4. Qual tipo de grader é mais adequado para avaliar se a resposta seguiu bem a tarefa do usuário?

A) Code grader simples
B) Model grader
C) Arquivo `.env`
D) Stop sequence

**Resposta correta: B.**

**Explicação:** Aderência à tarefa é um critério mais semântico e flexível, adequado para model graders.

---

### 5. Qual é uma vantagem dos human graders?

A) São sempre instantâneos e gratuitos
B) Oferecem alta flexibilidade de julgamento
C) Eliminam a necessidade de datasets
D) Nunca cometem erros

**Resposta correta: B.**

**Explicação:** Humanos conseguem avaliar nuances complexas, mas o processo é lento e trabalhoso.

---

### 6. Por que é útil pedir strengths, weaknesses e reasoning ao model grader?

A) Porque isso ajuda o modelo a justificar melhor a nota
B) Porque isso remove a necessidade de JSON
C) Porque isso impede qualquer variação
D) Porque isso substitui o dataset

**Resposta correta: A.**

**Explicação:** Pedir justificativa e análise reduz notas superficiais e melhora a utilidade da avaliação.

---

### 7. No exemplo, qual formato o model grader deve retornar?

A) Texto livre sem estrutura
B) JSON estruturado
C) Apenas um número solto
D) Código Python executável

**Resposta correta: B.**

**Explicação:** O prompt pede um objeto JSON com strengths, weaknesses, reasoning e score.

---

### 8. Como `run_test_case()` muda ao integrar o model grader?

A) Ele deixa de chamar Claude
B) Ele chama `grade_by_model()` e usa o score retornado
C) Ele remove o campo `output`
D) Ele apaga o dataset

**Resposta correta: B.**

**Explicação:** A função passa a obter a nota a partir da avaliação feita pelo model grader.

---

### 9. Para que serve calcular o average score em `run_eval()`?

A) Para comparar o desempenho geral entre versões de prompts
B) Para esconder casos com erro
C) Para gerar novas tarefas AWS
D) Para escolher uma API key

**Resposta correta: A.**

**Explicação:** A média dos scores fornece uma métrica agregada para comparar versões.

---

### 10. Qual é uma limitação dos model graders?

A) Eles nunca conseguem avaliar qualidade
B) Eles podem ser inconsistentes ou variar entre avaliações
C) Eles só funcionam com imagens
D) Eles eliminam completamente a necessidade de critérios

**Resposta correta: B.**

**Explicação:** Model graders são flexíveis, mas podem ser caprichosos e não devem ser tratados como perfeitos.
