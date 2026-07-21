# Fine-Grained Tool Calling

## 1. Ideia central

A aula explica **Fine-Grained Tool Calling**, um modo de uso de ferramentas com **streaming** na API da Anthropic.

Quando combinamos **tool use** com **streaming**, Claude pode enviar atualizações em tempo real enquanto está gerando os argumentos de uma ferramenta. Isso melhora a experiência do usuário, porque a aplicação pode mostrar progresso antes da resposta completa estar pronta.

Porém, existe uma diferença importante:

* No streaming padrão, a API **segura e valida partes do JSON** antes de enviá-las.
* No fine-grained tool calling, a API envia os chunks mais rapidamente, mas **desativa a validação de JSON**.

A ideia central é: **fine-grained tool calling dá mais velocidade e granularidade, mas transfere para sua aplicação a responsabilidade de lidar com JSON inválido ou incompleto.**

---

## 2. Principais conceitos

### Streaming com tools

Quando o streaming está ativado, Claude não envia a resposta inteira de uma vez. Ele envia eventos em partes.

Em respostas comuns de texto, você já pode receber eventos como:

```text
ContentBlockDelta
```

Esses eventos carregam pedaços do texto gerado.

Com tools, além de texto, Claude também pode gerar argumentos de ferramenta em partes. Para isso, aparece um novo tipo de evento:

```text
InputJsonEvent
```

Esse evento é usado para transmitir pedaços do JSON que representa os argumentos da ferramenta.

---

### Tipos de eventos no streaming

A imagem mostra uma sequência típica de eventos:

```text
MessageStart
ContentBlockStart
ContentBlockDelta
ContentBlockDelta
ContentBlockDelta
ContentBlockStop
MessageDelta
MessageStop
```

### `MessageStart`

Indica que uma nova mensagem começou a ser enviada.

### `ContentBlockStart`

Indica o início de um novo bloco de conteúdo.

Esse bloco pode conter:

* Texto.
* Uso de ferramenta.
* Outro tipo de conteúdo.

### `ContentBlockDelta`

Contém pedaços incrementais do conteúdo gerado.

Na prática, esses eventos carregam o texto ou partes dos argumentos sendo gerados.

### `ContentBlockStop`

Indica que o bloco atual foi concluído.

### `MessageDelta`

Indica atualização relacionada ao estado da mensagem como um todo.

### `MessageStop`

Indica o fim das informações da mensagem atual.

---

### `InputJsonEvent`

Quando Claude está gerando argumentos para uma tool em streaming, a aplicação precisa lidar com eventos do tipo:

```text
InputJsonEvent
```

Cada `InputJsonEvent` contém duas propriedades importantes:

```text
partial_json
snapshot
```

### `partial_json`

É apenas o pedaço novo de JSON recebido naquele evento.

Exemplo conceitual:

```json
{"title"
```

Depois:

```json
": "Vector
```

Depois:

```json
 Storage"}
```

### `snapshot`

É o JSON acumulado até aquele momento, juntando todos os chunks recebidos até agora.

Exemplo:

```json
{"title": "Vector Storage"}
```

Ou seja:

* `partial_json` = incremento atual.
* `snapshot` = estado acumulado até agora.

---

### Como tratar eventos `input_json`

A aula mostra um padrão como:

```python
for chunk in stream:
    if chunk.type == "input_json":
        print(chunk.partial_json)
        current_args = chunk.snapshot
```

Esse código permite acompanhar os argumentos da ferramenta conforme Claude os gera.

Isso é útil quando você quer mostrar progresso em tempo real ou começar a processar dados antes de tudo estar completo.

---

### Como funciona a validação padrão de JSON

No comportamento padrão, a API da Anthropic **não envia imediatamente cada pedaço do JSON** assim que Claude gera.

Ela faz buffering.

Isso significa que a API segura alguns chunks até formar um par completo de chave-valor de nível superior.

Exemplo de JSON esperado:

```json
{
  "abstract": "This paper presents a novel...",
  "meta": {
    "word_count": 847,
    "review": "This paper introduces QuanNet..."
  }
}
```

A API pode esperar até terminar todo o valor de:

```json
"abstract": "This paper presents a novel..."
```

Depois valida esse par chave-valor contra o schema. Só então envia os chunks relacionados.

Depois repete o processo com:

```json
"meta": {...}
```

Isso explica por que, mesmo com streaming ativado, às vezes há uma pausa seguida por uma rajada de conteúdo.

---

### Por que há atrasos no streaming padrão

O atraso acontece porque a API está protegendo a aplicação.

Ela espera até ter um par de chave-valor completo e válido antes de enviar.

Exemplo:

```json
{"abstract": "This paper presents..."}
```

A API pergunta, internamente:

