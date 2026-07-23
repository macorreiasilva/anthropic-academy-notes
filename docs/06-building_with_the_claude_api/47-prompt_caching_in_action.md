# Cache de prompts em ação

```
notebooks\30-prompt_caching_in_action.ipynb
```

## 1. Ideia central

O conteúdo mostra o **Prompt Caching na prática**, explicando como implementar cache para reduzir **custo** e **latência** em chamadas repetidas à API do Claude.

A ideia principal é:

**Quando você envia repetidamente o mesmo conteúdo para Claude, como tool schemas ou system prompts longos, pode marcar essas partes para cache e reutilizar o processamento em requests futuras.**

O cache é temporário, dura **uma hora**, e só funciona quando o conteúdo enviado permanece **idêntico**.

---

## 2. Principais conceitos

### Como o prompt caching funciona

Na primeira request, Claude processa o conteúdo normalmente e grava o trabalho no cache.

```text
Initial request → write to cache
```

Nas requests seguintes, se o conteúdo for igual, Claude pode reutilizar o processamento salvo.

```text
Follow-up request → read from cache
```

Isso torna a chamada:

* mais rápida;
* mais barata;
* mais eficiente;
* especialmente útil para conteúdo longo e repetitivo.

---

### Conteúdos bons para cache

O conteúdo destaca três casos importantes:

| Conteúdo               | Por que cachear                                  |
| ---------------------- | ------------------------------------------------ |
| System prompts grandes | Costumam ser reutilizados em muitas chamadas     |
| Tool schemas complexos | Podem ter muitos tokens e mudar pouco            |
| Mensagens repetidas    | Úteis em fluxos com contexto longo ou follow-ups |

Exemplos citados:

* system prompt de assistente de código com cerca de **6K tokens**;
* múltiplos tool schemas com cerca de **1.7K tokens**;
* conteúdo de mensagem repetido em várias chamadas.

---

### Cache de tool schemas

Para cachear definições de ferramentas, adiciona-se `cache_control` ao **último tool schema** da lista.

Exemplo:

```python
if tools:
    tools_clone = tools.copy()
    last_tool = tools_clone[-1].copy()
    last_tool["cache_control"] = {"type": "ephemeral"}
    tools_clone[-1] = last_tool
    params["tools"] = tools_clone
```

Esse código faz três coisas importantes:

1. copia a lista de ferramentas;
2. copia a última ferramenta;
3. adiciona `cache_control` apenas na cópia.

Isso evita alterar diretamente a lista original de ferramentas.

---

### Por que copiar as ferramentas?

O conteúdo recomenda não modificar diretamente:

```python
tools[-1]["cache_control"] = {"type": "ephemeral"}
```

Embora isso funcione, pode causar problemas se a lista original for reutilizada, reordenada ou modificada em outro ponto do código.

A abordagem com cópia é mais segura:

```python
tools_clone = tools.copy()
last_tool = tools_clone[-1].copy()
```

Boa prática importante: **não mutar estruturas originais sem necessidade**, especialmente em aplicações que reutilizam configurações.

---

### Cache de system prompt

Para cachear um system prompt, ele não pode ser apenas uma string simples. É necessário transformá-lo em um bloco estruturado.

Exemplo:

```python
if system:
    params["system"] = [
        {
            "type": "text",
            "text": system,
            "cache_control": {"type": "ephemeral"}
        }
    ]
```

Isso permite adicionar `cache_control` ao system prompt.

---

### Métricas de uso do cache

A API retorna informações no campo de uso que ajudam a entender se o cache foi criado ou reutilizado.

Exemplo na primeira request:

```text
cache_creation_input_tokens = 1772
```

Isso indica que Claude criou cache para 1772 tokens.

Em uma follow-up request:

```text
cache_read_input_tokens = 1772
```

Isso indica que Claude leu 1772 tokens do cache, em vez de reprocessá-los do zero.

Essas métricas são essenciais para depurar e confirmar se o caching está funcionando.

---

### Sensibilidade do cache

O cache é extremamente sensível.

Se você mudar até mesmo **um único caractere** no conteúdo cacheado, o cache pode ser invalidado.

Exemplos de mudanças que podem invalidar cache:

* alterar uma palavra no system prompt;
* mudar a ordem das ferramentas;
* adicionar ou remover um campo do schema;
* mudar a descrição de uma ferramenta;
* alterar espaços ou conteúdo antes do breakpoint.

Por isso, para aproveitar o cache, partes cacheadas devem ser **estáveis e idênticas entre requests**.

---

### Ordem de cache

A ordem dos componentes importa.

Claude considera a request nesta ordem:

1. tools;
2. system prompt;
3. messages.

Isso permite cache parcial.

Exemplo:

Se as ferramentas continuam iguais, mas o system prompt muda:

```text
Tools iguais → cache lido
System prompt mudou → novo cache criado
```

Ou seja, Claude pode reutilizar o cache das ferramentas e criar um novo cache para o system prompt alterado.

---

### Cache granular

