# Dados estruturados

```
006_controlling_output.ipynb
```

## 1. Ideia central

O conteúdo explica como fazer Claude gerar **dados estruturados limpos**, como JSON, código Python, listas ou CSV, sem adicionar explicações, markdown ou texto extra.

A ideia principal é:

**Quando você precisa de uma saída pronta para ser usada por uma aplicação, deve controlar o formato da resposta com técnicas como assistant message prefilling e stop sequences.**

Isso é especialmente útil quando o objetivo é obter apenas o dado bruto, por exemplo um JSON válido, sem frases antes ou depois.

---

## 2. Principais conceitos

### O problema das respostas padrão

Por padrão, Claude tenta ser útil. Então, quando você pede um JSON, ele pode responder com:

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["running"]
  }
}
```

E depois adicionar uma explicação como:

```text
This rule captures EC2 instance state changes when instances start running.
```

A resposta está correta, mas não é ideal para uma aplicação que precisa apenas do JSON puro.

Em um app web, por exemplo, o usuário espera clicar em “Generate” e copiar imediatamente o JSON gerado. Se Claude incluir markdown e explicações, o usuário precisa selecionar manualmente apenas a parte útil.

---

### Dados estruturados

**Dados estruturados** são saídas com formato previsível e organizado.

Exemplos:

* JSON;
* código Python;
* listas com bullets;
* CSV;
* YAML;
* XML;
* regras de configuração;
* objetos usados por APIs.

Esses formatos geralmente precisam ser limpos, porque serão copiados, analisados por código ou enviados para outro sistema.

---

### Assistant message prefilling

**Assistant message prefilling** é uma técnica em que você adiciona uma mensagem anterior do assistente como se Claude já tivesse começado a responder.

Exemplo:

````python
add_assistant_message(messages, "```json")
````

Isso “pré-preenche” a resposta de Claude com o início de um bloco de código JSON.

Claude então tende a continuar a partir desse ponto, gerando apenas o conteúdo esperado.

---

### Stop sequences

**Stop sequences** são sequências que fazem Claude parar de gerar quando aparecem.

No exemplo, a stop sequence é:

````python
stop_sequences=["```"]
````

Isso significa que, quando Claude tentar fechar o bloco de código markdown com:

```text
```

````

a geração será interrompida imediatamente.

Assim, o conteúdo retornado fica apenas com o JSON interno, sem o fechamento do bloco e sem explicações adicionais.

---

### Combinação das duas técnicas

A solução apresentada combina:

1. **Mensagem do usuário** pedindo o conteúdo.
2. **Mensagem do assistente pré-preenchida** com o início do formato.
3. **Stop sequence** para interromper a geração no ponto certo.

Exemplo:

```python
messages = []

add_user_message(messages, "Generate a very short event bridge rule as json")
add_assistant_message(messages, "```json")

text = chat(messages, stop_sequences=["```"])
````

O funcionamento é:

1. O usuário pede uma regra do EventBridge em JSON.
2. A mensagem pré-preenchida faz Claude continuar como se já estivesse dentro de um bloco JSON.
3. Claude gera o JSON.
4. Quando tenta fechar o bloco com ``` , a stop sequence para a geração.
5. O resultado final é apenas o JSON limpo.

---

### Processando a resposta

Mesmo com essa técnica, a resposta pode vir com espaços ou quebras de linha extras.

Para limpar e converter o texto em JSON, usa-se:

```python
import json

