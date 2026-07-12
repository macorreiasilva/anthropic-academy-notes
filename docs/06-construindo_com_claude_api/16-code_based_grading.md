# Avaliação baseada em código

```
10_code_based_grading.ipynb
```

## 1. Ideia central

O conteúdo explica **code based grading**, uma forma de avaliar respostas geradas por IA usando código.

A ideia principal é que, quando Claude gera código, JSON ou regex, não basta avaliar se a resposta “parece boa”. Também é necessário verificar se a saída:

1. está no **formato correto**;
2. tem **sintaxe válida**;
3. resolve a **tarefa solicitada**.

Nesse workflow, os dois primeiros critérios podem ser avaliados por código, enquanto o terceiro costuma ser melhor avaliado por um **model grader**.

---

## 2. Principais conceitos

### Code based grading

**Code based grading** é uma avaliação automatizada feita por código.

Ela verifica propriedades objetivas da saída, como:

* se o JSON consegue ser parseado;
* se o código Python tem sintaxe válida;
* se a regex compila corretamente;
* se a resposta contém apenas o formato pedido, sem explicações extras.

Esse tipo de grader é muito útil porque fornece uma avaliação mais determinística do que um model grader.

---

### Três critérios de avaliação

O conteúdo apresenta três critérios principais para avaliar saídas de código:

| Critério           | O que avalia                                                       | Tipo de grader mais adequado |
| ------------------ | ------------------------------------------------------------------ | ---------------------------- |
| **Format**         | Se a resposta contém apenas Python, JSON ou Regex, sem explicações | Code grader                  |
| **Valid Syntax**   | Se a saída tem sintaxe válida                                      | Code grader                  |
| **Task Following** | Se a resposta resolve corretamente a tarefa pedida                 | Model grader                 |

A combinação dos dois tipos de avaliação cria um sistema mais completo.

---

### Validação de sintaxe

Para validar a sintaxe, são criadas funções auxiliares.

#### Validação de JSON

```python
def validate_json(text):
    try:
        json.loads(text.strip())
        return 10
    except json.JSONDecodeError:
        return 0
```

Essa função tenta converter o texto em JSON usando `json.loads()`.

Se funcionar, retorna `10`.
Se falhar, retorna `0`.

---

#### Validação de Python

```python
def validate_python(text):
    try:
        ast.parse(text.strip())
        return 10
    except SyntaxError:
        return 0
```

Essa função usa `ast.parse()` para verificar se o código Python tem sintaxe válida.

Importante: ela valida sintaxe, mas não garante que o código funcione corretamente em tempo de execução.

---

#### Validação de Regex

```python
def validate_regex(text):
    try:
        re.compile(text.strip())
        return 10
    except re.error:
        return 0
```

Essa função tenta compilar a expressão regular com `re.compile()`.

Se a regex for válida, retorna `10`.
Se tiver erro de sintaxe, retorna `0`.

---

### Formato do dataset

Para o code grader saber qual validador usar, cada caso de teste precisa informar o formato esperado.

Exemplo:

```json
{
  "task": "Create a Python function to validate an AWS IAM username",
  "format": "python"
}
```

O campo importante aqui é:

```json
"format": "python"
```

Ele indica que a resposta deve ser validada como código Python.

Outros possíveis formatos seriam:

```json
"format": "json"
```

ou:

```json
"format": "regex"
```

---

### Melhorando a clareza do prompt

Para melhorar os resultados, o prompt deve deixar claro o formato esperado.

Exemplos de instruções úteis:

```text
Respond only with Python, JSON, or a plain Regex.
Do not add any comments or commentary or explanation.
```

Essas instruções reduzem a chance de Claude adicionar texto extra, como:

* introduções;
* comentários;
* markdown;
* explicações;
* cabeçalhos;
* rodapés.

---

### Assistant message prefilling

O conteúdo também sugere usar uma mensagem pré-preenchida do assistente:

````python
add_assistant_message(messages, "```code")
````

Isso incentiva Claude a começar a resposta como se já estivesse dentro de um bloco de código.

A ideia é direcionar o modelo para gerar apenas o conteúdo técnico, sem explicações.

No entanto, em uma aplicação real, ainda é importante limpar e validar a resposta, porque o modelo pode tentar fechar o bloco de código ou adicionar texto extra.

