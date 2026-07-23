# Pensamento expandido

```
notebooks\26-extended_thinking.ipynb
```

## 1. Ideia central

O conteúdo apresenta o recurso **Extended Thinking** do Claude.

A ideia central é que, para tarefas complexas, Claude pode receber um orçamento de tokens para “pensar” antes de gerar a resposta final. Com isso, a resposta da API pode vir em blocos estruturados: um bloco de **thinking** e um bloco de **text** com a resposta final.

Em resumo:

**Extended Thinking dá ao Claude mais espaço para raciocinar em tarefas difíceis, mas aumenta custo, latência e complexidade de implementação.**

---

## 2. Principais conceitos

### O que é Extended Thinking

**Extended Thinking** é um recurso avançado de raciocínio.

Ele permite que Claude faça uma etapa interna de análise antes de produzir a resposta final.

Sem thinking, a resposta tende a conter apenas um bloco de texto:

```text
TextBlock:
"Recursion is a powerful programming technique..."
```

Com thinking ativado, a resposta pode conter:

```text
ThinkingBlock:
"I should cover what recursion is, how it works..."

TextBlock:
"Recursion is a powerful programming technique..."
```

Ou seja, a mensagem do assistente pode ter mais de um bloco de conteúdo.

---

### Estrutura da resposta com thinking

Quando o servidor envia uma pergunta, por exemplo:

```text
Write a short guide on recursion
```

Claude pode responder com uma estrutura parecida com:

```json
{
  "content": [
    {
      "type": "thinking",
      "thinking": "The user is asking me to...",
      "signature": "..."
    },
    {
      "type": "text",
      "text": "A Guide to Recursion..."
    }
  ]
}
```

O ponto importante é que a resposta final continua vindo no bloco `text`, enquanto o bloco `thinking` representa o raciocínio gerado pelo modelo.

---

### Benefícios do Extended Thinking

O recurso pode ajudar em tarefas que exigem:

* raciocínio complexo;
* planejamento;
* resolução de problemas difíceis;
* análise com múltiplas etapas;
* maior precisão;
* mais transparência sobre como a resposta foi construída.

Exemplos de bons casos de uso:

* problemas de programação complexos;
* análise jurídica ou financeira com cuidado;
* planejamento de arquitetura;
* debugging difícil;
* tarefas matemáticas ou lógicas;
* decisões com muitos critérios.

---

### Trade-offs

Extended Thinking não deve ser ativado automaticamente para tudo.

Ele traz custos importantes:

| Trade-off                | Explicação                                   |
| ------------------------ | -------------------------------------------- |
| Maior custo              | Os tokens de thinking também são cobrados    |
| Maior latência           | Claude leva mais tempo para raciocinar       |
| Resposta mais complexa   | O código precisa lidar com blocos adicionais |
| Compatibilidade limitada | Não funciona com todos os recursos           |

O conteúdo destaca que Extended Thinking não é compatível com alguns recursos, como **message pre-filling** e **temperature**, de acordo com a documentação citada no curso.

---

### Quando usar Extended Thinking

A recomendação principal é:

**não comece usando Extended Thinking.**

Primeiro:

1. escreva um prompt claro;
2. aplique técnicas de prompt engineering;
3. avalie o desempenho com evals;
4. otimize o prompt;
5. só então considere Extended Thinking se a qualidade ainda não for suficiente.

Ou seja, Extended Thinking é uma ferramenta para casos em que o prompt bem projetado ainda não atinge o nível necessário de acurácia.

---

### Signature

O conteúdo apresenta o campo **signature**.

A signature é um token criptográfico associado ao texto do bloco de thinking.

Ela serve para verificar que:

* aquele texto foi realmente gerado por Claude;
* o conteúdo não foi alterado depois;
* o bloco de thinking não foi manipulado pelo desenvolvedor.

Exemplo conceitual:

```json
{
  "type": "thinking",
  "thinking": "The user is asking me to...",
  "signature": "EqoBCkgIAhABGAIiQDbvHj8yA..."
}
```

Esse mecanismo ajuda a proteger a integridade do conteúdo de thinking.

---

### Por que a signature importa

A signature existe por segurança.

Se desenvolvedores pudessem modificar livremente o bloco de thinking e reenviá-lo como se fosse original, poderiam tentar influenciar o comportamento do modelo de forma insegura.

A assinatura criptográfica ajuda a garantir que o conteúdo de thinking passado em conversas futuras corresponde ao que Claude realmente produziu.

---

### Redacted thinking

Às vezes, em vez de receber o thinking legível, a aplicação pode receber um bloco de **redacted thinking**.

Isso acontece quando sistemas internos de segurança sinalizam o conteúdo do raciocínio.

Nesse caso:

* o thinking não aparece em texto claro;
* o conteúdo fica criptografado ou redigido;
* ainda pode ser passado de volta para Claude em chamadas futuras;
* o contexto da conversa é preservado sem revelar o raciocínio sensível.

Isso evita que a aplicação quebre quando o conteúdo de thinking não puder ser exibido.

---

### Implementação no código

O conteúdo mostra uma função `chat()` atualizada para aceitar parâmetros de thinking:

