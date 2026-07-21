# Apresentando a Geração Aumentada de Recuperação (RAG)

## 1. Ideia central

O conteúdo introduz **Retrieval Augmented Generation**, ou **RAG**.

A ideia central é que, quando temos documentos muito grandes, como um relatório financeiro com mais de 800 páginas, nem sempre é viável colocar todo o conteúdo dentro do prompt.

Em vez disso, o RAG propõe uma abordagem mais eficiente:

**dividir o documento em pedaços menores, encontrar os trechos mais relevantes para a pergunta do usuário e enviar apenas esses trechos para Claude responder.**

Em resumo:

**RAG = buscar conteúdo relevante + usar esse conteúdo para gerar uma resposta.**

---

## 2. Principais conceitos

### O problema com documentos grandes

O exemplo apresentado envolve um documento financeiro privado de mais de 800 páginas.

O usuário pode querer fazer perguntas específicas, como:

```text
What risk factors does this company have?
```

O desafio é que Claude não sabe automaticamente o conteúdo desse documento privado.

Para responder, é necessário fornecer a Claude as partes relevantes do documento.

---

### Opção 1: incluir o documento inteiro no prompt

A primeira solução seria colocar todo o conteúdo do documento dentro do prompt.

Exemplo:

```xml
Answer the user's question about the financial document.

<user_question>
{user_question}
</user_question>

<financial_document>
{financial_document}
</financial_document>
```

Essa abordagem é chamada informalmente de **prompt stuffing**, ou seja, “encher” o prompt com todo o conteúdo disponível.

Ela é simples, mas pode ser ruim para documentos muito grandes.

Problemas principais:

* existe um limite máximo de contexto;
* o documento pode ser grande demais para caber no prompt;
* prompts longos custam mais;
* prompts longos demoram mais para processar;
* Claude pode ficar menos eficiente quando há conteúdo demais e pouco relevante.

---

### Opção 2: dividir o documento em chunks

A segunda opção é a base do RAG.

O documento é dividido em vários pedaços menores, chamados **chunks**.

Exemplos de chunks no slide:

* Strategy Outlook;
* Balance Sheet;
* Risk Factors;
* Auditor’s Report;
* Key Performance Indicators;
* Market Data.

Quando o usuário pergunta:

```text
What risks does this company face?
```

o sistema procura os chunks mais relevantes, como:

```text
Risk Factors
```

Depois, apenas esse trecho relevante é colocado no prompt.

---

### Como o RAG funciona

O fluxo básico de RAG é:

1. Receber ou armazenar documentos.
2. Dividir os documentos em chunks.
3. Receber a pergunta do usuário.
4. Buscar os chunks mais relevantes para a pergunta.
5. Inserir esses chunks no prompt.
6. Claude gera a resposta usando os trechos recuperados.

Visualmente:

```text
Documento grande
↓
Chunks menores
↓
Pergunta do usuário
↓
Busca por chunks relevantes
↓
Prompt com pergunta + trechos relevantes
↓
Resposta de Claude
```

---

### Por que RAG é útil

RAG permite que Claude responda perguntas sobre documentos grandes sem precisar ler tudo a cada chamada.

Isso é importante porque Claude pode se concentrar apenas no conteúdo mais relevante.

Benefícios citados:

* funciona melhor com documentos muito grandes;
* pode funcionar com múltiplos documentos;
* reduz tamanho do prompt;
* reduz custo;
* reduz latência;
* melhora o foco da resposta;
* evita enviar conteúdo irrelevante.

---

### Desafios do RAG

Apesar dos benefícios, RAG não é automático nem trivial.

Ele exige algumas decisões técnicas.

Principais desafios:

* é necessário fazer um pré-processamento para dividir documentos em chunks;
* é preciso ter um mecanismo de busca para encontrar chunks relevantes;
* os chunks recuperados podem não conter todo o contexto necessário;
* há várias formas de dividir texto em chunks;
* escolher a melhor estratégia de chunking depende do caso de uso.

Por exemplo, é possível dividir um documento:

* por tamanho fixo;
* por parágrafos;
* por páginas;
* por seções;
* por títulos e subtítulos;
* por estrutura semântica.

Cada abordagem tem vantagens e desvantagens.

---

### Trade-off principal

O conteúdo destaca que RAG troca **simplicidade** por **escalabilidade e eficiência**.

