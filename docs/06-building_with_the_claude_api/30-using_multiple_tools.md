# Utilizando múltiplas ferramentas

```
notebooks\17-using_multiple_tools.ipynb
```

## 1. Ideia central

A aula explica como usar **múltiplas ferramentas** em uma implementação com Claude.

Depois que a infraestrutura básica de tools já existe — loop de conversa, detecção de `tool_use`, execução de ferramentas, retorno de `tool_result` e roteamento por nome — adicionar novas ferramentas se torna um processo simples e modular.

O exemplo da aula constrói um sistema de lembretes que usa três ferramentas:

1. Obter a data e hora atual.
2. Somar uma duração a uma data.
3. Criar um lembrete.

A ideia central é: **Claude pode combinar várias ferramentas em sequência para resolver uma tarefa mais complexa, enquanto a aplicação executa cada ferramenta e devolve os resultados.**

---

## 2. Principais conceitos

### Ferramentas necessárias no exemplo

A aula apresenta três ferramentas principais para criar um sistema de lembretes.

### `get_current_datetime`

Usada quando Claude precisa saber a data e hora atuais.

Exemplo de necessidade:

```text
What day is 103 days from today?
```

Claude não deve depender apenas do próprio conhecimento para saber a data atual. Ele deve chamar uma ferramenta que retorna a data/hora real.

---

### `add_duration_to_datetime`

Usada para somar uma duração a uma data.

Exemplo:

```text
177 days after Jan 1st, 2050
```

Claude poderia tentar calcular isso sozinho, mas a aula destaca que ele **não é perfeito com adição de datas e horários**. Por isso, é melhor usar uma ferramenta especializada.

Essa é uma boa prática importante: use tools para operações que exigem precisão, especialmente cálculos de data, hora, valores ou dados externos.

---

### `set_reminder`

Usada para criar o lembrete depois que Claude já sabe a data correta.

Exemplo:

```text
Set a reminder for my doctor's appointment.
```

Nesse caso, Claude precisa primeiro descobrir **quando** será o compromisso e só depois chamar a ferramenta de lembrete.

---

### Adicionando várias tools à conversa

Para disponibilizar várias ferramentas a Claude, basta incluir todos os schemas na lista `tools`.

Exemplo da aula:

```python
response = chat(messages, tools=[
    get_current_datetime_schema,
    add_duration_to_datetime_schema,
    set_reminder_schema
])
```

Isso informa a Claude que ele pode usar qualquer uma dessas três ferramentas durante a conversa.

Ponto importante: Claude escolhe qual ferramenta usar com base no pedido do usuário, na descrição dos schemas e no estado atual da conversa.

---

### Atualizando o roteador de ferramentas

Depois de adicionar os schemas, também é necessário atualizar a função que executa as tools.

Exemplo:

```python
def run_tool(tool_name, tool_input):
    if tool_name == "get_current_datetime":
        return get_current_datetime(**tool_input)
    elif tool_name == "add_duration_to_datetime":
        return add_duration_to_datetime(**tool_input)
    elif tool_name == "set_reminder":
        return set_reminder(**tool_input)
```

Essa função é chamada de **tool router**, porque recebe o nome da ferramenta solicitada por Claude e direciona para a função correta no código.

---

### Padrão simples para adicionar novas ferramentas

A aula mostra um padrão reutilizável:

1. Criar a função da ferramenta.
2. Definir o schema da ferramenta.
3. Adicionar o schema à lista `tools`.
4. Adicionar um caso no `run_tool`.

Esse padrão é modular porque cada ferramenta nova entra no sistema sem exigir reescrever o loop principal da conversa.

---

### Testando múltiplas ferramentas

O exemplo de teste é:

```text
Set a reminder for my doctors appointment. Its 177 days after Jan 1st, 2050.
```

Essa solicitação exige pelo menos duas ferramentas:

1. `add_duration_to_datetime` para calcular a data.
2. `set_reminder` para criar o lembrete.

Claude primeiro entende que precisa calcular a data do compromisso. Depois, ao receber o resultado, usa esse resultado para definir o lembrete.

No exemplo, Claude calcula que 177 dias após 1º de janeiro de 2050 resulta em:

```text
June 27, 2050
```

Depois chama a ferramenta `set_reminder` com algo como:

```python
{
    "content": "Doctor's appointment",
    "timestamp": "2050-06-27T00:00:00"
}
```

---

### Fluxo da mensagem com múltiplas tools

Ao examinar o histórico da conversa, é possível ver uma sequência como:

