# Executando a avaliação

```
notebooks\08_running_the_eval.ipynb
```

## 1. Ideia central

O conteúdo explica como **executar uma avaliação de prompts** usando um dataset de testes já criado.

A ideia central é montar um **pipeline básico de avaliação**, que faz três coisas:

1. pega cada caso de teste do dataset;
2. combina esse caso com um prompt;
3. envia para Claude e registra o resultado com uma pontuação.

Neste momento, a avaliação ainda é simples, porque a nota está fixa em `10`. O objetivo da aula é construir a estrutura principal do pipeline antes de adicionar um sistema real de avaliação, chamado **grader**.

---

## 2. Principais conceitos

### O que significa “rodar o eval”

Rodar o eval significa executar automaticamente um conjunto de testes contra um prompt.

O fluxo é:

```text
dataset → prompt template → Claude → resposta → grader → resultado
```

Cada item do dataset é tratado como um caso de teste.

Por exemplo, se o dataset contém uma tarefa como:

```json
{
  "task": "Create a Python function that validates an AWS ARN."
}
```

essa tarefa é inserida em um prompt e enviada para Claude.

---

### Função `run_prompt`

A função `run_prompt()` é responsável por executar o prompt para um único caso de teste.

```python
def run_prompt(test_case):
    """Merges the prompt and test case input, then returns the result"""
    prompt = f"""
Please solve the following task:

{test_case["task"]}
"""
    
    messages = []
    add_user_message(messages, prompt)
    output = chat(messages)
    return output
```

Ela faz três coisas:

1. recebe um `test_case`;
2. monta o prompt usando `test_case["task"]`;
3. envia o prompt para Claude usando `chat(messages)`.

Essa função retorna apenas a saída gerada por Claude.

Neste momento, o prompt ainda é simples e não inclui instruções de formatação. Por isso, Claude provavelmente responderá com textos mais longos e explicações extras.

---

### Função `run_test_case`

A função `run_test_case()` executa um caso de teste completo e registra o resultado.

```python
def run_test_case(test_case):
    """Calls run_prompt, then grades the result"""
    output = run_prompt(test_case)
    
    # TODO - Grading
    score = 10
    
    return {
        "output": output,
        "test_case": test_case,
        "score": score
    }
```

Ela faz:

1. chama `run_prompt(test_case)`;
2. recebe a resposta de Claude;
3. atribui uma nota;
4. retorna um objeto com o resultado.

Por enquanto, a nota está fixa:

```python
score = 10
```

Isso é apenas um placeholder. A lógica real de avaliação será adicionada depois com um **grader**.

O retorno tem três campos principais:

| Campo       | Função                     |
| ----------- | -------------------------- |
| `output`    | Resposta gerada por Claude |
| `test_case` | Caso de teste original     |
| `score`     | Nota da avaliação          |

---

### Função `run_eval`

A função `run_eval()` coordena a avaliação completa.

```python
def run_eval(dataset):
    """Loads the dataset and calls run_test_case with each case"""
    results = []
    
    for test_case in dataset:
        result = run_test_case(test_case)
        results.append(result)
    
    return results
```

Ela percorre todos os casos do dataset e executa `run_test_case()` para cada um.

O resultado final é uma lista com todos os resultados da avaliação.

Em termos simples:

```text
para cada caso no dataset:
    rode o prompt
    avalie o resultado
    salve o resultado
```

---

### Carregando o dataset

Antes de rodar a avaliação, o dataset precisa ser carregado do arquivo `dataset.json`.

```python
with open("dataset.json", "r") as f:
    dataset = json.load(f)
```

Esse código lê o arquivo JSON e transforma seu conteúdo em uma estrutura Python.

Depois, o eval é executado:

```python
results = run_eval(dataset)
```

---

### Examinando os resultados

Depois de rodar o eval, os resultados podem ser impressos de forma legível:

```python
print(json.dumps(results, indent=2))
```

Isso mostra uma lista de objetos JSON, onde cada objeto representa um caso de teste avaliado.

Cada resultado inclui:

* o output completo de Claude;
* o test case original;
* a pontuação atribuída.

Como a nota ainda está fixa em `10`, o score ainda não mede qualidade real. Ele apenas permite testar se o pipeline funciona de ponta a ponta.

---

### Problema identificado