clean_json = json.loads(text.strip())
```

Aqui:

* `text.strip()` remove espaços e quebras de linha no início e no fim;
* `json.loads()` transforma a string JSON em um objeto Python.

Isso é importante porque, em aplicações reais, muitas vezes você não quer apenas exibir o JSON, mas também validar ou manipular os dados.

---

### Além de JSON

A técnica não serve apenas para JSON.

Ela pode ser usada sempre que você precisa de conteúdo estruturado sem comentários extras.

Exemplos:

| Tipo de saída  | Uso possível                             |
| -------------- | ---------------------------------------- |
| JSON           | Configurações, payloads de API, regras   |
| Python         | Snippets de código limpos                |
| CSV            | Dados tabulares                          |
| Bulleted lists | Listas padronizadas                      |
| YAML           | Arquivos de configuração                 |
| SQL            | Queries prontas para execução ou revisão |

A chave é identificar como Claude normalmente envolveria o conteúdo e usar isso a seu favor no prefill e na stop sequence.

---

## 3. Pontos importantes para certificação

* Claude costuma adicionar explicações porque tenta ser útil.
* Em aplicações, muitas vezes é necessário obter **apenas dados brutos**.
* Dados estruturados precisam ter formato limpo e previsível.
* JSON com markdown e explicações pode atrapalhar a experiência do usuário.
* **Assistant message prefilling** ajuda a direcionar o início da resposta.
* **Stop sequences** ajudam a controlar onde a geração deve parar.
* A combinação de prefill + stop sequence permite gerar saídas mais limpas.
* Para JSON, pode-se pré-preencher com `"```json"` e parar em `"```"`.
* O texto retornado pode precisar de limpeza com `.strip()`.
* `json.loads()` pode ser usado para validar e converter JSON em objeto Python.
* Essa técnica também funciona para código, listas, CSV e outros formatos.
* O objetivo é facilitar integração com aplicações que esperam dados estruturados.

---

## 4. Termos-chave

**Structured data**
Dados organizados em um formato previsível, como JSON, CSV, YAML ou listas.

**JSON**
Formato de dados muito usado em APIs e configurações, baseado em pares chave-valor.

**Raw data**
Dado bruto, sem explicações, markdown ou texto adicional.

**Assistant message prefilling**
Técnica de iniciar previamente a resposta do assistente para orientar Claude a continuar em um formato específico.

**Stop sequence**
Sequência de texto que interrompe a geração quando é encontrada.

**Markdown code block**
Bloco de código em markdown iniciado e encerrado por três crases.

**`stop_sequences`**
Parâmetro usado para definir uma ou mais sequências de parada.

**`text.strip()`**
Método Python que remove espaços e quebras de linha no início e no fim de uma string.

**`json.loads()`**
Função Python que converte uma string JSON em um objeto Python.

**AWS EventBridge rule**
Regra usada no AWS EventBridge para capturar e reagir a eventos em serviços da AWS.

---

## 5. Boas práticas

Use **assistant message prefilling** quando quiser que Claude comece ou continue uma resposta em um formato específico.

Use **stop sequences** quando precisar impedir que Claude adicione fechamento de markdown, explicações ou conteúdo extra.

Para JSON, uma estratégia útil é:

````python
add_assistant_message(messages, "```json")
text = chat(messages, stop_sequences=["```"])
````

Depois, limpe e valide a resposta:

```python
clean_json = json.loads(text.strip())
```

Em aplicações reais, sempre valide o dado estruturado antes de usar em produção.

Também é uma boa prática criar prompts claros, por exemplo:

```text
Generate only valid JSON. Do not include explanations.
```

Mas o conteúdo mostra que apenas pedir isso nem sempre é suficiente. Por isso, prefill e stop sequences dão mais controle.

---

## 6. Limitações, riscos e cuidados

Essa técnica melhora muito o controle do formato, mas não garante perfeição absoluta.

Claude ainda pode gerar JSON inválido, campos inesperados ou estruturas incompletas, especialmente se o pedido for ambíguo.

Por isso, em aplicações reais, é importante:

* validar JSON com `json.loads()`;
* tratar erros de parsing;
* definir campos obrigatórios;
* rejeitar ou corrigir saídas inválidas;
* evitar executar código gerado sem revisão;
* não confiar cegamente em dados estruturados gerados pelo modelo.

Outro cuidado: se a stop sequence for mal escolhida, ela pode interromper a resposta cedo demais.

Por exemplo, se você usar uma sequência que também pode aparecer dentro do conteúdo, Claude pode parar antes de terminar a saída.

Também é importante lembrar que remover explicações melhora integração com sistemas, mas pode reduzir a transparência para o usuário. Em ferramentas técnicas, talvez seja útil oferecer separadamente uma explicação opcional.

---

## 7. Resumo para revisão rápida

* Claude tende a adicionar explicações por padrão.
* Para aplicações, às vezes precisamos apenas do dado bruto.
* Dados estruturados incluem JSON, código, CSV e listas.
* JSON com markdown e explicação pode atrapalhar o usuário.
* Assistant message prefilling inicia a resposta em um formato desejado.
* Stop sequences interrompem a geração em um ponto específico.
* Para JSON, pode-se pré-preencher com `"```json"`.
* A stop sequence `"```"` impede que Claude feche o bloco e continue explicando.
* Use `.strip()` para limpar espaços e quebras de linha.
* Use `json.loads()` para validar e converter JSON.
* A técnica também funciona para código, listas e CSV.
* Sempre valide saídas estruturadas antes de usar em produção.

