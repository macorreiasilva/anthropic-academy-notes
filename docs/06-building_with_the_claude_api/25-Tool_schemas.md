# Esquemas de ferramentas

```
notebooks\14-tool_functions.ipynb
```

## 1. Ideia central

O conteúdo explica a segunda etapa do fluxo de criação de tools: depois de escrever a **tool function**, é necessário criar um **JSON schema** para descrever essa função para Claude.

A ideia central é:

**Claude precisa de uma descrição estruturada da ferramenta para saber quando usá-la, quais argumentos ela aceita e como chamar a função corretamente.**

Ou seja, a função Python existe no seu código, mas Claude só consegue utilizá-la bem se receber uma especificação clara em formato de schema.

---

## 2. Principais conceitos

### O que é um tool schema

Um **tool schema** é uma especificação em JSON que descreve uma ferramenta disponível para Claude.

Ele funciona como uma documentação estruturada da função.

A especificação normalmente contém três partes principais:

| Campo          | Função                                                      |
| -------------- | ----------------------------------------------------------- |
| `name`         | Nome da ferramenta                                          |
| `description`  | Explica o que a ferramenta faz, quando usar e o que retorna |
| `input_schema` | Define os argumentos esperados pela função                  |

Exemplo conceitual:

```python
get_current_datetime_schema = {
    "name": "get_current_datetime",
    "description": "Returns the current date and time formatted according to the specified format",
    "input_schema": {
        "type": "object",
        "properties": {
            "date_format": {
                "type": "string",
                "description": "A string specifying the format of the returned datetime. Uses Python's strftime format codes.",
                "default": "%Y-%m-%d %H:%M:%S"
            }
        },
        "required": []
    }
}
```

Esse schema diz a Claude que existe uma ferramenta chamada `get_current_datetime`, que ela aceita um argumento opcional chamado `date_format`, e que esse argumento deve ser uma string.

---

### JSON Schema não é exclusivo de IA

Um ponto importante da aula é que **JSON Schema não é algo criado apenas para LLMs**.

JSON Schema é uma especificação amplamente usada para:

* validar dados;
* definir estruturas esperadas;
* documentar formatos de entrada;
* garantir que objetos JSON sigam regras específicas.

A comunidade de IA adotou JSON Schema porque ele é uma forma conveniente de descrever argumentos de funções e validar entradas.

---

### Campo `name`

O campo `name` define o nome da ferramenta.

Exemplo:

```json
"name": "get_current_datetime"
```

Esse nome deve ser claro e descritivo, porque Claude usa essa informação para identificar qual ferramenta chamar.

Boas práticas:

* usar nomes específicos;
* evitar nomes vagos;
* manter correspondência com a função Python.

Exemplo bom:

```python
get_current_datetime()
```

Exemplo menos claro:

```python
get_data()
```

---

### Campo `description`

O campo `description` é uma das partes mais importantes do schema.

Ele ajuda Claude a entender:

* o que a ferramenta faz;
* quando ela deve ser usada;
* que tipo de dado ela retorna.

Exemplo:

```json
"description": "Returns the current date and time formatted according to the specified format"
```

A aula recomenda que descrições de tools sejam detalhadas, idealmente com **3 a 4 frases**, explicando claramente o comportamento da ferramenta.

Uma descrição fraca pode fazer Claude deixar de usar a ferramenta quando deveria ou usá-la no momento errado.

---

### Campo `input_schema`

O campo `input_schema` descreve os argumentos da função.

Exemplo:

```json
"input_schema": {
    "type": "object",
    "properties": {
        "date_format": {
            "type": "string",
            "description": "A string specifying the format of the returned datetime. Uses Python's strftime format codes.",
            "default": "%Y-%m-%d %H:%M:%S"
        }
    },
    "required": []
}
```

Nesse caso, o schema informa que:

* a entrada é um objeto;
* existe uma propriedade chamada `date_format`;
* `date_format` deve ser uma string;
* ela tem um valor padrão;
* nenhum campo é obrigatório, porque `required` está vazio.

---

### Campo `required`

O campo `required` indica quais argumentos são obrigatórios.

No exemplo:

```json
"required": []
```

Isso significa que Claude pode chamar a função sem fornecer nenhum argumento.

Isso faz sentido porque a função Python já tem um valor padrão:

```python
def get_current_datetime(date_format="%Y-%m-%d %H:%M:%S"):
```

Se a função exigisse um argumento obrigatório, ele deveria aparecer em `required`.

---

### Usar Claude para gerar schemas

O conteúdo também mostra uma prática útil: usar o próprio Claude para gerar o JSON schema.