---

### Combinando scores

Depois de avaliar a saída com um model grader e um code grader, os scores podem ser combinados.

Exemplo:

```python
model_grade = grade_by_model(test_case, output)
model_score = model_grade["score"]
syntax_score = grade_syntax(output, test_case)

score = (model_score + syntax_score) / 2
```

Nesse exemplo, os dois critérios têm o mesmo peso:

* 50% qualidade/aderência à tarefa;
* 50% validade sintática/formato.

Essa média pode ser ajustada conforme o caso de uso.

Por exemplo, em uma aplicação que exige JSON perfeitamente válido, o score de sintaxe pode ter peso maior.

---

## 3. Pontos importantes para certificação

* **Code based grading** avalia saídas usando lógica programática.
* É especialmente útil para respostas que devem ser código, JSON ou regex.
* Code graders são bons para critérios objetivos.
* Os critérios principais são: **format, valid syntax e task following**.
* **Format** e **valid syntax** podem ser avaliados por código.
* **Task following** geralmente exige model grader.
* `json.loads()` pode validar se uma saída é JSON válido.
* `ast.parse()` pode validar sintaxe Python.
* `re.compile()` pode validar regex.
* O dataset deve incluir um campo como `"format"` para indicar o tipo esperado.
* Prompts mais claros reduzem respostas com explicações extras.
* Assistant prefilling pode ajudar Claude a gerar apenas código.
* Scores de model grader e code grader podem ser combinados por média.
* A pontuação serve principalmente para comparar versões de prompts, não como valor absoluto definitivo.

---

## 4. Termos-chave

**Code based grading**
Avaliação automatizada feita por código para verificar formato, sintaxe ou outras regras objetivas.

**Code grader**
Função ou sistema que avalia a saída programaticamente.

**Format**
Critério que verifica se a resposta está no tipo esperado, como Python, JSON ou Regex.

**Valid syntax**
Critério que verifica se a saída tem sintaxe válida.

**Task following**
Critério que verifica se a resposta realmente atende à tarefa solicitada.

**Model grader**
Modelo de IA usado para avaliar critérios mais semânticos, como qualidade e aderência à tarefa.

**`json.loads()`**
Função Python que tenta converter uma string JSON em objeto Python.

**`ast.parse()`**
Função Python que analisa sintaticamente código Python.

**`re.compile()`**
Função Python que compila uma expressão regular e verifica se ela é válida.

**Regex**
Expressão regular usada para buscar, validar ou extrair padrões de texto.

**Syntax score**
Pontuação baseada na validade sintática da resposta.

**Combined score**
Pontuação final resultante da combinação de diferentes graders.

**Assistant prefilling**
Técnica de pré-preencher a resposta do assistente para orientar o formato da geração.

---

## 5. Boas práticas

* Use code graders para checagens objetivas e automatizáveis.
* Valide JSON com `json.loads()`.
* Valide Python com `ast.parse()`.
* Valide regex com `re.compile()`.
* Inclua no dataset um campo `"format"` para informar o tipo de saída esperado.
* Combine code graders com model graders para uma avaliação mais completa.
* Dê instruções explícitas no prompt sobre o formato da resposta.
* Peça para Claude não incluir comentários, explicações, headers ou footers quando a saída precisar ser limpa.
* Use pontuações para comparar versões de prompts.
* Ajuste os pesos entre sintaxe e qualidade conforme o risco do caso de uso.
* Não trate uma pontuação alta como garantia absoluta de funcionamento.

---

## 6. Limitações, riscos e cuidados

Code graders são objetivos, mas limitados.

Por exemplo, `ast.parse()` valida se o código Python tem sintaxe correta, mas não verifica se a função faz o que deveria.

Da mesma forma, `json.loads()` valida se o JSON é sintaticamente válido, mas não garante que o JSON tenha os campos certos ou valores semanticamente corretos.

E `re.compile()` valida se a regex compila, mas não garante que ela capture exatamente os padrões desejados.

Por isso, code grading deve ser combinado com outros métodos, como:

* model grading;
* testes unitários;
* validação contra schemas;
* execução controlada em sandbox;
* revisão humana para casos críticos.