O conteúdo mostra que o caching pode ser granular.

Isso significa que você não precisa invalidar tudo quando apenas uma parte muda. Dependendo de onde estão os breakpoints e da ordem dos blocos, Claude pode reutilizar as partes que permaneceram iguais.

Esse comportamento é importante em aplicações reais, porque tool schemas e system prompts podem ter ciclos de mudança diferentes.

---

## 3. Pontos importantes para certificação

* Prompt caching reduz custo e latência em requests repetidas.
* O cache dura uma hora.
* A primeira request escreve no cache.
* Follow-up requests podem ler do cache.
* O cache só ajuda quando o conteúdo é idêntico.
* Tool schemas são bons candidatos para cache.
* System prompts longos são bons candidatos para cache.
* Para cachear tools, adicione `cache_control` ao último tool da lista.
* É boa prática copiar a lista de tools antes de adicionar `cache_control`.
* Para cachear system prompt, use formato estruturado com bloco de texto.
* `cache_creation_input_tokens` indica tokens gravados no cache.
* `cache_read_input_tokens` indica tokens lidos do cache.
* Pequenas mudanças podem invalidar o cache.
* A ordem relevante é: tools, system prompt, messages.
* Cache pode funcionar parcialmente quando apenas algumas partes mudam.
* Prompt caching não é armazenamento permanente; é uma otimização temporária.

---

## 4. Termos-chave

**Prompt caching**
Recurso que reutiliza o processamento de conteúdo repetido para reduzir custo e latência.

**Cache**
Armazenamento temporário do trabalho computacional feito em uma request.

**Cache control**
Campo usado para marcar um bloco como elegível para cache.

**Ephemeral cache**
Cache temporário, usado com `"type": "ephemeral"`.

**Tool schema**
Definição estruturada de uma ferramenta que Claude pode usar.

**System prompt**
Instrução de alto nível que define o comportamento do modelo.

**Cache creation tokens**
Tokens que foram processados e gravados no cache.

**Cache read tokens**
Tokens reutilizados a partir do cache.

**Cache invalidation**
Situação em que o cache deixa de ser reutilizável porque o conteúdo mudou.

**Cache ordering**
Ordem em que Claude processa tools, system prompt e messages para fins de cache.

**Granular caching**
Capacidade de reaproveitar partes específicas do cache mesmo quando outras partes mudam.

---

## 5. Boas práticas

Use prompt caching em partes que mudam pouco entre requests.

Bons candidatos:

```text
Tool schemas fixos
System prompts longos
Instruções detalhadas reutilizadas
Contexto de documento repetido
Histórico estável de conversa
```

Ao cachear tools, prefira copiar a lista antes de modificar:

```python
tools_clone = tools.copy()
last_tool = tools_clone[-1].copy()
last_tool["cache_control"] = {"type": "ephemeral"}
tools_clone[-1] = last_tool
```

Isso evita efeitos colaterais no restante da aplicação.

Ao cachear system prompts, use formato estruturado:

```python
params["system"] = [
    {
        "type": "text",
        "text": system,
        "cache_control": {"type": "ephemeral"}
    }
]
```

Também é uma boa prática monitorar:

```text
cache_creation_input_tokens
cache_read_input_tokens
```

Essas métricas confirmam se o cache está sendo criado ou reutilizado.

---

## 6. Limitações, riscos e cuidados

Prompt caching é poderoso, mas exige cuidado.

Principais limitações:

* o cache dura apenas uma hora;
* só funciona bem com conteúdo repetido;
* conteúdo precisa ser idêntico;
* pequenas alterações invalidam cache;
* não substitui memória permanente;
* não substitui banco de dados;
* não elimina a necessidade de gerenciar estado da conversa;
* cache mal posicionado pode gerar pouco benefício.

Um cuidado importante é evitar conteúdo dinâmico antes do breakpoint.

Exemplo ruim:

```text
Timestamp atual
ID aleatório
Mensagem específica do usuário
Cache breakpoint
```

Se esses valores mudam a cada request, o cache provavelmente não será reutilizado.

Outro risco é alterar tool schemas em tempo de execução. Como tools vêm antes do system prompt e das messages, mudanças nelas podem afetar o reaproveitamento do cache.

---

## 7. Resumo para revisão rápida

* Prompt caching torna chamadas repetidas mais rápidas e baratas.
* A primeira request grava no cache.
* Follow-ups podem ler do cache.
* O cache dura uma hora.
* O conteúdo precisa ser idêntico para ser reutilizado.
* Tool schemas são bons candidatos para cache.
* System prompts longos também são bons candidatos.
* Para tools, adicione `cache_control` ao último item da lista.
* Copie a lista de tools antes de modificar.
* Para system prompt, use bloco estruturado com `type: "text"`.
* `cache_creation_input_tokens` mostra cache criado.
* `cache_read_input_tokens` mostra cache reutilizado.
* A ordem é: tools, system prompt, messages.
* Mudanças pequenas podem invalidar o cache.
* Cache pode ser parcial quando apenas parte da request muda.