O processo recomendado é:

1. Copiar a função Python.
2. Pedir a Claude para criar um JSON schema válido para tool calling.
3. Incluir a documentação de boas práticas da Anthropic como contexto.
4. Revisar o schema gerado.
5. Copiar o schema para o código.

Exemplo de prompt:

```text
Write a valid JSON schema spec for the purposes of tool calling for this function.
Follow the best practices listed in the attached documentation.
```

Isso acelera o desenvolvimento, mas ainda exige revisão humana.

---

### Padrão de nomeação

A aula recomenda um padrão simples para organizar schemas:

```python
nome_da_funcao_schema
```

Exemplo:

```python
def get_current_datetime(...):
    ...

get_current_datetime_schema = {
    ...
}
```

Esse padrão facilita encontrar qual schema corresponde a qual função.

---

### Type safety com `ToolParam`

Para melhorar checagem de tipos, o conteúdo mostra o uso de `ToolParam` da biblioteca da Anthropic:

```python
from anthropic.types import ToolParam

get_current_datetime_schema = ToolParam({
    "name": "get_current_datetime",
    "description": "Returns the current date and time formatted according to the specified format",
    "input_schema": {
        ...
    }
})
```

Isso não é estritamente obrigatório para a funcionalidade, mas ajuda a evitar erros de tipo ao usar o schema com a API.

---

## 3. Pontos importantes para certificação

* Depois de criar uma tool function, é necessário criar um **JSON schema**.
* O schema descreve a ferramenta para Claude.
* Claude usa o schema para entender **quando e como chamar a tool**.
* Um tool schema tem três partes principais: `name`, `description` e `input_schema`.
* `name` deve ser claro e corresponder à função.
* `description` deve explicar o que a ferramenta faz, quando usar e o que retorna.
* `input_schema` descreve os argumentos da função.
* JSON Schema é uma especificação geral de validação de dados, não algo exclusivo de LLMs.
* Descrições detalhadas ajudam Claude a usar tools corretamente.
* É possível usar Claude para ajudar a gerar schemas.
* O schema gerado deve ser revisado antes de ir para o código.
* Um bom padrão é usar `function_name_schema`.
* `ToolParam` pode ser usado para melhorar type safety.
* O campo `required` deve refletir quais argumentos são realmente obrigatórios.

---

## 4. Termos-chave

**Tool schema**
Especificação em JSON que descreve uma ferramenta para Claude.

**JSON Schema**
Padrão usado para descrever e validar estruturas de dados JSON.

**`name`**
Campo que define o nome da tool.

**`description`**
Campo que explica o propósito da ferramenta, quando usá-la e o que ela retorna.

**`input_schema`**
Campo que descreve os argumentos aceitos pela ferramenta.

**`properties`**
Lista de parâmetros que a função pode receber.

**`required`**
Lista de parâmetros obrigatórios.

**`default`**
Valor padrão de um parâmetro.

**Tool calling**
Processo em que Claude solicita a execução de uma ferramenta.

**ToolParam**
Tipo da biblioteca Anthropic usado para melhorar checagem de tipos em schemas de tools.

**Type safety**
Prática de reduzir erros relacionados a tipos de dados no código.

---

## 5. Boas práticas

* Escreva nomes de tools claros e descritivos.
* Faça o nome do schema corresponder ao nome da função.
* Use descrições detalhadas para a tool.
* Explique na descrição:

  * o que a ferramenta faz;
  * quando Claude deve usá-la;
  * o que a ferramenta retorna.
* Descreva cada argumento com detalhes.
* Informe tipos corretos em `input_schema`.
* Use `required` apenas para parâmetros realmente obrigatórios.
* Revise schemas gerados por Claude antes de usar.
* Use `ToolParam` quando quiser mais segurança de tipos.
* Mantenha tools e schemas organizados lado a lado no código.

Exemplo de organização:

```python
def get_current_datetime(date_format="%Y-%m-%d %H:%M:%S"):
    ...

get_current_datetime_schema = ToolParam({
    "name": "get_current_datetime",
    "description": "...",
    "input_schema": {...}
})
```

---

## 6. Limitações, riscos e cuidados

Um schema mal escrito pode fazer Claude usar a ferramenta incorretamente.

Riscos comuns:

* descrição vaga;
* nome pouco claro;
* argumentos mal descritos;
* campo obrigatório ausente;
* tipo incorreto no `input_schema`;
* schema diferente da função real;
* descrição que não explica quando usar a ferramenta.

Outro cuidado importante: se a função Python tem argumento opcional, o schema também deve refletir isso. No exemplo, `date_format` é opcional porque a função já define um valor padrão.

