# Funções da ferramenta

```
notebooks\14-tool_functions.ipynb
```

## 1. Ideia central

O conteúdo explica o que são **tool functions** no contexto de aplicações com Claude.

A ideia central é que uma **tool function** é uma função Python comum que pode ser executada pelo sistema quando Claude precisa de informação externa ou precisa realizar uma ação.

Exemplo:

Se o usuário perguntar:

```text
What time is it?
```

Claude pode decidir que precisa da hora atual e solicitar a execução de uma ferramenta como:

```python
get_current_datetime()
```

Isso permite que Claude acesse informações que ele não possui internamente, como data/hora atual, clima, banco de dados, lembretes ou outras ações do sistema.

---

## 2. Principais conceitos

### O que são tool functions

Uma **tool function** é uma função implementada no backend da aplicação.

Ela pode ser chamada quando Claude identifica que precisa de dados extras ou de uma ação externa para responder ao usuário.

Importante:

**Claude não executa Python diretamente por conta própria.**
Claude solicita o uso da ferramenta, e o seu sistema executa a função Python correspondente.

---

### Exemplo do projeto de lembretes

No projeto apresentado, serão criadas três ferramentas principais:

| Tool function              | Função                            |
| -------------------------- | --------------------------------- |
| `get_current_datetime`     | Obter a data e hora atual         |
| `add_duration_to_datetime` | Somar uma duração a uma data/hora |
| `set_reminder`             | Criar um lembrete no sistema      |

A primeira ferramenta implementada é a de obter data e hora atual.

---

### Primeira tool function: `get_current_datetime`

A função apresentada é:

```python
def get_current_datetime(date_format="%Y-%m-%d %H:%M:%S"):
    if not date_format:
        raise ValueError("date_format cannot be empty")
    return datetime.now().strftime(date_format)
```

Essa função faz três coisas:

1. recebe um formato de data opcional;
2. valida se o formato não está vazio;
3. retorna a data e hora atual no formato solicitado.

O formato padrão é:

```text
%Y-%m-%d %H:%M:%S
```

Isso gera algo como:

```text
2024-01-15 14:30:25
```

---

### Parâmetro `date_format`

O parâmetro `date_format` permite controlar o formato da resposta.

Exemplo com formato padrão:

```python
get_current_datetime()
```

Saída esperada:

```text
2024-01-15 14:30:25
```

Exemplo retornando apenas hora e minuto:

```python
get_current_datetime("%H:%M")
```

Saída esperada:

```text
14:30
```

Esse parâmetro dá flexibilidade para Claude pedir a data/hora no formato mais útil para a tarefa.

---

### Validação de entrada

A função verifica se `date_format` está vazio:

```python
if not date_format:
    raise ValueError("date_format cannot be empty")
```

Isso é importante porque tools devem ser robustas.

Se Claude passar um argumento inválido, a ferramenta retorna um erro claro. Claude pode usar essa mensagem de erro para tentar novamente com parâmetros corrigidos.

Exemplo de erro útil:

```text
date_format cannot be empty
```

Esse tipo de mensagem ajuda Claude a entender o que precisa ser corrigido.

---

### Próximo passo: JSON schema

Criar a função Python é apenas a primeira parte.

Depois, será necessário criar um **JSON schema** que descreve a ferramenta para Claude.

Esse schema informa:

* nome da ferramenta;
* descrição da ferramenta;
* parâmetros aceitos;
* tipos dos parâmetros;
* quais parâmetros são obrigatórios;
* o que cada parâmetro significa.

Sem esse schema, Claude não sabe quando ou como chamar a função.

---

## 3. Pontos importantes para certificação

* Tool functions são funções Python comuns executadas pelo backend.
* Claude pode solicitar uma tool quando precisa de dados externos ou ações.
* O servidor executa a função; Claude não executa o código diretamente.
* Tool functions permitem acesso a informações em tempo real.
* No projeto, as tools principais são: obter data/hora, somar duração e criar lembrete.
* Funções devem ter nomes descritivos.
* Parâmetros também devem ter nomes claros.
* Inputs devem ser validados.
* Erros devem ser claros e úteis.
* Claude pode usar mensagens de erro para corrigir uma nova chamada.
* `get_current_datetime()` usa `datetime.now()` para obter a hora atual.
* `strftime()` formata a data/hora conforme o formato solicitado.
* Depois de criar a função, é necessário descrevê-la para Claude com um JSON schema.

---

## 4. Termos-chave

**Tool function**
Função Python disponibilizada para Claude solicitar dados ou ações externas.

**Backend**
Camada da aplicação que executa as tools e controla acesso a sistemas externos.

**`get_current_datetime`**
Função que retorna a data e hora atual.

**`date_format`**
Parâmetro que define o formato da data/hora retornada.

**`datetime.now()`**
Função Python que obtém a data e hora atual do sistema.

**`strftime()`**
Método usado para formatar data e hora em string.

**Input validation**
Verificação de que os parâmetros recebidos são válidos.

**Error message**
Mensagem retornada quando algo dá errado, idealmente clara e acionável.

**JSON schema**
Descrição estruturada da ferramenta para Claude entender como chamá-la.

**Tool calling**
Processo em que Claude solicita que uma ferramenta seja executada com parâmetros específicos.

---

## 5. Boas práticas

Use nomes descritivos para funções.

Bom exemplo:

```python
get_current_datetime()
```

Menos claro:

```python
get_time()
```

Use nomes claros para parâmetros.

