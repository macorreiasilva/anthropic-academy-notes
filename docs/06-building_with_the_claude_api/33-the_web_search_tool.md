# A ferramenta de busca na web

## 1. Ideia central

O conteúdo apresenta a **Web Search Tool**, uma ferramenta integrada ao Claude que permite buscar informações na internet para responder perguntas que exigem dados atuais, especializados ou não presentes no treinamento do modelo.

A ideia central é:

**Com a Web Search Tool, Claude pode decidir automaticamente quando uma busca na web é útil, executar a busca e usar os resultados com citações para fundamentar a resposta.**

Diferente de tools customizadas, nessa ferramenta você **não precisa implementar a busca manualmente**. Claude gerencia o processo de pesquisa; sua aplicação só precisa habilitar a ferramenta com um schema simples.

---

## 2. Principais conceitos

### O que é a Web Search Tool

A **Web Search Tool** é uma ferramenta embutida no Claude para pesquisa na internet.

Ela é útil quando o usuário pergunta algo como:

```text
What are the latest developments in quantum computing?
```

Nesse caso, Claude pode transformar a intenção do usuário em uma query de busca, como:

```text
quantum computing news
```

Depois, ele analisa os resultados e responde com base nas fontes encontradas.

---

### Pré-requisito: habilitar no console

Um ponto importante do conteúdo é que a organização precisa **habilitar a Web Search Tool nas configurações do console** antes de usá-la.

Isso é um cuidado administrativo e de privacidade. A busca na web não fica automaticamente disponível em todos os ambientes.

---

### Schema básico da ferramenta

Para ativar a Web Search Tool em uma chamada, você fornece um schema simples:

```python
web_search_schema = {
    "type": "web_search_20250305",
    "name": "web_search",
    "max_uses": 5
}
```

Esse schema informa a Claude que a ferramenta de busca está disponível.

Os campos principais são:

| Campo      | Função                                  |
| ---------- | --------------------------------------- |
| `type`     | Versão/tipo da ferramenta de web search |
| `name`     | Nome da ferramenta                      |
| `max_uses` | Número máximo de buscas permitidas      |

---

### Campo `max_uses`

O campo `max_uses` limita quantas buscas Claude pode fazer.

Isso é importante porque Claude pode decidir fazer buscas adicionais com base nos primeiros resultados.

Por exemplo:

1. faz uma busca inicial;
2. percebe que precisa refinar a consulta;
3. faz uma segunda busca;
4. compara resultados;
5. gera a resposta final.

Sem limite, isso poderia gerar muitas chamadas desnecessárias.

---

### Como a resposta funciona

Quando Claude usa a Web Search Tool, a resposta pode conter diferentes tipos de blocos.

O conteúdo cita:

| Bloco                      | Função                                               |
| -------------------------- | ---------------------------------------------------- |
| `TextBlock`                | Texto comum gerado por Claude                        |
| `ServerToolUseBlock`       | Mostra a query exata que Claude executou             |
| `WebSearchToolResultBlock` | Contém os resultados da busca                        |
| `WebSearchResultBlock`     | Representa um resultado individual, com título e URL |
| Citation blocks            | Trechos que fundamentam afirmações na resposta       |

Esse formato permite ver:

* o que Claude decidiu buscar;
* quais resultados encontrou;
* quais fontes foram usadas;
* quais trechos sustentam a resposta final.

---

### Exemplo de estrutura da resposta

A imagem mostra algo parecido com:

```python
# Initial text block - Claude decides to use the web search tool
TextBlock(
    text="I'll help you find information about the best exercises for building leg muscle."
)

# Exact query Claude executed
ServerToolUseBlock(
    input={
        "query": "best exercises building leg muscle strength scientific research"
    }
)

# Search results that Claude found
WebSearchToolResultBlock(
    content=[
        WebSearchResultBlock(
            title="Gluteus Maximus Activation during Common Strength and Hypertrophy Exercises",
            type="web_search_result",
            url="https://..."
        )
    ]
)
```

Esse exemplo mostra que Claude não apenas responde: ele também registra a busca realizada e os resultados usados.

---

### Restrição por domínio

A Web Search Tool pode ser limitada a domínios específicos com `allowed_domains`.

Exemplo:

```python
web_search_schema = {
    "type": "web_search_20250305",
    "name": "web_search",
    "max_uses": 5,
    "allowed_domains": ["nih.gov"]
}
```

Isso é útil quando você quer fontes mais confiáveis ou especializadas.

No exemplo do conteúdo, para perguntas sobre saúde, medicina ou exercícios físicos, restringir a busca a domínios como `nih.gov` ajuda a priorizar fontes como PubMed e National Institutes of Health, em vez de blogs aleatórios.

