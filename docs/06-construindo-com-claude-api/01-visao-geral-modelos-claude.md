# Visão Geral dos Modelos de Claude

# Resumo do conteúdo

## 1. Ideia central

A ideia principal é entender **quando escolher Claude Opus, Sonnet ou Haiku** com base em três critérios de prova: **capacidade**, **custo** e **latência**.

A comparação está correta como recorte entre **Opus 4.8, Sonnet 5 e Haiku 4.5**, mas há um detalhe importante: a documentação atual também lista **Claude Fable 5** como modelo mais capaz amplamente lançado; portanto, Opus/Sonnet/Haiku não representam mais “toda” a família atual, e sim três níveis práticos bastante relevantes.

## 2. Principais conceitos

**Claude Opus 4.8** é voltado para **codificação agêntica complexa, raciocínio avançado e trabalho empresarial**. Ele tem preço de **US$5 por milhão de tokens de entrada** e **US$25 por milhão de tokens de saída**, janela de contexto de **1 milhão de tokens** e latência moderada em comparação com Fable, mas mais custoso que Sonnet e Haiku.

**Claude Sonnet 5** é descrito como a melhor combinação de **velocidade e inteligência**. É o modelo mais indicado para a maioria das cargas de produção, especialmente quando se busca equilíbrio entre qualidade, custo e tempo de resposta. Seu preço padrão é **US$3/US$15 por milhão de tokens**, com preço introdutório de **US$2/US$10 até 31 de agosto de 2026**.

**Claude Haiku 4.5** é o modelo mais rápido e barato entre esses três, com preço de **US$1/US$5 por milhão de tokens**. É adequado para tarefas simples, de alto volume ou sensíveis a latência, como classificação, extração, resumo, moderação e chatbots em tempo real.

Um ponto técnico importante: **Opus 4.8 e Sonnet 5 têm janela de contexto de 1 milhão de tokens**, enquanto **Haiku 4.5 tem 200 mil tokens**. A saída máxima também difere: Opus e Sonnet chegam a **128 mil tokens**, enquanto Haiku chega a **64 mil tokens**.

## 3. Pontos importantes para certificação

A regra prática mais provável de aparecer em prova é:

**Haiku = rápido e barato para tarefas simples.
Sonnet = equilíbrio para produção.
Opus = tarefas complexas, agênticas e de maior risco.**

Outro ponto que pode cair: não basta comparar “inteligência”. Em produção, a escolha do modelo deve considerar **custo, latência, janela de contexto, volume, risco da tarefa e necessidade de raciocínio**.

Atenção para a diferença entre **extended thinking** e **adaptive thinking**. Na documentação atual, **Haiku 4.5 aparece com extended thinking**, enquanto **Opus 4.8 e Sonnet 5 aparecem com adaptive thinking**; portanto, dizer genericamente que “todos usam extended thinking” pode esconder uma diferença importante.

No **Sonnet 5**, o “manual extended thinking” com `budget_tokens` foi removido; a recomendação é usar **adaptive thinking** com o parâmetro de **effort**.

## 4. Termos-chave

**Latência**: tempo que o modelo leva para responder. Haiku tende a ser o mais rápido; Opus tende a ser mais lento quando usado em tarefas complexas.

**MTok**: milhão de tokens. Os preços são normalmente expressos em custo por milhão de tokens de entrada e saída.

**Token de entrada**: texto, imagem ou conteúdo enviado ao modelo, convertido em tokens.

**Token de saída**: resposta gerada pelo modelo, também cobrada em tokens.

**Janela de contexto**: quantidade máxima de informação que o modelo consegue considerar em uma única interação ou sessão.

**Extended thinking**: modo em que o modelo usa mais processamento interno para raciocinar antes de responder.

**Adaptive thinking**: forma mais nova de pensamento em que o modelo decide quando e quanto raciocinar, com controle via parâmetro de esforço.

**Effort**: parâmetro que controla o trade-off entre profundidade de raciocínio, custo e latência. Em Opus 4.8 e Sonnet 5, o padrão é `high` em APIs e ferramentas principais, e níveis como `low`, `medium`, `high`, `xhigh` e `max` podem ser usados conforme suporte do ambiente.

## 5. Boas práticas

Use **Haiku** quando a tarefa for simples, repetitiva, de alto volume ou precisar de resposta quase instantânea.

Use **Sonnet** como escolha padrão para a maioria dos casos de produção: agentes comuns, escrita, análise, codificação do dia a dia, RAG e automações.

Use **Opus** quando o erro for caro, a tarefa exigir raciocínio em várias etapas, houver codificação complexa, planejamento longo ou uso intensivo de ferramentas.