1. Mensagem do usuário com o pedido.
2. Mensagem do assistant com texto e `ToolUseBlock`.
3. Mensagem do usuário com `ToolResultBlock`.
4. Nova mensagem do assistant com outro `ToolUseBlock`.
5. Nova mensagem do usuário com outro `ToolResultBlock`.
6. Resposta final do assistant.

Isso mostra que Claude pode misturar:

* Blocos de texto.
* Blocos `tool_use`.
* Resultados de ferramentas.
* Chamadas subsequentes baseadas nos resultados anteriores.

---

### Exemplo demonstrado

Pedido do usuário:

```text
Set a reminder for my doctors appointment. Its 177 days after Jan 1st, 2050.
```

Claude responde inicialmente algo como:

```text
I'll set a reminder for your doctor's appointment that's 177 days after January 1st, 2050. Let me calculate that date first.
```

Depois chama:

```text
add_duration_to_datetime
```

Com input parecido com:

```python
{
    "datetime_str": "2050-01-01",
    "duration": 177,
    "unit": "days"
}
```

A aplicação retorna:

```text
Monday, June 27, 2050 12:00:00 AM
```

Então Claude chama:

```text
set_reminder
```

Com input parecido com:

```python
{
    "content": "Doctor's appointment",
    "timestamp": "2050-06-27T00:00:00"
}
```

Por fim, Claude responde que o lembrete foi criado com sucesso.

Esse exemplo demonstra o conceito de **encadeamento de tools**, em que o resultado de uma ferramenta vira informação de entrada para a próxima etapa do raciocínio.

---

## 3. Pontos importantes para certificação

* Depois de implementar a infraestrutura básica de tools, adicionar novas ferramentas é simples.
* Para adicionar uma ferramenta, normalmente são necessários:

  * função de implementação;
  * schema da ferramenta;
  * inclusão do schema na lista `tools`;
  * caso correspondente no roteador `run_tool`.
* A lista `tools` pode conter múltiplos schemas.
* Claude escolhe qual tool usar com base na necessidade da tarefa.
* O roteador `run_tool` mapeia nomes de ferramentas para funções reais.
* O nome usado no schema deve corresponder ao nome verificado no roteador.
* Claude pode chamar várias tools em sequência.
* Uma tool pode produzir informação necessária para outra tool.
* No exemplo, Claude primeiro calcula a data com `add_duration_to_datetime` e depois cria o lembrete com `set_reminder`.
* Usar ferramenta para cálculo de datas é mais confiável do que depender apenas do modelo.
* O histórico de mensagens mostra blocos `TextBlock`, `ToolUseBlock` e `ToolResultBlock`.
* Uma mensagem do assistant pode conter texto explicativo e tool use ao mesmo tempo.
* O resultado da tool deve voltar como `tool_result`.
* O loop de conversa continua até Claude parar de solicitar tools.
* O padrão modular permite expandir as capacidades do assistente sem reestruturar o código principal.

---

## 4. Termos-chave

### Multiple tools

Uso de mais de uma ferramenta disponível para Claude em uma mesma aplicação ou conversa.

### Tool schema

Descrição estruturada da ferramenta, informando nome, descrição e parâmetros que Claude pode usar.

### `tools` list

Lista enviada na chamada da API contendo os schemas das ferramentas disponíveis.

### `run_tool`

Função roteadora que recebe o nome da ferramenta e executa a implementação correta.

### Tool router

Mecanismo que mapeia uma chamada de tool para uma função real no código.

### `get_current_datetime`

Ferramenta usada para obter a data e hora atual.

### `add_duration_to_datetime`

Ferramenta usada para somar uma duração a uma data ou horário.

### `set_reminder`

Ferramenta usada para criar um lembrete.

### `ToolUseBlock`

Bloco retornado por Claude quando ele solicita a execução de uma ferramenta.

### `ToolResultBlock`

Bloco enviado pela aplicação para Claude com o resultado da ferramenta executada.

### Encadeamento de tools

Quando Claude usa o resultado de uma ferramenta para decidir ou executar a próxima ferramenta.

### Modularidade

Organização do código que permite adicionar novas ferramentas sem alterar toda a arquitetura.

---

## 5. Boas práticas

* Use tools para tarefas que exigem precisão, como datas, horários, cálculos e ações externas.
* Inclua todos os schemas necessários na lista `tools`.
* Mantenha o loop principal genérico.
* Coloque a lógica específica de cada ferramenta no `run_tool`.
* Garanta que o nome da ferramenta no schema seja igual ao nome usado no roteador.
* Adicione uma nova ferramenta seguindo sempre o mesmo padrão:

  * função;
  * schema;
  * inclusão em `tools`;
  * caso no `run_tool`.