Como o prompt usado ainda é muito simples:

```python
prompt = f"""
Please solve the following task:

{test_case["task"]}
"""
```

Claude pode gerar respostas verbosas, com explicações, introduções ou formatação indesejada.

Isso é importante porque o objetivo do projeto era gerar saídas limpas em formatos como:

* Python;
* JSON;
* Regex.

Esse comportamento será corrigido depois com melhorias no prompt e com um grader mais rigoroso.

---

## 3. Pontos importantes para certificação

* Rodar um eval significa processar um dataset de testes usando um prompt.
* O pipeline básico combina cada caso de teste com um prompt template.
* Cada prompt completo é enviado para Claude.
* A resposta de Claude é armazenada como `output`.
* O caso de teste original é preservado como `test_case`.
* A avaliação recebe uma pontuação chamada `score`.
* Neste estágio, o score está hardcoded como `10`.
* O hardcoded score é apenas um placeholder, não uma avaliação real.
* A função `run_prompt()` executa o prompt para um caso individual.
* A função `run_test_case()` executa e avalia um caso individual.
* A função `run_eval()` executa todos os casos do dataset.
* `json.load()` carrega o dataset a partir de um arquivo.
* `json.dumps(..., indent=2)` ajuda a visualizar os resultados.
* A parte mais importante que ainda falta é o **grader real**.
* Mesmo um pipeline simples já representa a base de muitos sistemas de avaliação de IA.

---

## 4. Termos-chave

**Eval**
Processo de avaliação usado para medir o desempenho de um prompt ou modelo.

**Evaluation pipeline**
Fluxo automatizado que executa testes, coleta respostas e calcula avaliações.

**Dataset**
Conjunto de casos de teste usados na avaliação.

**Test case**
Um exemplo individual do dataset, contendo uma tarefa ou entrada.

**Prompt template**
Modelo de prompt que recebe dados variáveis, como `test_case["task"]`.

**`run_prompt()`**
Função que monta o prompt e envia para Claude.

**`run_test_case()`**
Função que executa um caso de teste e retorna output, test case e score.

**`run_eval()`**
Função que executa a avaliação em todo o dataset.

**Output**
Resposta gerada por Claude para um caso de teste.

**Score**
Pontuação atribuída ao resultado da avaliação.

**Hardcoded score**
Nota fixa definida diretamente no código, usada temporariamente.

**Grader**
Sistema ou lógica responsável por avaliar a qualidade da resposta.

**`json.load()`**
Função Python que carrega dados JSON de um arquivo.

**`json.dumps()`**
Função Python que converte dados em string JSON formatada.

---

## 5. Boas práticas

* Separar o pipeline em funções pequenas e claras.
* Usar `run_prompt()` apenas para montar e executar o prompt.
* Usar `run_test_case()` para organizar a execução e avaliação de um caso.
* Usar `run_eval()` para processar o dataset inteiro.
* Preservar o `test_case` original junto com o `output`.
* Salvar resultados em formato estruturado para análise posterior.
* Usar `json.dumps(..., indent=2)` para inspecionar os resultados.
* Começar com um pipeline simples antes de adicionar complexidade.
* Substituir scores fixos por um grader real em etapas futuras.
* Observar os outputs para identificar problemas no prompt, como excesso de explicação ou formato incorreto.

---

## 6. Limitações, riscos e cuidados

Neste estágio, o principal limite é que a avaliação ainda **não mede qualidade real**, porque o score está fixo em `10`.

Isso significa que, mesmo que Claude gere uma resposta ruim, o sistema ainda atribuirá nota máxima.

Outro cuidado é que o prompt inicial é muito genérico. Como ele apenas diz:

```text
Please solve the following task
```

Claude pode responder com explicações, markdown, cabeçalhos e rodapés, o que pode ser inadequado se a aplicação exige saída limpa.

Também existe o cuidado com tempo e custo. Rodar o eval pode levar vários segundos ou até minutos dependendo de:

* tamanho do dataset;
* modelo usado;
* tamanho das respostas;
* número de chamadas à API;
* latência da rede.

Por fim, é importante lembrar que esse pipeline é sequencial. Ele processa um caso depois do outro. Em datasets maiores, talvez seja necessário pensar em otimizações, como paralelização, cache ou redução de tokens.

---

## 7. Resumo para revisão rápida