```text
Esse par chave-valor de nível superior corresponde ao schema save_article_schema?
```

Se sim, ela envia os chunks.

Esse comportamento ajuda a garantir que a aplicação receba JSON mais seguro e coerente com o schema.

---

### Fine-Grained Tool Calling

Fine-grained tool calling muda esse comportamento.

Em vez de esperar um par completo de chave-valor de nível superior, a API envia grupos menores de chunks assim que possível.

A grande consequência:

```text
A validação de JSON no lado da API é desativada.
```

Isso significa que:

* A aplicação recebe chunks mais rapidamente.
* Há menos atraso entre pedaços de JSON.
* A experiência de streaming parece mais imediata.
* Mas a aplicação pode receber JSON incompleto ou inválido.

---

### Como ativar fine-grained tool calling

A aula mostra a ativação com o parâmetro:

```python
run_conversation(
    messages,
    tools=[save_article_schema],
    fine_grained=True
)
```

O ponto importante é o argumento:

```python
fine_grained=True
```

Esse modo faz a API enviar os chunks de argumentos da ferramenta de forma mais granular.

---

### Validação desativada

Este é o ponto mais importante da aula:

```text
Critical: JSON validation is disabled!
```

Com fine-grained tool calling, sua aplicação precisa estar preparada para receber dados como:

```json
"word_count": undefined
```

Esse valor não é JSON válido.

No modo padrão, a API poderia segurar, corrigir, validar ou impedir esse tipo de estrutura antes de entregar para sua aplicação. No modo fine-grained, essa proteção deixa de existir.

---

### Tratamento de JSON inválido

A aplicação precisa tratar erros de parsing.

Exemplo:

```python
try:
    parsed_args = json.loads(chunk.snapshot)
except json.JSONDecodeError:
    print("Received invalid JSON, continuing...")
```

Esse padrão evita que a aplicação quebre quando o JSON ainda estiver incompleto ou inválido.

Em outras palavras, no modo fine-grained, receber erro de parsing pode ser normal durante o streaming.

---

### Quando usar Fine-Grained Tool Calling

Esse modo faz sentido quando você precisa de maior responsividade.

Use quando:

* Você quer mostrar ao usuário o progresso dos argumentos gerados.
* Você precisa processar partes da ferramenta o mais cedo possível.
* Os atrasos do buffering padrão prejudicam a experiência.
* Sua aplicação consegue lidar bem com JSON inválido ou incompleto.
* Você tem tratamento robusto de erros.

Para a maioria das aplicações, o comportamento padrão com validação é suficiente e mais seguro.

---

## 3. Pontos importantes para certificação

* Tool use pode ser combinado com streaming.
* Com streaming, Claude envia eventos incrementais.
* Eventos como `ContentBlockDelta` carregam partes do conteúdo gerado.
* Para argumentos de ferramentas em streaming, existe o evento `InputJsonEvent`.
* `InputJsonEvent` contém:

  * `partial_json`
  * `snapshot`
* `partial_json` é o chunk atual.
* `snapshot` é o JSON acumulado até aquele ponto.
* No streaming padrão, a API faz buffering e validação.
* A API espera pares completos de chave-valor de nível superior antes de enviar chunks.
* Esse buffering pode gerar atrasos seguidos de rajadas de texto.
* Fine-grained tool calling envia chunks mais rapidamente.
* Fine-grained tool calling desativa a validação de JSON no lado da API.
* Com fine-grained habilitado, a aplicação precisa lidar com JSON inválido.
* O parâmetro usado no exemplo é:

```python
fine_grained=True
```

* Se o JSON estiver incompleto ou inválido, `json.loads` pode gerar `JSONDecodeError`.
* O código deve tratar esse erro de forma segura.
* Fine-grained tool calling é útil para maior responsividade, mas exige mais responsabilidade da aplicação.
* Para a maioria dos casos, o modo padrão com validação é adequado.

---

## 4. Termos-chave

### Streaming

Modo em que a resposta é enviada em partes, conforme é gerada, em vez de esperar a resposta completa.

### Tool use

Capacidade de Claude solicitar a execução de ferramentas externas por meio de blocos estruturados.

### Fine-Grained Tool Calling

Modo mais granular de streaming de argumentos de tools, que envia chunks mais rapidamente, mas sem validação de JSON pela API.

### `InputJsonEvent`

Evento emitido durante streaming de argumentos de tools, contendo pedaços de JSON.

### `partial_json`

Pedaço atual de JSON recebido no evento.

### `snapshot`

JSON acumulado até o momento, formado pelos chunks já recebidos.

### `ContentBlockDelta`

Evento de streaming que contém uma parte incremental do conteúdo do bloco atual.

### `ContentBlockStart`

Evento que indica o início de um bloco de conteúdo.

### `ContentBlockStop`