* Preserve o histórico completo da conversa.
* Permita que Claude faça várias chamadas em sequência.
* Retorne `ToolResultBlock` para cada `ToolUseBlock`.
* Use tratamento de erro para cada ferramenta.
* Valide inputs antes de executar ações reais, especialmente lembretes, escrita em banco, envio de mensagens ou mudanças externas.
* Teste com solicitações que forcem o uso de múltiplas tools.

---

## 6. Limitações, riscos e cuidados

* Claude pode errar cálculos de data se não usar uma ferramenta especializada.
* Se uma tool estiver no schema, mas não estiver no roteador, Claude pode solicitá-la e a aplicação não saber executá-la.
* Se o nome da ferramenta no schema não bater com o nome no `run_tool`, a execução pode falhar.
* Se o histórico não for preservado, Claude pode perder o resultado da ferramenta anterior.
* Se o `tool_result` não for enviado corretamente, Claude não consegue continuar o fluxo.
* Ferramentas que executam ações reais, como `set_reminder`, devem ter validações e controles.
* O sistema deve tratar erros de ferramentas e retornar `is_error: True` quando necessário.
* Deve haver limite de iterações para evitar loops infinitos.
* Múltiplas tools aumentam a capacidade do assistente, mas também aumentam a responsabilidade da aplicação em rotear, validar e auditar chamadas.
* O modelo decide qual tool usar, mas a aplicação deve controlar o que realmente será executado.

---

## 7. Resumo para revisão rápida

* A aula mostra como usar várias tools em uma aplicação com Claude.
* O exemplo usa:

  * `get_current_datetime`;
  * `add_duration_to_datetime`;
  * `set_reminder`.
* Para adicionar tools, inclua seus schemas na lista `tools`.
* Também atualize o roteador `run_tool`.
* Claude pode usar uma tool para obter informação e outra para agir.
* Exemplo: calcular data futura e depois criar lembrete.
* O histórico contém mensagens com `TextBlock`, `ToolUseBlock` e `ToolResultBlock`.
* O padrão para adicionar tools é:

  * criar função;
  * criar schema;
  * adicionar schema à lista;
  * adicionar caso no roteador.
* Esse padrão é modular e escalável.
* Tools são úteis para precisão e ações externas.
* Validação, tratamento de erros e preservação do histórico são essenciais.

---

## 8. Perguntas simuladas

### Pergunta 1

Qual é o objetivo principal da aula sobre uso de múltiplas tools?

A. Mostrar como substituir completamente o histórico de conversa.
B. Mostrar como adicionar várias ferramentas a Claude usando uma infraestrutura de tools já existente.
C. Ensinar Claude a executar funções sem schemas.
D. Remover a necessidade de `ToolResultBlock`.

**Resposta correta: B**

**Explicação:** A aula mostra que, depois da infraestrutura básica, adicionar múltiplas tools envolve adicionar schemas e atualizar o roteador.

---

### Pergunta 2

Quais são as três ferramentas principais do exemplo?

A. `send_email`, `search_web`, `delete_file`
B. `get_current_datetime`, `add_duration_to_datetime`, `set_reminder`
C. `translate_text`, `summarize_pdf`, `create_image`
D. `read_file`, `write_file`, `run_linter`

**Resposta correta: B**

**Explicação:** O sistema de lembretes usa uma ferramenta para data atual, uma para somar duração e outra para criar lembrete.

---

### Pergunta 3

Por que a ferramenta `add_duration_to_datetime` é útil?

A. Porque Claude não consegue gerar texto.
B. Porque cálculos de data e hora exigem precisão e Claude pode não ser perfeito nisso.
C. Porque ela substitui o schema de tools.
D. Porque ela apaga lembretes antigos.

**Resposta correta: B**

**Explicação:** A aula destaca que Claude não é perfeito com adição de datas, então uma ferramenta especializada aumenta a confiabilidade.

---

### Pergunta 4

Como várias tools são disponibilizadas para Claude na chamada?

A. Colocando seus schemas na lista `tools`.
B. Escrevendo os nomes das tools em uma string única.
C. Adicionando as funções diretamente no prompt do usuário.
D. Salvando as tools apenas no histórico.

**Resposta correta: A**

**Explicação:** A chamada deve incluir algo como `tools=[schema1, schema2, schema3]`.

---

### Pergunta 5