* O eval executa um prompt contra um dataset.
* Cada item do dataset é um test case.
* `run_prompt()` monta o prompt e chama Claude.
* `run_test_case()` executa o prompt e retorna output, test case e score.
* `run_eval()` percorre todo o dataset.
* O dataset é carregado de `dataset.json`.
* Os resultados são uma lista estruturada.
* Cada resultado contém `output`, `test_case` e `score`.
* Neste estágio, o score é fixo em `10`.
* O próximo passo é substituir o score fixo por um grader real.
* O prompt ainda é simples e pode gerar respostas verbosas.
* O pipeline básico já é a fundação de um sistema de avaliação de IA.

---

## 8. Perguntas simuladas

### 1. Qual é o objetivo de rodar um eval?

A) Criar automaticamente uma API key
B) Executar um prompt contra casos de teste e coletar resultados
C) Substituir o dataset por um system prompt
D) Impedir Claude de gerar respostas longas

**Resposta correta: B.**

**Explicação:** Rodar um eval significa testar o prompt com entradas do dataset e registrar as respostas para avaliação.

---

### 2. Qual é a função de `run_prompt()`?

A) Carregar o arquivo `.env`
B) Montar o prompt com o caso de teste e chamar Claude
C) Calcular a média final das notas
D) Apagar o dataset após o uso

**Resposta correta: B.**

**Explicação:** `run_prompt()` recebe um test case, insere a tarefa no prompt e retorna a resposta de Claude.

---

### 3. Qual campo do `test_case` é usado no prompt do exemplo?

A) `test_case["model"]`
B) `test_case["score"]`
C) `test_case["task"]`
D) `test_case["temperature"]`

**Resposta correta: C.**

**Explicação:** A tarefa do caso de teste está armazenada na chave `"task"`.

---

### 4. O que a função `run_test_case()` retorna?

A) Apenas a API key
B) Um objeto com `output`, `test_case` e `score`
C) Apenas o nome do modelo
D) Um arquivo `.env` atualizado

**Resposta correta: B.**

**Explicação:** Ela retorna os dados estruturados de um caso avaliado.

---

### 5. Neste estágio da aula, como o score é calculado?

A) Por um grader avançado
B) Por comparação com uma resposta esperada
C) Ele está fixo manualmente como `10`
D) Pela contagem de tokens

**Resposta correta: C.**

**Explicação:** O score ainda é hardcoded como `10`, servindo apenas como placeholder.

---

### 6. Qual é a função de `run_eval()`?

A) Executar todos os casos do dataset usando `run_test_case()`
B) Criar novas tarefas automaticamente
C) Instalar o SDK da Anthropic
D) Converter Python em JSON

**Resposta correta: A.**

**Explicação:** `run_eval()` percorre o dataset inteiro e coleta os resultados de cada caso de teste.

---

### 7. Como o dataset é carregado no exemplo?

A) Com `json.load(f)` a partir de `dataset.json`
B) Com `json.dumps(results)`
C) Com `client.messages.create()`
D) Com `add_user_message()`

**Resposta correta: A.**

**Explicação:** `json.load(f)` lê o arquivo JSON e transforma seu conteúdo em objeto Python.

---

### 8. Para que serve `json.dumps(results, indent=2)`?

A) Para enviar a resposta para Claude
B) Para exibir os resultados em JSON formatado e legível
C) Para apagar outputs ruins
D) Para gerar uma stop sequence

**Resposta correta: B.**

**Explicação:** `json.dumps(..., indent=2)` formata os resultados para inspeção humana.

---

### 9. Qual é a principal peça que ainda falta no pipeline?

A) Um grader real
B) Uma API key dentro do prompt
C) Um arquivo de imagem
D) Um chatbot front-end

**Resposta correta: A.**

**Explicação:** O score fixo precisa ser substituído por lógica real de avaliação, chamada grader.

---

### 10. Por que Claude pode gerar respostas verbosas nesse estágio?

A) Porque o prompt ainda não tem instruções específicas de formatação
B) Porque o dataset está vazio
C) Porque `json.load()` adiciona explicações automaticamente
D) Porque o score está fixo em 10

**Resposta correta: A.**

**Explicação:** O prompt apenas pede para resolver a tarefa, sem exigir saída limpa, então Claude pode adicionar explicações.