Evento que indica que o bloco atual terminou.

### `MessageStart`

Evento que indica o início de uma nova mensagem.

### `MessageStop`

Evento que indica o fim da mensagem atual.

### Buffering

Processo em que a API segura chunks temporariamente antes de enviá-los.

### JSON validation

Validação para verificar se os argumentos gerados estão de acordo com JSON válido e com o schema da ferramenta.

### Top-level key-value pair

Par chave-valor no nível principal do JSON, como `"abstract": "..."`.

### `JSONDecodeError`

Erro gerado quando a aplicação tenta interpretar uma string que não é JSON válido.

---

## 5. Boas práticas

* Use streaming para melhorar a percepção de velocidade da aplicação.
* Ao usar tools com streaming, trate eventos `input_json`.
* Use `partial_json` para exibir progresso incremental.
* Use `snapshot` para acompanhar o estado acumulado dos argumentos.
* No modo padrão, aproveite a validação da API quando segurança e confiabilidade forem prioridade.
* Use fine-grained tool calling apenas quando a responsividade for realmente importante.
* Ao ativar `fine_grained=True`, implemente tratamento robusto de JSON inválido.
* Envolva `json.loads` em `try/except`.
* Continue processando o stream mesmo se um snapshot intermediário ainda não for JSON válido.
* Valide os argumentos no seu próprio código antes de executar a ferramenta.
* Não execute ações sensíveis com argumentos parciais sem validação.
* Para ferramentas que escrevem arquivos, salvam dados ou executam ações externas, valide tudo antes da execução final.

---

## 6. Limitações, riscos e cuidados

* Fine-grained tool calling desativa a validação de JSON no lado da API.
* A aplicação pode receber JSON incompleto.
* A aplicação pode receber JSON inválido.
* A aplicação pode receber valores incompatíveis com o schema esperado.
* Sem tratamento de erro, `json.loads` pode quebrar a aplicação.
* Argumentos parciais não devem ser usados automaticamente em ações sensíveis.
* O modo padrão pode parecer menos responsivo por causa do buffering.
* O modo fine-grained é mais rápido, mas menos protegido.
* O desenvolvedor precisa decidir entre:

  * mais segurança e validação;
  * mais velocidade e controle.
* Para a maioria das aplicações, o modo padrão com validação é suficiente.
* Fine-grained deve ser usado quando a aplicação está preparada para lidar com entradas imperfeitas.

---

## 7. Resumo para revisão rápida

* Streaming envia respostas em chunks.
* Com tools, argumentos também podem vir em streaming.
* `InputJsonEvent` é usado para chunks de argumentos JSON.
* `partial_json` é o pedaço atual.
* `snapshot` é o JSON acumulado.
* No modo padrão, a API faz buffering.
* A API espera pares completos de chave-valor de nível superior.
* Depois valida contra o schema e envia os chunks.
* Isso pode causar pausas e rajadas.
* Fine-grained tool calling envia chunks mais cedo.
* Fine-grained desativa validação de JSON pela API.
* Com fine-grained, sua aplicação deve tratar JSON inválido.
* Use `try/except json.JSONDecodeError`.
* Use fine-grained quando precisar de máxima responsividade.
* Prefira o padrão quando segurança e validação forem mais importantes.

---

## 8. Perguntas simuladas

### Pergunta 1

O que acontece quando combinamos tool use com streaming?

A. Claude deixa de usar ferramentas.
B. Claude pode enviar atualizações em tempo real enquanto gera argumentos de ferramentas.
C. A API retorna apenas texto simples.
D. O histórico da conversa é apagado automaticamente.

**Resposta correta: B**

**Explicação:** Com streaming e tools, a aplicação pode receber eventos incrementais relacionados ao texto e aos argumentos da ferramenta.

---

### Pergunta 2

Qual evento é usado para transmitir partes dos argumentos JSON de uma ferramenta?

A. `MessageStop`
B. `InputJsonEvent`
C. `SystemPromptEvent`
D. `FinalAnswerEvent`

**Resposta correta: B**

**Explicação:** `InputJsonEvent` contém partes dos argumentos JSON gerados para uma tool.

---

### Pergunta 3

Quais são as duas propriedades principais de um `InputJsonEvent`?

A. `role` e `model`
B. `partial_json` e `snapshot`
C. `temperature` e `max_tokens`
D. `tool_name` e `schema_name`

**Resposta correta: B**

**Explicação:** `partial_json` contém o chunk atual, enquanto `snapshot` contém o JSON acumulado até o momento.

---

### Pergunta 4

O que representa `partial_json`?

A. O schema completo da ferramenta.
B. O pedaço atual de JSON recebido no evento.
C. A resposta final do usuário.
D. O histórico completo da conversa.