Ao migrar para **Sonnet 5**, recontar tokens é importante, porque a documentação informa que o novo tokenizer pode produzir cerca de **30% mais tokens** para o mesmo texto em comparação com Sonnet 4.6, afetando custo e limites de saída.

## 6. Limitações, riscos e cuidados

Um erro comum é escolher sempre o modelo “mais forte”. Isso pode aumentar custo e latência sem necessidade.

Outro erro é escolher sempre o modelo mais barato. Haiku pode ser excelente para tarefas simples, mas pode não ser ideal quando a tarefa exige raciocínio profundo, decisões ambíguas ou uso complexo de ferramentas.

Também é importante não confundir **preço por token** com **custo real final**. O custo real depende do tamanho do prompt, da saída, do uso de pensamento, do uso de cache, da quantidade de chamadas e do volume em produção.

Em tarefas de maior risco, como decisões legais, financeiras, médicas, segurança, compliance ou ações irreversíveis, o modelo mais capaz ainda deve ser combinado com validação humana, testes, logs e avaliação de qualidade.

## 7. Resumo para revisão rápida

- **Opus 4.8**: mais indicado para raciocínio complexo, codificação agêntica e tarefas de alto risco.
- **Sonnet 5**: melhor equilíbrio entre inteligência, velocidade e custo.
- **Haiku 4.5**: mais rápido e barato; ideal para alto volume e baixa latência.
- **Opus/Sonnet**: janela de contexto de 1M tokens.
- **Haiku**: janela de contexto de 200K tokens.
- **Sonnet 5**: preço promocional até **31/08/2026**.
- **Adaptive thinking ≠ extended thinking**.
- Para prova: a escolha do modelo é um problema de **trade-off** entre qualidade, custo e latência.

## 8. Perguntas simuladas

**1. Qual modelo é mais adequado para uma tarefa simples de classificação em alto volume e com baixo orçamento?**

A) Claude Opus 4.8

B) Claude Sonnet 5

C) Claude Haiku 4.5

D) Claude Fable 5

**Resposta correta: C.**

Haiku é o modelo mais rápido e barato entre Opus, Sonnet e Haiku, sendo adequado para classificação, extração e tarefas de alto volume.

---

**2. Qual modelo tende a ser a melhor escolha padrão para a maioria das cargas de produção?**

A) Haiku, porque sempre é o mais barato

B) Sonnet, porque equilibra inteligência e velocidade

C) Opus, porque sempre tem melhor custo-benefício

D) Nenhum, todos têm o mesmo perfil

**Resposta correta: B.**

Sonnet é posicionado como o modelo de equilíbrio, ideal para muitos casos de produção.

---

**3. Quando faz mais sentido usar Claude Opus 4.8?**

A) Quando a tarefa é simples e precisa de resposta instantânea

B) Quando o objetivo principal é reduzir custo ao máximo

C) Quando a tarefa exige raciocínio complexo, codificação agêntica ou alta confiabilidade

D) Quando a janela de contexto deve ser sempre menor

**Resposta correta: C.**

Opus é indicado para tarefas complexas, especialmente quando uma resposta errada pode sair cara.

---

**4. Qual afirmação está correta sobre janela de contexto?**

A) Opus, Sonnet e Haiku têm todos 1 milhão de tokens

B) Opus e Sonnet têm 1 milhão; Haiku tem 200 mil

C) Haiku tem a maior janela de contexto

D) Sonnet tem apenas 64 mil tokens de contexto

**Resposta correta: B.**

Opus 4.8 e Sonnet 5 têm 1M tokens de contexto; Haiku 4.5 tem 200K.

---

**5. Qual é o principal risco de escolher sempre o modelo mais poderoso?**

A) O modelo deixa de responder

B) O custo e a latência podem ser maiores sem necessidade

C) A janela de contexto diminui

D) A saída máxima sempre cai

**Resposta correta: B.**

A escolha de modelo deve considerar trade-offs. Usar Opus para tarefas simples pode ser desperdício.

---

**6. Qual diferença conceitual é importante entre extended thinking e adaptive thinking?**

A) São exatamente a mesma coisa

B) Adaptive thinking decide dinamicamente quando e quanto pensar; extended thinking é um modo mais manual/tradicional

C) Extended thinking só existe em modelos de imagem

D) Adaptive thinking reduz a janela de contexto para zero

**Resposta correta: B.**

A documentação distingue esses modos. Em especial, Sonnet 5 usa adaptive thinking, e o manual extended thinking com `budget_tokens` foi removido.