Colocar tudo no prompt é mais simples, mas pode ser caro, lento ou impossível em documentos grandes.

RAG exige mais trabalho inicial, mas permite lidar com documentos muito maiores e com coleções de documentos.

---

## 3. Pontos importantes para certificação

* **RAG significa Retrieval Augmented Generation.**
* RAG é usado para responder perguntas com base em documentos grandes ou múltiplos documentos.
* Em vez de colocar todo o documento no prompt, RAG recupera apenas os trechos relevantes.
* Documentos são divididos em **chunks**.
* A pergunta do usuário é usada para encontrar chunks relevantes.
* Os chunks relevantes são inseridos no prompt.
* Claude gera a resposta com base nesses trechos.
* Incluir o documento inteiro no prompt pode ser simples, mas tem limitações.
* Prompts muito grandes custam mais e demoram mais.
* Claude pode ser menos eficiente com prompts muito longos.
* RAG reduz custo, latência e tamanho do prompt.
* RAG exige pré-processamento, chunking e mecanismo de busca.
* Um risco do RAG é recuperar chunks incompletos ou insuficientes.
* A escolha da estratégia de chunking impacta a qualidade da resposta.
* RAG é especialmente útil para documentos privados que Claude não conhece.

---

## 4. Termos-chave

**RAG**
Técnica que combina recuperação de informações relevantes com geração de resposta pelo modelo.

**Retrieval Augmented Generation**
Geração aumentada por recuperação; busca trechos relevantes antes de gerar a resposta.

**Large document**
Documento grande demais para ser colocado facilmente em um único prompt.

**Prompt stuffing**
Estratégia de colocar todo o conteúdo disponível dentro do prompt.

**Chunk**
Pedaço menor de um documento usado em sistemas RAG.

**Chunking**
Processo de dividir documentos em partes menores.

**Retrieval**
Etapa de busca dos chunks mais relevantes para a pergunta.

**Relevant chunks**
Trechos do documento que têm maior relação com a pergunta do usuário.

**Preprocessing**
Etapa anterior à consulta, em que documentos são preparados, divididos e organizados.

**Context window**
Limite de conteúdo que pode ser enviado ao modelo em uma única chamada.

**Private document**
Documento que não faz parte do treinamento do modelo e precisa ser fornecido pela aplicação.

**Search mechanism**
Sistema usado para encontrar trechos relevantes dentro dos documentos.

---

## 5. Boas práticas

Use RAG quando o documento for grande demais para caber confortavelmente em um prompt.

Também use RAG quando houver muitos documentos e o usuário fizer perguntas específicas.

Boas práticas:

* dividir documentos em chunks com significado claro;
* preservar metadados, como título da seção, página ou origem;
* recuperar apenas os trechos relevantes;
* usar XML tags para separar pergunta e documento no prompt;
* evitar enviar conteúdo irrelevante;
* testar diferentes estratégias de chunking;
* avaliar se os chunks recuperados contêm contexto suficiente;
* comparar qualidade, custo e latência entre abordagens.

Exemplo de prompt estruturado com RAG:

```xml
Answer the user's question about the financial document.

<user_question>
What risks does this company face?
</user_question>

<financial_document>
[Chunk relevante da seção Risk Factors]
</financial_document>
```

---

## 6. Limitações, riscos e cuidados

RAG não garante automaticamente respostas corretas.

A qualidade da resposta depende muito da qualidade da recuperação.

Se o sistema recuperar o chunk errado, Claude pode responder com base em informação incompleta ou irrelevante.

Riscos comuns:

* chunk pequeno demais e sem contexto;
* chunk grande demais e com ruído;
* busca que não encontra a seção certa;
* pergunta ambígua;
* perda de informações importantes durante o chunking;
* documentos mal estruturados;
* resposta baseada em trechos insuficientes.

Também é importante lembrar que RAG adiciona complexidade ao sistema. É necessário implementar ou configurar:

* extração de texto;
* divisão em chunks;
* indexação;
* busca;
* montagem do prompt;
* avaliação da qualidade das respostas.

Por isso, RAG é mais indicado quando os benefícios compensam essa complexidade.

---

## 7. Resumo para revisão rápida