---

## 8. Perguntas simuladas

### 1. Qual é o principal benefício do prompt caching?

A) Tornar requests repetidas mais rápidas e baratas
B) Aumentar automaticamente a criatividade do modelo
C) Substituir a API key
D) Eliminar a necessidade de mensagens

**Resposta correta: A.**

**Explicação:** Prompt caching reutiliza processamento anterior, reduzindo latência e custo.

---

### 2. Quanto tempo o cache vive, segundo o conteúdo?

A) Uma hora
B) Um dia
C) Uma semana
D) Para sempre

**Resposta correta: A.**

**Explicação:** O cache é temporário e dura uma hora.

---

### 3. O que acontece na primeira request com caching habilitado?

A) Claude escreve conteúdo no cache
B) Claude sempre lê do cache
C) Claude ignora o system prompt
D) Claude remove as ferramentas

**Resposta correta: A.**

**Explicação:** A primeira request cria o cache para que requests futuras possam reutilizá-lo.

---

### 4. O que indica `cache_creation_input_tokens`?

A) Quantidade de tokens gravados no cache
B) Quantidade de tokens removidos da resposta
C) Quantidade de tokens de saída
D) Número de mensagens do usuário

**Resposta correta: A.**

**Explicação:** Esse campo mostra quantos tokens foram processados e escritos no cache.

---

### 5. O que indica `cache_read_input_tokens`?

A) Quantidade de tokens lidos do cache
B) Quantidade de imagens enviadas
C) Número de tool calls executadas
D) Total de erros na request

**Resposta correta: A.**

**Explicação:** Esse campo confirma que Claude reutilizou tokens previamente cacheados.

---

### 6. Onde o `cache_control` deve ser adicionado para cachear tool schemas?

A) No último tool da lista
B) Apenas na primeira mensagem do usuário
C) No campo `temperature`
D) No campo `max_tokens`

**Resposta correta: A.**

**Explicação:** O exemplo adiciona `cache_control` ao último item da lista de tools.

---

### 7. Por que é recomendado copiar a lista de tools antes de adicionar `cache_control`?

A) Para evitar modificar a estrutura original e causar efeitos colaterais
B) Porque Claude não aceita listas
C) Porque cache só funciona com listas vazias
D) Porque tools não podem ter schemas

**Resposta correta: A.**

**Explicação:** Copiar a lista e o último item evita problemas caso a lista original seja reutilizada.

---

### 8. Como um system prompt deve ser estruturado para receber cache control?

A) Como lista de blocos com `type: "text"` e `cache_control`
B) Como string simples sempre
C) Como número inteiro
D) Como imagem base64

**Resposta correta: A.**

**Explicação:** Para adicionar `cache_control`, o system prompt precisa estar no formato estruturado.

---

### 9. O que pode invalidar o cache?

A) Mudar um único caractere no conteúdo cacheado
B) Repetir exatamente o mesmo prompt
C) Usar o mesmo tool schema
D) Manter o mesmo system prompt

**Resposta correta: A.**

**Explicação:** O cache é sensível a mudanças. O conteúdo precisa ser idêntico.

---

### 10. Qual é a ordem relevante para cache ordering?

A) Tools, system prompt, messages
B) Messages, tools, system prompt
C) System prompt, messages, tools
D) Max tokens, temperature, stop sequence

**Resposta correta: A.**

**Explicação:** Claude processa primeiro tools, depois system prompt e depois mensagens.

---

### 11. O que acontece se as tools permanecem iguais, mas o system prompt muda?

A) Claude pode ler cache das tools e criar novo cache para o system prompt
B) Todo cache é sempre apagado permanentemente
C) Claude ignora as tools
D) A request falha automaticamente

**Resposta correta: A.**

**Explicação:** O caching pode ser granular, reutilizando partes que permanecem iguais.

---

### 12. Qual é um bom candidato para prompt caching?

A) Um system prompt longo usado em várias requests
B) Uma pergunta curta enviada uma única vez
C) Um timestamp que muda sempre
D) Um ID aleatório gerado a cada chamada

**Resposta correta: A.**

**Explicação:** Conteúdos longos, estáveis e repetidos são os melhores candidatos.

---

### 13. Qual é uma má prática ao usar prompt caching?

A) Colocar conteúdo dinâmico antes do breakpoint
B) Cachear tool schemas estáveis
C) Cachear system prompts longos
D) Monitorar tokens de cache

**Resposta correta: A.**

**Explicação:** Conteúdo dinâmico muda com frequência e pode invalidar o cache.

---

### 14. Qual frase resume melhor “Prompt caching in action”?

A) É a implementação prática de cache em tools e system prompts para reduzir custo e latência em requests repetidas.
B) É uma técnica para gerar embeddings vetoriais.
C) É um método para criar PDFs com citações.
D) É uma forma de substituir tool calling.

**Resposta correta: A.**

**Explicação:** O foco do conteúdo é mostrar como aplicar caching em requests reais, especialmente em tools e system prompts.