---

### Renderização dos resultados

Os blocos da resposta foram pensados para diferentes formas de exibição na interface.

Boas práticas de UI:

* renderizar `TextBlock` como texto normal;
* mostrar resultados de busca como uma lista de fontes;
* exibir citações inline no texto;
* incluir domínio, título da página, URL e trecho citado.

Isso aumenta a transparência e ajuda o usuário a entender de onde veio a informação.

---

### Uso prático

A Web Search Tool é mais útil para:

* eventos atuais;
* notícias recentes;
* desenvolvimentos tecnológicos;
* informações especializadas;
* fact-checking;
* pesquisa com fontes;
* informações não presentes no treinamento do modelo;
* respostas que precisam de fontes confiáveis.

Exemplos:

```text
What are the latest developments in quantum computing?
```

```text
Find evidence-based exercises for building leg muscle.
```

```text
What changed recently in a specific regulation?
```

---

## 3. Pontos importantes para certificação

* A Web Search Tool é uma ferramenta integrada ao Claude.
* Ela permite buscar informações na internet.
* É útil para dados atuais, recentes ou especializados.
* A organização precisa habilitar a ferramenta no console antes do uso.
* Diferente de tools customizadas, Claude gerencia a busca automaticamente.
* A aplicação fornece apenas um schema simples para ativar a tool.
* O schema inclui `type`, `name` e `max_uses`.
* `max_uses` limita o número máximo de buscas.
* Claude pode fazer buscas adicionais se achar necessário.
* A resposta pode conter blocos como `TextBlock`, `ServerToolUseBlock`, `WebSearchToolResultBlock` e `WebSearchResultBlock`.
* `ServerToolUseBlock` mostra a query exata usada por Claude.
* `WebSearchResultBlock` representa resultados individuais.
* Citações ajudam a mostrar quais fontes sustentam a resposta.
* `allowed_domains` permite restringir a busca a domínios confiáveis.
* Restringir domínio é útil para áreas sensíveis, como saúde, ciência e pesquisa.
* A UI deve mostrar fontes e citações de forma transparente.

---

## 4. Termos-chave

**Web Search Tool**
Ferramenta integrada ao Claude para buscar informações na internet.

**Schema**
Objeto que declara a disponibilidade e configuração da ferramenta.

**`web_search_20250305`**
Tipo/versão da ferramenta de web search mostrado no exemplo.

**`web_search`**
Nome da ferramenta no schema.

**`max_uses`**
Limite máximo de buscas que Claude pode executar em uma chamada.

**`allowed_domains`**
Lista de domínios permitidos para restringir a busca.

**TextBlock**
Bloco de texto comum gerado por Claude.

**ServerToolUseBlock**
Bloco que mostra a chamada da ferramenta, incluindo a query usada.

**WebSearchToolResultBlock**
Bloco que contém os resultados retornados pela busca.

**WebSearchResultBlock**
Resultado individual de busca, normalmente com título, tipo e URL.

**Citation block**
Bloco ou marcação que conecta uma afirmação da resposta a uma fonte.

**Query**
Consulta de busca formulada por Claude.

**Source transparency**
Transparência sobre quais fontes foram usadas na resposta.

---

## 5. Boas práticas

Use a Web Search Tool quando a pergunta exigir informação atual, especializada ou verificável.

Use `max_uses` para controlar quantas buscas Claude pode fazer.

Use `allowed_domains` quando a qualidade da fonte for importante.

Exemplo para saúde ou ciência:

```python
web_search_schema = {
    "type": "web_search_20250305",
    "name": "web_search",
    "max_uses": 5,
    "allowed_domains": ["nih.gov"]
}
```

Mostre as fontes ao usuário de forma clara, principalmente em respostas baseadas em pesquisa.

Para interfaces de aplicação, uma boa experiência é:

* mostrar uma lista de fontes no topo ou no final;
* exibir citações próximas às afirmações;
* diferenciar claramente texto gerado de resultados recuperados;
* permitir que o usuário veja os links originais.

---

## 6. Limitações, riscos e cuidados

A Web Search Tool melhora o acesso a informações atuais, mas não garante que toda fonte encontrada seja correta ou confiável.

Por isso, em domínios sensíveis, como saúde, finanças, direito ou ciência, é recomendável restringir fontes com `allowed_domains`.

Outro cuidado é o excesso de buscas. Claude pode decidir fazer buscas adicionais, então `max_uses` ajuda a controlar custo, latência e comportamento.

Também é importante não confundir busca com verdade absoluta. A ferramenta retorna resultados da web, e Claude ainda precisa interpretar esses resultados corretamente.