Bom exemplo:

```python
date_format
```

Menos claro:

```python
fmt
```

Valide os inputs antes de executar a lógica principal.

Exemplo:

```python
if not date_format:
    raise ValueError("date_format cannot be empty")
```

Forneça mensagens de erro úteis, porque Claude pode interpretá-las e tentar corrigir a chamada.

Mantenha cada tool focada em uma responsabilidade específica. Uma ferramenta que só retorna data e hora é mais fácil de testar e manter do que uma ferramenta que tenta fazer tudo.

---

## 6. Limitações, riscos e cuidados

Uma tool function mal implementada pode causar erros, respostas incorretas ou falhas no sistema.

Por exemplo, se `date_format` for inválido, a função pode falhar. Por isso, em produção, pode ser útil validar também se o formato é aceitável ou tratar exceções de `strftime`.

Outro cuidado é que a hora retornada depende do ambiente onde o código roda. Se o servidor estiver em UTC e o usuário estiver em outro fuso horário, o lembrete pode ser criado no horário errado.

Para aplicações reais de lembrete, é importante considerar:

* fuso horário do usuário;
* data e hora atual do servidor;
* horários ambíguos;
* formatos inválidos;
* permissões;
* logs;
* confirmação antes de criar ações sensíveis.

Também é importante lembrar que criar a função Python não basta. Claude precisa de uma descrição estruturada da ferramenta para saber como usá-la.

---

## 7. Resumo para revisão rápida

* Tool functions são funções Python que Claude pode solicitar.
* Elas permitem acessar dados atuais ou executar ações.
* O servidor executa as funções.
* Claude apenas decide quando precisa da tool e com quais parâmetros.
* A primeira tool do projeto é `get_current_datetime`.
* Ela retorna a data e hora atual.
* O parâmetro `date_format` define o formato da resposta.
* `datetime.now()` obtém a hora atual.
* `strftime()` formata a data/hora.
* Inputs devem ser validados.
* Mensagens de erro devem ser claras.
* O próximo passo é criar um JSON schema para descrever a tool para Claude.

---

## 8. Perguntas simuladas

### 1. O que é uma tool function?

A) Uma função Python que pode ser executada pelo sistema quando Claude precisa de dados ou ações externas
B) Um tipo de API key
C) Um prompt que nunca muda
D) Um formato especial de markdown

**Resposta correta: A.**

**Explicação:** Tool functions são funções implementadas no backend que Claude pode solicitar durante uma interação.

---

### 2. Quem executa a tool function?

A) Claude diretamente dentro do modelo
B) O servidor ou backend da aplicação
C) O usuário manualmente sempre
D) O arquivo README

**Resposta correta: B.**

**Explicação:** Claude solicita a ferramenta, mas a execução real acontece no servidor da aplicação.

---

### 3. Qual é a primeira tool function implementada no projeto?

A) `set_reminder`
B) `get_current_datetime`
C) `delete_user`
D) `generate_image`

**Resposta correta: B.**

**Explicação:** A primeira ferramenta obtém a data e hora atual.

---

### 4. Para que serve o parâmetro `date_format`?

A) Para definir o formato da data/hora retornada
B) Para escolher o modelo Claude
C) Para armazenar a API key
D) Para criar automaticamente um lembrete

**Resposta correta: A.**

**Explicação:** `date_format` controla como a data e hora serão formatadas.

---

### 5. O que faz `datetime.now()`?

A) Retorna a data e hora atual do sistema
B) Cria um JSON schema
C) Envia uma mensagem para Claude
D) Valida uma regex

**Resposta correta: A.**

**Explicação:** `datetime.now()` obtém o momento atual no ambiente onde o código está rodando.

---

### 6. O que faz `strftime()`?

A) Formata uma data/hora como string
B) Cria uma API key
C) Executa uma chamada HTTP
D) Apaga um lembrete

**Resposta correta: A.**

**Explicação:** `strftime()` converte uma data/hora para uma string seguindo um formato especificado.

---

### 7. Por que validar inputs em tool functions?

A) Para garantir que parâmetros inválidos sejam detectados e tratados com mensagens úteis
B) Para impedir Claude de usar qualquer ferramenta
C) Para aumentar a temperature
D) Para substituir o prompt do usuário

**Resposta correta: A.**

**Explicação:** Validação evita entradas inválidas e ajuda Claude a corrigir chamadas quando há erro.

---

### 8. Qual mensagem de erro é usada no exemplo?

A) `"model cannot be empty"`
B) `"date_format cannot be empty"`
C) `"API key is public"`
D) `"messages missing"`

**Resposta correta: B.**

**Explicação:** A função lança `ValueError("date_format cannot be empty")` quando o formato está vazio.

---

### 9. Qual é o próximo passo depois de criar a função Python?

A) Criar um JSON schema que descreva a ferramenta para Claude
B) Apagar a função
C) Colocar a API key dentro da função
D) Remover validações

**Resposta correta: A.**

**Explicação:** Claude precisa de uma descrição estruturada da ferramenta para saber como chamá-la.

---

### 10. Qual é uma boa prática ao criar tool functions?

A) Usar nomes vagos e mensagens de erro genéricas
B) Usar nomes descritivos, validar entradas e retornar erros claros
C) Nunca validar parâmetros
D) Misturar todas as ações em uma única função enorme

**Resposta correta: B.**

**Explicação:** Tools devem ser claras, robustas e fáceis de usar pelo modelo e pelo sistema.