```python
def chat(
    messages,
    system=None,
    temperature=1.0,
    stop_sequences=[],
    tools=None,
    thinking=False,
    thinking_budget=1024
):
```

Os novos parâmetros são:

```python
thinking=False
thinking_budget=1024
```

* `thinking`: ativa ou desativa o recurso;
* `thinking_budget`: define o número máximo de tokens que Claude pode usar para raciocinar.

---

### Configuração dos parâmetros da API

Quando `thinking=True`, adiciona-se a configuração:

```python
if thinking:
    params["thinking"] = {
        "type": "enabled",
        "budget": thinking_budget
    }
```

Depois, a função pode ser chamada assim:

```python
chat(messages, thinking=True)
```

---

### Thinking budget

O **thinking budget** define o limite de tokens disponíveis para o raciocínio.

Segundo o conteúdo:

* o valor mínimo é `1024`;
* `max_tokens` precisa ser maior que `thinking_budget`.

Exemplo:

```python
params = {
    "model": model,
    "max_tokens": 2000,
    "messages": messages,
    "thinking": {
        "type": "enabled",
        "budget": 1024
    }
}
```

Se `max_tokens` for menor ou igual ao orçamento de thinking, a chamada pode falhar ou não ter espaço suficiente para a resposta final.

---

## 3. Pontos importantes para certificação

* Extended Thinking é um recurso avançado de raciocínio do Claude.
* Ele permite que Claude use tokens para raciocinar antes da resposta final.
* A resposta pode conter um `thinking block` e um `text block`.
* O `text block` contém a resposta final para o usuário.
* O recurso pode melhorar tarefas complexas, mas aumenta custo e latência.
* Tokens de thinking também entram no custo.
* Extended Thinking deve ser usado com base em avaliação, não como padrão para tudo.
* Primeiro otimize o prompt; depois considere thinking se necessário.
* O campo `signature` protege a integridade do thinking gerado.
* A signature verifica que o texto foi gerado por Claude e não foi alterado.
* Pode haver blocos de `redacted thinking`.
* Redacted thinking ocorre por razões de segurança.
* Mesmo redigido, o bloco pode ser passado de volta para preservar contexto.
* Para ativar, usa-se `params["thinking"] = {"type": "enabled", "budget": thinking_budget}`.
* O `thinking_budget` mínimo apresentado é `1024`.
* `max_tokens` deve ser maior que o `thinking_budget`.
* Extended Thinking tem restrições de compatibilidade, incluindo message pre-filling e temperature, conforme o conteúdo.

---

## 4. Termos-chave

**Extended Thinking**
Recurso que permite ao Claude usar tokens adicionais para raciocinar antes de responder.

**Thinking block**
Bloco da resposta que contém o raciocínio gerado pelo modelo.

**Text block**
Bloco que contém a resposta final entregue ao usuário.

**Thinking budget**
Limite máximo de tokens disponíveis para o raciocínio.

**Signature**
Token criptográfico que valida a integridade do bloco de thinking.

**Redacted thinking**
Bloco de thinking ocultado ou criptografado por razões de segurança.

**`max_tokens`**
Limite total de tokens que Claude pode gerar na resposta.

**Message pre-filling**
Técnica de pré-preencher uma mensagem do assistente para controlar formato de saída.

**Temperature**
Parâmetro que controla aleatoriedade/criatividade da resposta.

**Response handling**
Lógica do código que processa os diferentes blocos retornados pela API.

---

## 5. Boas práticas

Use Extended Thinking somente quando houver necessidade real.

A ordem recomendada é:

```text
Prompt simples
↓
Prompt engineering
↓
Prompt evaluation
↓
Otimização
↓
Extended Thinking, se necessário
```

Também é importante ajustar corretamente o orçamento:

```python
thinking_budget=1024
```

e garantir que:

```text
max_tokens > thinking_budget
```

Na aplicação, prepare o código para lidar com múltiplos tipos de bloco:

* `thinking`;
* `redacted_thinking`;
* `text`.

Também não assuma que o thinking sempre estará legível. O sistema deve funcionar mesmo quando o thinking vier redigido.

---

## 6. Limitações, riscos e cuidados

Extended Thinking aumenta custo porque usa tokens extras.

Também aumenta latência, porque Claude passa mais tempo raciocinando antes de responder.

Outro cuidado é a compatibilidade. Segundo o conteúdo, Extended Thinking não é compatível com alguns recursos, como **message pre-filling** e **temperature**. Por isso, é importante verificar restrições antes de combinar recursos.

Também há complexidade adicional no código. A aplicação precisa processar blocos diferentes, incluindo possíveis blocos redigidos.

A signature não deve ser alterada. Se você modificar o thinking ou remover a assinatura, pode quebrar a continuidade ou a validação da conversa.

Além disso, mesmo com Extended Thinking, não há garantia absoluta de resposta correta. O recurso melhora capacidade de raciocínio em muitos casos, mas ainda deve ser avaliado com testes.

---

## 7. Resumo para revisão rápida

