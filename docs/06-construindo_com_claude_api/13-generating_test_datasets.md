# Geração de conjuntos de dados de teste

```
007_prompt_evals.ipynb
```

## 1. Ideia central

O conteúdo explica como **gerar datasets de teste para avaliação de prompts**.

A ideia central é que, para avaliar se um prompt funciona bem, você precisa de um conjunto de entradas de teste, chamado **evaluation dataset**. Esse dataset será usado para alimentar o prompt e medir se Claude responde no formato esperado.

Neste exemplo, o objetivo é avaliar um prompt que gera soluções para tarefas relacionadas à AWS em três formatos:

* código Python;
* arquivos JSON;
* expressões regulares;
* sempre sem explicações extras, cabeçalhos ou rodapés.

---

## 2. Principais conceitos

### Objetivo da avaliação

O prompt que será avaliado deve ajudar usuários a criar soluções para tarefas da AWS.

Os formatos esperados são:

| Tipo de saída | Exemplo de uso                        |
| ------------- | ------------------------------------- |
| Python        | Funções simples para automação AWS    |
| JSON          | Configurações, políticas ou regras    |
| Regex         | Padrões para validar ou extrair dados |

A exigência principal é que Claude gere uma saída **limpa**, sem texto adicional.

Ou seja, não deve retornar:

* explicações;
* introduções;
* títulos;
* markdown desnecessário;
* comentários fora do formato pedido.

---

### Prompt inicial

O prompt inicial usado como baseline é simples:

```python id="653mi0"
prompt = f"""
Please provide a solution to the following task:
{task}
"""
```

Esse prompt recebe uma variável:

```python id="efdpww"
{task}
```

Essa variável será substituída por cada tarefa do dataset.

Esse tipo de prompt é chamado de **prompt template**.

---

### Evaluation dataset

O **evaluation dataset** é o conjunto de entradas que será usado para testar o prompt.

Neste caso, o dataset será uma lista de objetos JSON.

Cada objeto contém uma propriedade chamada `"task"`.

Exemplo conceitual:

```json id="38aw55"
[
  {
    "task": "Create a Python function that validates an AWS ARN."
  },
  {
    "task": "Generate a JSON IAM policy that allows read-only access to S3."
  },
  {
    "task": "Write a regex that matches AWS Lambda function ARNs."
  }
]
```

Cada tarefa será enviada ao prompt para verificar se Claude gera a resposta correta e no formato esperado.

---

### Gerar dataset manualmente ou com Claude

O conteúdo destaca duas formas de criar o dataset:

1. **manualmente**, escrevendo os casos de teste;
2. **automaticamente**, pedindo para Claude gerar os exemplos.

Neste caso, a aula usa Claude para gerar o dataset.

Como gerar dados de teste é uma tarefa mais simples, pode fazer sentido usar um modelo mais rápido e barato, como **Haiku**, em vez de um modelo mais poderoso.

---

### Funções auxiliares

A aula reutiliza funções auxiliares para montar mensagens e chamar Claude.

Função para adicionar mensagem do usuário:

```python id="v3ot2a"
def add_user_message(messages, text):
    user_message = {"role": "user", "content": text}
    messages.append(user_message)
```

Função para adicionar mensagem do assistente:

```python id="ty3zf2"
def add_assistant_message(messages, text):
    assistant_message = {"role": "assistant", "content": text}
    messages.append(assistant_message)
```

Função principal para chamar Claude:

```python id="94ww4g"
def chat(messages, system=None, temperature=1.0, stop_sequences=[]):
    params = {
        "model": model,
        "max_tokens": 1000,
        "messages": messages,
        "temperature": temperature
    }
    if system:
        params["system"] = system
    if stop_sequences:
        params["stop_sequences"] = stop_sequences
    
    response = client.messages.create(**params)
    return response.content[0].text
```

Essa função agora aceita:

* `system`;
* `temperature`;
* `stop_sequences`.

Isso permite maior controle da geração.

---

### Função `generate_dataset()`

A função `generate_dataset()` pede a Claude para criar um dataset em JSON.

Ela instrui Claude a gerar uma lista de objetos, cada um contendo uma tarefa relacionada a AWS.

Trecho principal:

````python id="eg99jh"
def generate_dataset():
    prompt = """
Generate an evaluation dataset for a prompt evaluation. The dataset will be used to evaluate prompts that generate Python, JSON, or Regex specifically for AWS-related tasks. Generate an array of JSON objects, each representing task that requires Python, JSON, or a Regex to complete.

Example output:
```json
[
  {
    "task": "Description of task",
  },
  ...additional
]
````

* Focus on tasks that can be solved by writing a single Python function, a single JSON object, or a single regex
* Focus on tasks that do not require writing much code

Please generate 3 objects.
"""

````

A função limita o escopo das tarefas para que elas sejam simples e adequadas à avaliação.

---

### Uso de prefilling e stop sequences

Para garantir que Claude retorne apenas JSON, a aula usa a mesma técnica vista anteriormente:

```python id="n1u5tz"
messages = []
add_user_message(messages, prompt)
add_assistant_message(messages, "```json")
text = chat(messages, stop_sequences=["```"])
return json.loads(text)
````

O que acontece aqui:

1. A mensagem do usuário pede um dataset.
2. A mensagem do assistente é pré-preenchida com `"```json"`.
3. Claude continua gerando apenas o conteúdo JSON.
4. A geração para quando Claude tenta fechar o bloco com `"```"`.
5. `json.loads(text)` converte o texto em objeto Python.

Essa técnica ajuda a obter uma saída estruturada e fácil de processar.

---

### Testando a geração do dataset

Depois de criar a função, ela é executada:

```python id="dku1pi"
dataset = generate_dataset()
print(dataset)
```

O resultado esperado é uma lista com três objetos, cada um contendo uma tarefa de avaliação.

Essas tarefas devem cobrir os três tipos de saída desejados:

* Python;
* JSON;
* Regex.

---

### Salvando o dataset

Depois de gerar o dataset, ele é salvo em um arquivo:

```python id="fmb15m"
with open('dataset.json', 'w') as f:
    json.dump(dataset, f, indent=2)
```

Isso cria um arquivo:

```text id="zqs8hw"
dataset.json
```

Esse arquivo pode ser carregado depois durante o processo de avaliação.

Salvar o dataset é importante porque garante que as próximas avaliações usem os mesmos casos de teste, permitindo comparação justa entre versões de prompts.

---

## 3. Pontos importantes para certificação

* Um prompt evaluation workflow precisa de um **evaluation dataset**.
* O dataset contém entradas usadas para testar o prompt.
* Neste exemplo, cada entrada é um objeto JSON com a propriedade `"task"`.
* O prompt avaliado deve gerar soluções AWS em Python, JSON ou Regex.
* A saída esperada deve ser limpa, sem explicações, headers ou footers.
* O dataset pode ser criado manualmente ou gerado por Claude.
* Para gerar dados de teste, pode ser eficiente usar um modelo mais rápido e barato.
* A função `chat()` pode aceitar `system`, `temperature` e `stop_sequences`.
* Prefilling com `"```json"` ajuda Claude a gerar JSON.
* `stop_sequences=["```"]` interrompe a geração antes de explicações adicionais.
* `json.loads(text)` converte o JSON gerado em objeto Python.
* Salvar o dataset em `dataset.json` permite reutilizar os mesmos testes.
* Usar o mesmo dataset é essencial para comparar versões de prompts de forma objetiva.

---

## 4. Termos-chave

**Evaluation dataset**
Conjunto de entradas usadas para testar um prompt.

**Test dataset**
Dataset com casos de teste para medir o comportamento do modelo.

**Prompt template**
Prompt com variáveis, como `{task}`, que serão substituídas por valores reais.

**Task**
Descrição da tarefa que Claude deve resolver.

**AWS-specific task**
Tarefa relacionada a serviços, configurações ou padrões da AWS.

**Python function**
Função Python usada como saída esperada em algumas tarefas.

**JSON configuration**
Arquivo ou objeto JSON usado para representar configurações.

**Regular expression / Regex**
Padrão textual usado para validar ou extrair informações.

**Assistant message prefilling**
Técnica de pré-preencher a resposta do assistente para orientar o formato de saída.

**Stop sequence**
Sequência que interrompe a geração quando aparece.

**`json.loads()`**
Função que converte uma string JSON em objeto Python.

**`json.dump()`**
Função que grava um objeto Python em arquivo JSON.

**`indent=2`**
Opção que formata o JSON com indentação legível.

**Dataset generation**
Processo de gerar exemplos de teste, manualmente ou com ajuda de um modelo.

---

## 5. Boas práticas

* Criar datasets representativos do uso real da aplicação.
* Incluir diferentes tipos de saída esperada, como Python, JSON e Regex.
* Manter tarefas pequenas e focadas quando o objetivo é avaliar formato e aderência.
* Usar Claude para gerar dados de teste quando isso acelerar o processo.
* Usar modelos mais rápidos e baratos para geração de datasets simples.
* Usar prefilling e stop sequences para obter JSON limpo.
* Validar o JSON com `json.loads()` antes de usar.
* Salvar o dataset em arquivo para reutilização.
* Usar o mesmo dataset ao comparar versões do prompt.
* Expandir o dataset com casos novos conforme surgirem falhas.

---

## 6. Limitações, riscos e cuidados

Gerar o dataset com Claude é prático, mas pode produzir exemplos pouco variados, muito fáceis ou semelhantes entre si.

Por isso, em aplicações reais, é recomendável revisar o dataset manualmente e adicionar casos importantes, como:

* entradas ambíguas;
* tarefas mal formuladas;
* solicitações fora do escopo;
* casos que exigem JSON válido;
* casos que exigem regex específica;
* tarefas AWS com nomes de serviços parecidos.

Outro cuidado é que o dataset gerado tem apenas três objetos. Isso é suficiente para uma demonstração, mas não para uma avaliação robusta em produção.