---

## 8. Perguntas simuladas

### 1. Qual é o problema comum ao pedir JSON para Claude sem controle adicional?

A) Claude nunca consegue gerar JSON
B) Claude pode adicionar markdown e explicações ao redor do JSON
C) Claude sempre retorna CSV em vez de JSON
D) Claude apaga automaticamente as chaves do objeto

**Resposta correta: B.**

**Explicação:** Claude tenta ser útil e pode envolver o JSON em blocos markdown e adicionar explicações, o que nem sempre é desejado.

---

### 2. Em uma aplicação web que gera JSON para o usuário copiar, qual saída é mais desejável?

A) JSON puro, sem texto extra
B) Um parágrafo explicando o JSON antes do conteúdo
C) Um bloco markdown com comentários longos
D) Uma resposta em linguagem natural sem estrutura

**Resposta correta: A.**

**Explicação:** Para facilitar cópia e uso imediato, a aplicação geralmente precisa do JSON bruto.

---

### 3. O que é assistant message prefilling?

A) Uma forma de armazenar API keys
B) Uma técnica de pré-preencher a resposta do assistente para guiar a continuação de Claude
C) Um método para aumentar automaticamente a janela de contexto
D) Uma ferramenta para converter JSON em imagem

**Resposta correta: B.**

**Explicação:** Prefilling consiste em adicionar uma mensagem do assistente como se Claude já tivesse começado a responder.

---

### 4. Para que servem stop sequences?

A) Para definir quando Claude deve parar de gerar texto
B) Para instalar o SDK da Anthropic
C) Para transformar texto em embeddings
D) Para escolher automaticamente o modelo

**Resposta correta: A.**

**Explicação:** Stop sequences interrompem a geração quando uma sequência específica aparece.

---

### 5. No exemplo de geração de JSON, por que a mensagem do assistente é pré-preenchida com `"```json"`?

A) Para fazer Claude continuar como se estivesse dentro de um bloco JSON
B) Para impedir que Claude gere qualquer texto
C) Para transformar a API key em JSON
D) Para fazer Claude responder em linguagem natural

**Resposta correta: A.**

**Explicação:** O prefill orienta Claude a continuar a partir do início de um bloco de código JSON.

---

### 6. No exemplo, por que a stop sequence é `"```"`?

A) Porque ela fecha o bloco de código markdown e permite parar antes de explicações extras
B) Porque ela inicia uma nova chamada à API
C) Porque ela define o modelo usado
D) Porque ela converte o JSON em CSV

**Resposta correta: A.**

**Explicação:** Quando Claude tenta fechar o bloco markdown, a geração para, deixando apenas o JSON.

---

### 7. Qual função Python pode validar e converter uma string JSON em objeto Python?

A) `json.dumps()`
B) `json.loads()`
C) `text.parse()`
D) `dict.read()`

**Resposta correta: B.**

**Explicação:** `json.loads()` lê uma string JSON e a converte em uma estrutura Python.

---

### 8. Para que serve `text.strip()` nesse contexto?

A) Para remover espaços e quebras de linha no início e no fim da resposta
B) Para traduzir JSON para Python
C) Para apagar campos obrigatórios
D) Para enviar a mensagem para Claude

**Resposta correta: A.**

**Explicação:** `.strip()` limpa caracteres extras antes ou depois do conteúdo principal.

---

### 9. Essa técnica pode ser usada além de JSON?

A) Não, funciona apenas com JSON
B) Sim, pode ser usada com código, listas, CSV e outros formatos estruturados
C) Não, só funciona com imagens
D) Sim, mas apenas sem stop sequences

**Resposta correta: B.**

**Explicação:** O mesmo padrão pode ser adaptado para diferentes tipos de saída estruturada.

---

### 10. Qual é um risco ao usar stop sequences?

A) A resposta pode parar cedo demais se a sequência aparecer dentro do conteúdo
B) A API key pode ser exibida automaticamente
C) Claude deixa de aceitar mensagens do usuário
D) O modelo ignora completamente o prompt

**Resposta correta: A.**

**Explicação:** Uma stop sequence mal escolhida pode aparecer no conteúdo gerado e interromper a resposta antes do esperado.