Outro cuidado: dar peso igual para model score e syntax score pode não ser ideal em todos os contextos. Em alguns casos, sintaxe válida pode ser requisito mínimo. Em outros, a aderência à tarefa pode ser mais importante.

---

## 7. Resumo para revisão rápida

* Code based grading avalia respostas com código.
* É útil para Python, JSON e Regex.
* Avalia formato e sintaxe válida.
* Task following costuma exigir model grader.
* `json.loads()` valida JSON.
* `ast.parse()` valida sintaxe Python.
* `re.compile()` valida regex.
* O dataset precisa indicar o formato esperado.
* Prompts claros ajudam a evitar explicações extras.
* Assistant prefilling pode orientar saída em código.
* Combine syntax score com model score.
* Use a média ou pesos personalizados.
* A pontuação serve para medir melhoria entre versões de prompt.

---

## 8. Perguntas simuladas

### 1. Qual é o objetivo principal do code based grading?

A) Avaliar respostas usando apenas opinião humana
B) Verificar programaticamente formato e sintaxe da saída
C) Criar prompts automaticamente
D) Substituir completamente o dataset de avaliação

**Resposta correta: B.**

**Explicação:** Code based grading usa código para verificar critérios objetivos, como formato e sintaxe válida.

---

### 2. Quais critérios são melhor avaliados por code graders?

A) Formato e sintaxe válida
B) Criatividade e empatia
C) Profundidade filosófica e humor
D) Preferência estética do usuário

**Resposta correta: A.**

**Explicação:** Formato e sintaxe são critérios objetivos que podem ser checados programaticamente.

---

### 3. Qual critério costuma ser melhor avaliado por um model grader?

A) Se o JSON parseia corretamente
B) Se a regex compila
C) Se a resposta seguiu corretamente a tarefa
D) Se o texto tem quebras de linha

**Resposta correta: C.**

**Explicação:** Task following exige julgamento semântico, por isso é mais adequado para model graders.

---

### 4. Qual função pode validar se uma string é JSON válido?

A) `ast.parse()`
B) `json.loads()`
C) `re.compile()`
D) `json.dump()`

**Resposta correta: B.**

**Explicação:** `json.loads()` tenta converter uma string JSON em objeto Python. Se falhar, o JSON é inválido.

---

### 5. Qual função pode validar a sintaxe de código Python?

A) `ast.parse()`
B) `json.loads()`
C) `re.compile()`
D) `print()`

**Resposta correta: A.**

**Explicação:** `ast.parse()` analisa sintaticamente código Python e lança `SyntaxError` se houver erro.

---

### 6. Qual função pode validar se uma regex compila?

A) `json.loads()`
B) `ast.parse()`
C) `re.compile()`
D) `mean()`

**Resposta correta: C.**

**Explicação:** `re.compile()` tenta compilar a regex. Se houver erro, lança `re.error`.

---

### 7. Por que o dataset precisa de um campo `"format"`?

A) Para indicar qual validador deve ser usado
B) Para armazenar a API key
C) Para calcular automaticamente a temperature
D) Para substituir o prompt

**Resposta correta: A.**

**Explicação:** O campo `"format"` informa se a saída esperada é Python, JSON ou Regex.

---

### 8. Qual instrução ajuda a reduzir explicações extras na saída?

A) “Respond only with Python, JSON, or a plain Regex.”
B) “Write a long essay before the code.”
C) “Always explain every line.”
D) “Ignore the requested format.”

**Resposta correta: A.**

**Explicação:** Essa instrução deixa claro que Claude deve retornar apenas o conteúdo no formato solicitado.

---

### 9. Como o exemplo combina model score e syntax score?

A) Usa apenas o menor score
B) Soma os dois sem dividir
C) Calcula a média dos dois
D) Ignora o syntax score

**Resposta correta: C.**

**Explicação:** O exemplo usa `(model_score + syntax_score) / 2`.

---

### 10. Qual é uma limitação de validar Python com `ast.parse()`?

A) Ele não verifica se o código resolve corretamente a tarefa
B) Ele nunca detecta erros de sintaxe
C) Ele só funciona com JSON
D) Ele executa o código automaticamente em produção

**Resposta correta: A.**

**Explicação:** `ast.parse()` verifica sintaxe, mas não garante corretude funcional ou aderência à tarefa.