Qual código representa a ideia correta de adicionar múltiplas tools?

A.

```python
response = chat(messages, tools=[
    get_current_datetime_schema,
    add_duration_to_datetime_schema,
    set_reminder_schema
])
```

B.

```python
response = chat(messages, tools="all")
```

C.

```python
response = chat(messages, tool="set_reminder")
```

D.

```python
response = chat(messages)
```

**Resposta correta: A**

**Explicação:** A lista `tools` deve conter os schemas das ferramentas disponíveis.

---

### Pergunta 6

Para que serve a função `run_tool`?

A. Para transformar a resposta final em texto simples.
B. Para mapear o nome da ferramenta solicitada por Claude para a função real da aplicação.
C. Para remover blocos `tool_use`.
D. Para criar automaticamente todos os schemas.

**Resposta correta: B**

**Explicação:** `run_tool` funciona como roteador de ferramentas.

---

### Pergunta 7

Qual alteração é necessária em `run_tool` ao adicionar uma nova ferramenta?

A. Adicionar um novo caso que reconheça o nome da tool e chame a função correspondente.
B. Remover todos os schemas anteriores.
C. Alterar o role da mensagem para `assistant`.
D. Ignorar o input enviado por Claude.

**Resposta correta: A**

**Explicação:** Cada nova ferramenta precisa ser mapeada no roteador para que possa ser executada.

---

### Pergunta 8

No exemplo do lembrete, por que Claude chama primeiro `add_duration_to_datetime`?

A. Para calcular a data que fica 177 dias após 1º de janeiro de 2050.
B. Para enviar o lembrete diretamente.
C. Para apagar o histórico.
D. Para verificar se o usuário é médico.

**Resposta correta: A**

**Explicação:** Antes de criar o lembrete, Claude precisa descobrir a data correta do compromisso.

---

### Pergunta 9

Depois de calcular a data do compromisso, qual ferramenta Claude deve usar?

A. `get_current_datetime`
B. `set_reminder`
C. `json.dumps`
D. `text_from_message`

**Resposta correta: B**

**Explicação:** Após saber a data, Claude pode chamar `set_reminder` para criar o lembrete.

---

### Pergunta 10

O que o histórico da conversa demonstra quando múltiplas tools são usadas?

A. Que apenas mensagens de texto são mantidas.
B. Que a conversa pode conter mensagens de usuário, mensagens do assistant, `ToolUseBlock` e `ToolResultBlock`.
C. Que Claude executa ferramentas sem a aplicação.
D. Que schemas não são necessários.

**Resposta correta: B**

**Explicação:** O histórico completo registra o pedido, as chamadas de tools, os resultados e as respostas intermediárias.

---

### Pergunta 11

Qual é o padrão recomendado para adicionar uma nova tool?

A. Criar função, definir schema, adicionar schema à lista `tools` e adicionar caso no `run_tool`.
B. Apenas escrever o nome da tool no prompt.
C. Criar um `ToolResultBlock` antes de Claude pedir a tool.
D. Remover o loop de conversa.

**Resposta correta: A**

**Explicação:** Esse é o padrão modular apresentado na aula.

---

### Pergunta 12

Qual é uma vantagem do padrão modular de tools?

A. Ele impede Claude de usar múltiplas ferramentas.
B. Ele permite expandir capacidades sem reestruturar o código principal.
C. Ele elimina a necessidade de validação.
D. Ele faz o modelo executar código internamente.

**Resposta correta: B**

**Explicação:** Novas ferramentas podem ser adicionadas com mudanças localizadas na lista de schemas e no roteador.

---

### Pergunta 13

Qual risco existe se uma ferramenta for adicionada à lista `tools`, mas não ao `run_tool`?

A. Claude pode solicitar a ferramenta, mas a aplicação não saberá executá-la.
B. Claude nunca verá o schema.
C. O histórico será automaticamente apagado.
D. O `ToolResultBlock` será criado sozinho.

**Resposta correta: A**

**Explicação:** O schema informa a Claude que a ferramenta existe, mas o roteador precisa saber como executá-la.

---

### Pergunta 14

Qual destas afirmações melhor descreve o encadeamento de tools?

A. Uma ferramenta sempre substitui todas as outras.
B. Claude pode usar o resultado de uma ferramenta para decidir ou alimentar a próxima chamada de ferramenta.
C. Tools só podem ser usadas uma vez por conversa.
D. Tools não podem retornar resultados.

**Resposta correta: B**

**Explicação:** No exemplo, Claude calcula a data e depois usa essa data para criar o lembrete.