* Extended Thinking dá mais espaço de raciocínio ao Claude.
* A resposta pode vir com `thinking block` e `text block`.
* O bloco `text` é a resposta final.
* Thinking pode melhorar tarefas difíceis.
* Custa mais tokens.
* Aumenta latência.
* Exige tratamento mais complexo da resposta.
* Use apenas quando evals indicarem necessidade.
* Primeiro otimize o prompt.
* `thinking_budget` define tokens de raciocínio.
* O mínimo apresentado é `1024`.
* `max_tokens` deve ser maior que `thinking_budget`.
* `signature` valida que o thinking não foi alterado.
* `redacted thinking` pode aparecer por razões de segurança.
* Nem todos os recursos são compatíveis com Extended Thinking.

---

## 8. Perguntas simuladas

### 1. O que é Extended Thinking?

A) Um recurso que permite a Claude raciocinar com tokens adicionais antes da resposta final
B) Um mecanismo para reduzir o custo das chamadas
C) Uma ferramenta exclusiva para busca na web
D) Um tipo de JSON Schema para tools

**Resposta correta: A.**

**Explicação:** Extended Thinking dá ao modelo um orçamento de tokens para raciocinar antes de gerar a resposta.

---

### 2. Qual é a estrutura típica de uma resposta com thinking ativado?

A) Apenas um bloco de erro
B) Um bloco de thinking e um bloco de texto final
C) Apenas uma API key
D) Apenas um schema vazio

**Resposta correta: B.**

**Explicação:** A resposta pode conter um `thinking block` e depois um `text block` com a resposta final.

---

### 3. Qual bloco contém a resposta final ao usuário?

A) `thinking`
B) `text`
C) `signature`
D) `redacted`

**Resposta correta: B.**

**Explicação:** O bloco `text` contém a resposta final, enquanto `thinking` contém o raciocínio.

---

### 4. Qual é um benefício do Extended Thinking?

A) Melhorar o raciocínio em tarefas complexas
B) Eliminar todos os custos da API
C) Garantir compatibilidade com qualquer recurso
D) Remover a necessidade de testes

**Resposta correta: A.**

**Explicação:** O recurso pode melhorar desempenho em problemas difíceis, mas não elimina custos nem avaliação.

---

### 5. Qual é um trade-off importante do Extended Thinking?

A) Maior custo e maior latência
B) Menor qualidade obrigatória
C) Impossibilidade de gerar texto
D) Remoção automática de tools

**Resposta correta: A.**

**Explicação:** Thinking usa tokens adicionais e leva mais tempo para ser processado.

---

### 6. Quando o curso recomenda considerar Extended Thinking?

A) Depois de otimizar o prompt e avaliar que a qualidade ainda não é suficiente
B) Sempre, em todas as chamadas
C) Apenas para respostas de uma palavra
D) Antes de escrever qualquer prompt

**Resposta correta: A.**

**Explicação:** A recomendação é começar com prompting normal, avaliar e só usar thinking quando necessário.

---

### 7. Para que serve a `signature`?

A) Para verificar que o thinking foi gerado por Claude e não foi alterado
B) Para calcular cosine similarity
C) Para limitar o número de buscas web
D) Para criar chunks de texto

**Resposta correta: A.**

**Explicação:** A signature é um token criptográfico ligado ao bloco de thinking.

---

### 8. O que é redacted thinking?

A) Um bloco de thinking ocultado por razões de segurança
B) Uma resposta final sem texto
C) Um erro de Python
D) Um tipo de embedding

**Resposta correta: A.**

**Explicação:** Quando o thinking é sinalizado por sistemas de segurança, ele pode vir redigido ou criptografado.

---

### 9. O que o parâmetro `thinking_budget` controla?

A) A quantidade máxima de tokens para raciocínio
B) O número máximo de documentos no RAG
C) O tamanho do arquivo `.env`
D) O número de API keys disponíveis

**Resposta correta: A.**

**Explicação:** `thinking_budget` define o orçamento de tokens para o processo de thinking.

---

### 10. Segundo o conteúdo, qual é o valor mínimo apresentado para `thinking_budget`?

A) 1
B) 100
C) 1024
D) 10000

**Resposta correta: C.**

**Explicação:** O conteúdo afirma que o orçamento mínimo é `1024` tokens.

---

### 11. Qual relação deve existir entre `max_tokens` e `thinking_budget`?

A) `max_tokens` deve ser maior que `thinking_budget`
B) `max_tokens` deve ser sempre zero
C) `thinking_budget` deve ser maior que todo o contexto
D) Eles devem ser exatamente iguais

**Resposta correta: A.**

**Explicação:** É necessário haver espaço para o raciocínio e para a resposta final.

---

### 12. Qual frase resume melhor Extended Thinking?

A) Extended Thinking melhora tarefas complexas dando a Claude mais tokens para raciocinar, mas aumenta custo, latência e complexidade.
B) Extended Thinking substitui RAG e ferramentas.
C) Extended Thinking serve apenas para formatar JSON.
D) Extended Thinking elimina a necessidade de prompt evaluation.

**Resposta correta: A.**

**Explicação:** Esse é o principal trade-off do recurso: mais capacidade de raciocínio em troca de maior custo e complexidade.