Também é importante validar se o JSON gerado está correto. Mesmo usando prefilling e stop sequences, o modelo pode gerar JSON inválido.

Por fim, ao usar AWS como domínio, é importante verificar se os exemplos fazem sentido tecnicamente. Um dataset com tarefas incorretas pode prejudicar a avaliação do prompt.

---

## 7. Resumo para revisão rápida

* Para avaliar prompts, você precisa de um dataset de teste.
* O dataset contém entradas que serão inseridas no prompt.
* Neste exemplo, cada entrada tem uma propriedade `"task"`.
* O prompt avaliado gera Python, JSON ou Regex para tarefas AWS.
* A saída esperada deve ser limpa, sem explicações.
* O dataset pode ser criado manualmente ou gerado por Claude.
* Usar Haiku pode ser adequado para gerar dados simples.
* Prefill com `"```json"` ajuda a forçar saída em JSON.
* Stop sequence `"```"` evita markdown e explicações extras.
* `json.loads()` transforma o texto em objeto Python.
* `json.dump()` salva o dataset em `dataset.json`.
* Reutilizar o dataset permite comparar prompts de forma justa.

---

## 8. Perguntas simuladas

### 1. Para que serve um evaluation dataset?

A) Para armazenar API keys
B) Para testar um prompt com entradas representativas
C) Para escolher automaticamente o modelo mais caro
D) Para impedir Claude de gerar respostas

**Resposta correta: B.**

**Explicação:** O evaluation dataset contém exemplos que serão usados para medir o desempenho do prompt.

---

### 2. No exemplo, qual propriedade cada objeto do dataset deve conter?

A) `"model"`
B) `"temperature"`
C) `"task"`
D) `"api_key"`

**Resposta correta: C.**

**Explicação:** Cada objeto JSON contém uma propriedade `"task"` descrevendo a tarefa que Claude deve resolver.

---

### 3. Quais são os três tipos de saída que o prompt deve gerar no exemplo?

A) HTML, CSS e SQL
B) Python, JSON e Regex
C) Markdown, PDF e YAML
D) Java, XML e imagens

**Resposta correta: B.**

**Explicação:** O objetivo é gerar soluções AWS em Python, JSON ou expressões regulares.

---

### 4. Qual é a principal exigência sobre a saída gerada?

A) Deve conter longas explicações
B) Deve incluir headers e footers
C) Deve ser limpa, sem texto extra
D) Deve estar sempre em português

**Resposta correta: C.**

**Explicação:** A saída deve conter apenas o código, JSON ou regex necessário, sem explicações adicionais.

---

### 5. Por que usar um modelo mais rápido como Haiku para gerar datasets?

A) Porque geração de dados de teste pode ser uma tarefa simples e de menor custo
B) Porque Haiku é o único modelo que aceita JSON
C) Porque modelos maiores não conseguem gerar datasets
D) Porque Haiku elimina a necessidade de validação

**Resposta correta: A.**

**Explicação:** Para tarefas simples como geração de casos de teste, modelos mais rápidos e baratos podem ser suficientes.

---

### 6. Para que serve `add_assistant_message(messages, "```json")`?

A) Para pré-preencher a resposta e orientar Claude a continuar em JSON
B) Para encerrar imediatamente a conversa
C) Para transformar Python em Regex
D) Para salvar o arquivo no disco

**Resposta correta: A.**

**Explicação:** Esse é o uso de assistant message prefilling para direcionar a saída ao formato JSON.

---

### 7. Por que usar `stop_sequences=["```"]`?

A) Para parar a geração quando Claude tentar fechar o bloco de código
B) Para impedir que o JSON seja validado
C) Para aumentar a temperature
D) Para trocar automaticamente o modelo

**Resposta correta: A.**

**Explicação:** A stop sequence interrompe a geração ao encontrar as crases de fechamento do bloco markdown.

---

### 8. O que faz `json.loads(text)`?

A) Salva o dataset em arquivo
B) Converte uma string JSON em objeto Python
C) Cria uma nova API key
D) Remove todas as tarefas do dataset

**Resposta correta: B.**

**Explicação:** `json.loads()` lê uma string JSON e a transforma em estruturas Python, como listas e dicionários.

---

### 9. O que faz `json.dump(dataset, f, indent=2)`?

A) Grava o dataset em um arquivo JSON formatado
B) Envia o dataset para Claude
C) Remove as stop sequences
D) Converte JSON em regex

**Resposta correta: A.**

**Explicação:** `json.dump()` escreve o objeto Python em arquivo JSON, e `indent=2` melhora a legibilidade.

---

### 10. Por que salvar o dataset em `dataset.json`?

A) Para reutilizar os mesmos casos de teste em avaliações futuras
B) Para expor a API key no repositório
C) Para impedir que o prompt seja alterado
D) Para substituir o uso de Claude

**Resposta correta: A.**

**Explicação:** Reutilizar o mesmo dataset permite comparar versões de prompts de forma consistente e objetiva.