Também é importante lembrar que usar Claude para gerar schemas é útil, mas não elimina a necessidade de revisão. Claude pode gerar um schema sintaticamente plausível, mas desalinhado com a função real.

---

## 7. Resumo para revisão rápida

* Tool function é a função Python.
* Tool schema é a descrição da função para Claude.
* JSON Schema descreve argumentos esperados.
* O schema ajuda Claude a saber quando e como usar a tool.
* Campos principais: `name`, `description`, `input_schema`.
* `description` é crucial para orientar o uso da ferramenta.
* `input_schema` define tipos, propriedades e obrigatoriedade.
* `required` lista argumentos obrigatórios.
* Claude pode ajudar a gerar schemas.
* Sempre revise o schema gerado.
* Use o padrão `function_name_schema`.
* `ToolParam` melhora type safety.
* Bons schemas tornam tool use mais confiável.

---

## 8. Perguntas simuladas

### 1. Para que serve um tool schema?

A) Para armazenar a API key
B) Para descrever uma ferramenta para Claude, incluindo nome, descrição e argumentos
C) Para executar Python automaticamente dentro do modelo
D) Para substituir o prompt do usuário

**Resposta correta: B.**

**Explicação:** O schema informa a Claude como a ferramenta funciona e quais argumentos ela aceita.

---

### 2. Quais são as três partes principais de uma especificação de tool?

A) `temperature`, `max_tokens` e `messages`
B) `name`, `description` e `input_schema`
C) `user`, `assistant` e `system`
D) `json`, `python` e `regex`

**Resposta correta: B.**

**Explicação:** Esses três campos descrevem a tool para Claude.

---

### 3. Qual é a função do campo `name`?

A) Definir o nome da ferramenta
B) Definir o limite máximo de tokens
C) Armazenar o resultado da ferramenta
D) Criar uma variável de ambiente

**Resposta correta: A.**

**Explicação:** `name` identifica a ferramenta que Claude pode solicitar.

---

### 4. Por que o campo `description` é importante?

A) Porque ajuda Claude a entender quando usar a ferramenta e o que ela retorna
B) Porque substitui todos os argumentos da função
C) Porque impede validação de dados
D) Porque define o preço da chamada

**Resposta correta: A.**

**Explicação:** A descrição orienta Claude sobre o propósito e o uso correto da tool.

---

### 5. O que o campo `input_schema` descreve?

A) A interface visual do usuário
B) Os argumentos aceitos pela função
C) O histórico da conversa
D) O nome do modelo Claude

**Resposta correta: B.**

**Explicação:** `input_schema` define os parâmetros, tipos e campos obrigatórios da ferramenta.

---

### 6. JSON Schema é exclusivo de LLMs?

A) Sim, foi criado apenas para Claude
B) Não, é uma especificação amplamente usada para validação de dados
C) Sim, só funciona com tool use
D) Não, mas não serve para validação

**Resposta correta: B.**

**Explicação:** JSON Schema já era usado para validar e documentar dados antes de ser adotado em tool calling.

---

### 7. No exemplo, por que `required` está vazio?

A) Porque `date_format` é opcional e tem valor padrão na função
B) Porque a ferramenta não tem nome
C) Porque Claude não aceita argumentos obrigatórios
D) Porque o schema está incompleto por definição

**Resposta correta: A.**

**Explicação:** Como `date_format` tem valor padrão, Claude pode chamar a ferramenta sem informar esse argumento.

---

### 8. Qual padrão de nomeação é recomendado para schemas?

A) `schema1`, `schema2`, `schema3`
B) `function_name_schema`
C) `tool_random_name`
D) `json_prompt_final`

**Resposta correta: B.**

**Explicação:** Usar o nome da função seguido de `_schema` facilita organização e manutenção.

---

### 9. Para que serve `ToolParam`?

A) Para melhorar a checagem de tipos ao usar schemas com a API da Anthropic
B) Para executar a ferramenta automaticamente
C) Para criar lembretes sem servidor
D) Para substituir JSON Schema

**Resposta correta: A.**

**Explicação:** `ToolParam` ajuda a tornar o código mais robusto contra erros de tipo.

---

### 10. Qual é uma boa prática ao escrever descrições de tools?

A) Usar descrições vagas e curtas demais
B) Explicar o que a ferramenta faz, quando usar e o que retorna
C) Omitir todos os detalhes dos argumentos
D) Usar nomes genéricos como `tool1`

**Resposta correta: B.**

**Explicação:** Descrições detalhadas ajudam Claude a escolher e usar a ferramenta corretamente.