Em aplicações reais, a interface deve deixar claro quais fontes foram usadas, especialmente quando a resposta influencia decisões importantes.

---

## 7. Resumo para revisão rápida

* A Web Search Tool permite Claude pesquisar na internet.
* Ela é integrada ao Claude.
* A organização precisa habilitá-la no console.
* Você ativa a ferramenta com um schema simples.
* Campos principais: `type`, `name`, `max_uses`.
* `max_uses` limita o número de buscas.
* Claude decide quando buscar.
* Claude também formula a query.
* A resposta inclui blocos de texto, tool use, resultados e citações.
* `ServerToolUseBlock` mostra a query executada.
* `WebSearchResultBlock` mostra resultados individuais.
* `allowed_domains` restringe a busca a fontes específicas.
* Citações aumentam transparência e confiança.
* A ferramenta é ideal para dados atuais, pesquisa e fact-checking.

---

## 8. Perguntas simuladas

### 1. Para que serve a Web Search Tool?

A) Para permitir que Claude pesquise informações na internet
B) Para armazenar API keys
C) Para editar arquivos locais
D) Para controlar a temperature

**Resposta correta: A.**

**Explicação:** A Web Search Tool permite que Claude busque dados atuais ou especializados na web.

---

### 2. O que precisa acontecer antes de usar a Web Search Tool?

A) A organização precisa habilitar a ferramenta no console
B) O usuário precisa criar manualmente um buscador em Python
C) Claude precisa ser treinado novamente
D) O arquivo `.env` precisa ser apagado

**Resposta correta: A.**

**Explicação:** O conteúdo destaca que a ferramenta deve ser habilitada nas configurações da organização.

---

### 3. Qual campo limita quantas buscas Claude pode executar?

A) `name`
B) `max_uses`
C) `messages`
D) `temperature`

**Resposta correta: B.**

**Explicação:** `max_uses` define o número máximo de buscas permitidas.

---

### 4. Qual é o papel do campo `allowed_domains`?

A) Restringir a busca a domínios específicos
B) Definir o nome do modelo
C) Criar uma API key
D) Salvar o resultado em arquivo

**Resposta correta: A.**

**Explicação:** `allowed_domains` permite limitar fontes, por exemplo a `nih.gov`.

---

### 5. Por que restringir a busca a `nih.gov` pode ser útil?

A) Para obter fontes mais confiáveis em temas médicos, científicos ou de saúde
B) Para impedir qualquer citação
C) Para transformar resultados em JSON
D) Para remover a necessidade de busca

**Resposta correta: A.**

**Explicação:** Domínios como NIH/PubMed tendem a ser mais adequados para respostas baseadas em evidência.

---

### 6. O que o `ServerToolUseBlock` mostra?

A) A query exata que Claude executou
B) A senha do usuário
C) O custo final da API
D) O histórico completo do navegador

**Resposta correta: A.**

**Explicação:** Esse bloco registra a chamada da ferramenta e a consulta usada.

---

### 7. O que é um `WebSearchResultBlock`?

A) Um resultado individual de busca, com informações como título e URL
B) Um system prompt
C) Uma variável de ambiente
D) Um tipo de temperature

**Resposta correta: A.**

**Explicação:** Ele representa cada resultado encontrado pela busca.

---

### 8. Qual é uma boa prática de renderização da resposta?

A) Mostrar fontes e citações de forma clara para o usuário
B) Esconder todas as fontes usadas
C) Misturar URLs com API keys
D) Remover os resultados da interface

**Resposta correta: A.**

**Explicação:** Mostrar fontes aumenta transparência e confiança na resposta.

---

### 9. Qual caso de uso combina melhor com Web Search Tool?

A) Perguntar “quanto é 2 + 2?”
B) Perguntar as últimas novidades sobre computação quântica
C) Pedir para reescrever uma frase já fornecida
D) Pedir para listar mensagens anteriores da conversa

**Resposta correta: B.**

**Explicação:** Desenvolvimentos recentes exigem informação atualizada, tornando web search apropriado.

---

### 10. Qual afirmação resume melhor a Web Search Tool?

A) Claude pode pesquisar a web automaticamente quando a ferramenta está habilitada e declarada no schema.
B) Claude sempre precisa que você implemente manualmente a busca em Python.
C) A ferramenta serve apenas para editar arquivos.
D) A busca na web elimina a necessidade de verificar fontes.

**Resposta correta: A.**

**Explicação:** A Web Search Tool é integrada: Claude gerencia a busca, mas a aplicação precisa habilitar e fornecer o schema.