* RAG ajuda Claude a trabalhar com documentos grandes.
* Documentos privados não são conhecidos por Claude.
* Colocar tudo no prompt pode ser caro, lento ou impossível.
* RAG divide documentos em chunks.
* A pergunta do usuário guia a busca pelos chunks relevantes.
* Apenas os trechos úteis entram no prompt.
* Claude responde usando os chunks recuperados.
* RAG reduz custo e latência.
* RAG melhora o foco da resposta.
* RAG exige pré-processamento e mecanismo de busca.
* A estratégia de chunking é uma decisão importante.
* O maior risco é recuperar contexto insuficiente ou errado.
* RAG troca simplicidade por escalabilidade e eficiência.

---

## 8. Perguntas simuladas

### 1. O que significa RAG?

A) Random Answer Generation
B) Retrieval Augmented Generation
C) Recursive API Gateway
D) Real-time Anthropic Guidance

**Resposta correta: B.**

**Explicação:** RAG significa Retrieval Augmented Generation, uma técnica que recupera informações relevantes antes de gerar a resposta.

---

### 2. Qual problema o RAG ajuda a resolver?

A) Falta de API key
B) Documentos grandes demais para colocar inteiros em um prompt
C) Erros de instalação do Python
D) Ausência de system prompt

**Resposta correta: B.**

**Explicação:** RAG é útil quando documentos são grandes demais, caros ou lentos para serem enviados inteiros ao modelo.

---

### 3. No exemplo, qual pergunta o usuário faz sobre o documento financeiro?

A) “What is the weather today?”
B) “What risks does this company face?”
C) “Generate a poem about finance.”
D) “Translate this document to Spanish.”

**Resposta correta: B.**

**Explicação:** A pergunta busca uma seção específica do documento: fatores de risco.

---

### 4. Qual é uma limitação de colocar o documento inteiro no prompt?

A) Não existe limite de tamanho de prompt
B) Prompts maiores custam mais e demoram mais
C) Claude sempre ignora documentos longos
D) Documentos financeiros não podem ser enviados para modelos

**Resposta correta: B.**

**Explicação:** Prompts grandes aumentam custo e latência, além de poderem ultrapassar limites de contexto.

---

### 5. O que são chunks?

A) API keys temporárias
B) Pedaços menores de um documento
C) Respostas finais de Claude
D) Erros de sintaxe JSON

**Resposta correta: B.**

**Explicação:** Chunks são trechos em que o documento é dividido para facilitar recuperação e uso no prompt.

---

### 6. Em RAG, o que acontece quando o usuário faz uma pergunta?

A) O sistema busca chunks relevantes e os inclui no prompt
B) Claude ignora todos os documentos
C) O documento inteiro é sempre enviado automaticamente
D) O modelo é treinado novamente

**Resposta correta: A.**

**Explicação:** O sistema recupera trechos relevantes para a pergunta e os fornece a Claude.

---

### 7. Qual seria um chunk relevante para a pergunta “What risks does this company face?”

A) Risk Factors
B) Random Footer
C) Document Cover Image
D) Font Settings

**Resposta correta: A.**

**Explicação:** A seção “Risk Factors” provavelmente contém os riscos enfrentados pela empresa.

---

### 8. Qual é um benefício do RAG?

A) Reduzir tamanho do prompt e focar no conteúdo relevante
B) Eliminar completamente a necessidade de avaliar respostas
C) Garantir que Claude nunca erre
D) Remover todos os custos da API

**Resposta correta: A.**

**Explicação:** RAG envia apenas os trechos relevantes, reduzindo ruído, custo e latência.

---

### 9. Qual é um desafio do RAG?

A) Escolher como dividir o documento em chunks
B) Criar uma API key manualmente a cada pergunta
C) Impedir o uso de documentos privados
D) Remover perguntas do usuário

**Resposta correta: A.**

**Explicação:** Existem várias estratégias de chunking, e a escolha impacta a qualidade da recuperação.

---

### 10. Qual frase resume melhor o trade-off do RAG?

A) RAG é mais simples que colocar tudo no prompt, mas sempre pior.
B) RAG troca simplicidade por escalabilidade e eficiência.
C) RAG só serve para documentos pequenos.
D) RAG substitui completamente a necessidade de Claude.

**Resposta correta: B.**

**Explicação:** RAG exige mais implementação, mas permite lidar melhor com documentos grandes e múltiplas fontes.