**Resposta correta: B**

**Explicação:** `partial_json` é apenas o fragmento novo recebido naquele evento.

---

### Pergunta 5

O que representa `snapshot`?

A. O JSON acumulado até aquele ponto do streaming.
B. Apenas o primeiro chunk recebido.
C. O nome da ferramenta.
D. O erro final da API.

**Resposta correta: A**

**Explicação:** `snapshot` mostra o estado acumulado dos argumentos gerados até o momento.

---

### Pergunta 6

No comportamento padrão de streaming com tools, por que podem ocorrer atrasos antes de receber chunks?

A. Porque Claude sempre espera o usuário confirmar manualmente.
B. Porque a API faz buffering e valida pares completos de chave-valor antes de enviar.
C. Porque tools não funcionam com streaming.
D. Porque `partial_json` é sempre vazio.

**Resposta correta: B**

**Explicação:** A API segura chunks até formar e validar pares de chave-valor de nível superior.

---

### Pergunta 7

O que é um top-level key-value pair?

A. Um par chave-valor no nível principal do JSON.
B. Uma mensagem do sistema.
C. Um erro de parsing.
D. Um tipo de modelo Claude.

**Resposta correta: A**

**Explicação:** Exemplo: `"abstract": "This paper presents..."` no nível principal do objeto JSON.

---

### Pergunta 8

O que o fine-grained tool calling faz de principal?

A. Desativa o uso de ferramentas.
B. Envia chunks mais rapidamente ao desativar a validação de JSON no lado da API.
C. Remove a necessidade de schemas.
D. Impede o streaming de argumentos.

**Resposta correta: B**

**Explicação:** Fine-grained tool calling reduz o buffering e envia chunks mais cedo, mas sem validação de JSON pela API.

---

### Pergunta 9

Qual é o cuidado crítico ao usar fine-grained tool calling?

A. Não usar mensagens de usuário.
B. A aplicação precisa lidar com JSON inválido ou incompleto.
C. Remover todos os schemas de tools.
D. Nunca usar `try/except`.

**Resposta correta: B**

**Explicação:** Como a validação de JSON é desativada, a aplicação assume a responsabilidade de validar e tratar erros.

---

### Pergunta 10

Qual parâmetro aparece no exemplo para habilitar fine-grained tool calling?

A. `stream=False`
B. `fine_grained=True`
C. `validate_json=True`
D. `tools=None`

**Resposta correta: B**

**Explicação:** A aula mostra `fine_grained=True` como forma de ativar esse modo.

---

### Pergunta 11

Qual erro pode ocorrer ao tentar interpretar um snapshot incompleto ou inválido como JSON?

A. `JSONDecodeError`
B. `ToolUseStop`
C. `MessageStartError`
D. `SchemaApproved`

**Resposta correta: A**

**Explicação:** `json.loads` lança `JSONDecodeError` quando a string não é JSON válido.

---

### Pergunta 12

Qual padrão de código é recomendado para lidar com JSON inválido?

A.

```python
try:
    parsed_args = json.loads(chunk.snapshot)
except json.JSONDecodeError:
    print("Received invalid JSON, continuing...")
```

B.

```python
parsed_args = json.loads(chunk.snapshot)
raise Exception("Always fail")
```

C.

```python
delete_all_chunks()
```

D.

```python
chunk.snapshot = None
```

**Resposta correta: A**

**Explicação:** O `try/except` permite que a aplicação continue funcionando mesmo quando o JSON ainda não está válido.

---

### Pergunta 13

Quando faz sentido usar fine-grained tool calling?

A. Quando você precisa de maior responsividade e está preparado para tratar JSON inválido.
B. Quando você quer máxima validação pela API.
C. Quando não há necessidade de streaming.
D. Quando a aplicação não consegue tratar erros.

**Resposta correta: A**

**Explicação:** Fine-grained é útil para atualizações rápidas, mas exige tratamento robusto no código.

---

### Pergunta 14

Para a maioria das aplicações, qual modo tende a ser suficiente?

A. O comportamento padrão com buffering e validação.
B. Sempre fine-grained sem validação.
C. Nenhum modo com tools.
D. Apenas respostas sem streaming.

**Resposta correta: A**

**Explicação:** O modo padrão é mais seguro e adequado para a maioria dos casos porque preserva a validação da API.

---

### Pergunta 15

Qual é a principal troca envolvida no fine-grained tool calling?

A. Menos velocidade em troca de mais contexto.
B. Mais velocidade e granularidade em troca de menos validação automática.
C. Menos controle da aplicação em troca de mais segurança.
D. Mais validação em troca de menos chunks.

**Resposta correta: B**

**Explicação:** Fine-grained entrega chunks mais cedo, mas a aplicação passa a ser responsável pela validação do JSON.
