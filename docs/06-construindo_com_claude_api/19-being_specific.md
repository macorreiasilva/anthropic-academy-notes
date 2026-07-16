# Sendo específico

## 1. Ideia central

O conteúdo explica a importância de ser **específico** ao criar prompts para Claude.

A ideia central é que, quanto mais específicas forem as instruções, menor será a chance de Claude interpretar a tarefa de forma ampla, genérica ou inconsistente.

Em resumo:

**Ser específico significa dar critérios claros sobre o resultado esperado e, quando necessário, indicar o processo que Claude deve seguir.**

Isso melhora a consistência, a qualidade e a previsibilidade das respostas.

---

## 2. Principais conceitos

### Por que ser específico importa

Quando você dá uma instrução vaga, Claude pode seguir muitos caminhos diferentes.

Exemplo genérico:

```text
Write a short story about a character who discovers a hidden talent.
```

Esse prompt deixa várias coisas em aberto:

* o tamanho da história;
* o número de personagens;
* o tipo de talento;
* o tom da narrativa;
* a estrutura;
* o conflito;
* o desfecho.

Claude ainda pode gerar uma boa resposta, mas a variação será maior e talvez o resultado não seja o que você queria.

Ao adicionar critérios específicos, você dá a Claude um alvo mais claro.

---

### Dois tipos de especificidade

O conteúdo apresenta dois tipos principais de diretrizes:

1. **Output Quality Guidelines**
2. **Process Steps**

Eles podem ser usados separadamente, mas em prompts profissionais muitas vezes aparecem juntos.

---

### 1. Output Quality Guidelines

**Output Quality Guidelines** são instruções que definem as características que a resposta final deve ter.

Elas controlam aspectos como:

* tamanho da resposta;
* estrutura;
* formato;
* elementos obrigatórios;
* tom;
* estilo;
* nível de detalhe.

Exemplo para uma história:

```text
Write a story under 1,000 words. Include one main character, one supporting character, and a clear action that reveals the hidden talent.
```

Esse prompt é mais específico porque limita o tamanho, define personagens e exige uma cena de revelação do talento.

---

### 2. Process Steps

**Process Steps** são instruções passo a passo que orientam como Claude deve pensar ou trabalhar antes de chegar à resposta final.

Esse tipo de especificidade é útil quando a tarefa exige análise, comparação, diagnóstico ou tomada de decisão.

Exemplo de processo para escrever uma história:

```text
First, brainstorm three talents that could create dramatic tension.
Then, pick the most interesting one.
Next, outline a pivotal scene that reveals the talent.
Finally, write the story.
```

Esses passos ajudam Claude a construir a resposta de maneira mais sistemática.

---

### Exemplo com plano alimentar

No caso do prompt de plano alimentar, adicionar diretrizes específicas melhorou muito a avaliação.

A pontuação subiu de:

```text
3.92 → 7.86
```

As diretrizes adicionadas foram:

1. incluir quantidade calórica diária precisa;
2. mostrar quantidades de proteína, gordura e carboidratos;
3. especificar quando comer cada refeição;
4. usar apenas alimentos compatíveis com as restrições;
5. listar todas as porções em gramas;
6. manter a proposta econômica se isso for mencionado.

Essas instruções transformam um pedido genérico em uma tarefa bem definida.

---

### Quando usar output guidelines

O conteúdo recomenda usar **output guidelines quase sempre**.

Elas funcionam como uma rede de segurança para garantir que a resposta tenha formato, conteúdo e qualidade mais consistentes.

Exemplos de output guidelines:

```text
Keep the answer under 300 words.
Use a table.
Include three examples.
Avoid technical jargon.
Return only valid JSON.
Use a professional tone.
```

---

### Quando usar process steps

**Process steps** são mais úteis em tarefas complexas.

Use passos quando a tarefa envolver:

* troubleshooting;
* tomada de decisão;
* análise crítica;
* comparação de opções;
* diagnóstico;
* múltiplas perspectivas;
* problemas com várias causas possíveis.

Exemplo citado: se Claude precisa analisar por que o desempenho de uma equipe de vendas caiu, não basta perguntar genericamente. É melhor instruí-lo a examinar:

* métricas de mercado;
* mudanças na indústria;
* desempenho individual;
* mudanças organizacionais;
* feedback de clientes.

Isso evita que Claude foque em apenas uma causa possível.

---

### Combinar as duas abordagens

Em prompts profissionais, é comum combinar:

* diretrizes de qualidade da saída;
* passos de processo.

Isso oferece dois benefícios:

1. **Consistência na resposta final**
2. **Confiança de que Claude considerou fatores importantes**

Exemplo:

```text
Analyze why the sales team's performance dropped.

Process:
1. Examine market metrics.
2. Consider industry changes.
3. Review individual performance.
4. Check organizational changes.
5. Consider customer feedback.

Output:
- Summarize the top three likely causes.
- Include supporting evidence for each.
- Recommend next steps.
- Keep the answer under 500 words.
```

---

## 3. Pontos importantes para certificação

* Ser específico é uma das formas mais eficazes de melhorar prompts.
* Prompts vagos deixam muitas decisões para o modelo.
* Diretrizes específicas aumentam consistência e qualidade.
* Existem dois tipos principais de especificidade: **output guidelines** e **process steps**.
* Output guidelines definem como a resposta final deve ser.
* Process steps definem como Claude deve trabalhar ou raciocinar.
* Output guidelines devem ser usadas em quase todos os prompts.
* Process steps são especialmente úteis para tarefas complexas.
* No exemplo do plano alimentar, diretrizes específicas elevaram a pontuação de **3.92 para 7.86**.
* Prompts profissionais frequentemente combinam diretrizes de saída com passos de processo.
* Ser específico reduz ambiguidade e melhora previsibilidade.

---

## 4. Termos-chave

**Specificity**
Grau de detalhe e clareza das instruções dadas no prompt.

**Output Quality Guidelines**
Diretrizes que definem características esperadas da resposta final.

**Process Steps**
Passos que orientam como Claude deve abordar ou resolver a tarefa.

**Consistency**
Capacidade de gerar respostas semelhantes em qualidade e formato em diferentes execuções.

**Ambiguity**
Falta de clareza que permite múltiplas interpretações.

**Structure**
Organização da resposta, como tabela, tópicos, seções ou JSON.

**Format**
Forma final esperada da saída, como texto, lista, tabela, código ou JSON.

**Tone**
Estilo comunicativo da resposta, como profissional, simples, didático ou criativo.

**Troubleshooting**
Processo de diagnosticar e resolver problemas.

**Decision-making**
Processo de analisar opções e escolher uma decisão.

---

## 5. Boas práticas

* Inclua diretrizes de saída em quase todos os prompts.
* Defina o tamanho esperado da resposta quando isso importar.
* Especifique o formato desejado, como tabela, tópicos ou JSON.
* Liste elementos obrigatórios que a resposta deve conter.
* Informe restrições importantes, como orçamento, idioma, tom ou público.
* Use passos de processo para tarefas complexas.
* Oriente Claude a considerar múltiplos fatores quando houver risco de análise superficial.
* Combine diretrizes de saída e passos de processo em prompts profissionais.
* Evite prompts genéricos como “faça um plano” ou “analise isso”.
* Meça o impacto das mudanças com avaliações, sempre que possível.

---

## 6. Limitações, riscos e cuidados

Ser específico melhora bastante o resultado, mas há alguns cuidados.

Um prompt específico demais pode ficar rígido e impedir que Claude use flexibilidade quando ela seria útil.

Outro risco é criar diretrizes conflitantes, por exemplo:

```text
Be extremely concise.
Include detailed explanations for every item.
```

Também é possível sobrecarregar o prompt com muitas regras, fazendo Claude priorizar algumas e esquecer outras.

Por isso, as diretrizes devem ser:

* claras;
* relevantes;
* não conflitantes;
* alinhadas ao objetivo principal.

Além disso, process steps devem ser usados com cuidado. Em tarefas simples, muitos passos podem tornar a resposta desnecessariamente longa.

---

## 7. Resumo para revisão rápida

* Ser específico melhora qualidade e consistência.
* Prompts vagos deixam Claude decidir demais.
* Existem dois tipos principais de especificidade.
* Output guidelines definem a resposta final.
* Process steps definem o processo de raciocínio/trabalho.
* Use output guidelines quase sempre.
* Use process steps em problemas complexos.
* No plano alimentar, a pontuação subiu de **3.92 para 7.86**.
* Diretrizes podem definir tamanho, formato, tom e elementos obrigatórios.
* Passos ajudam em troubleshooting, decisões e análises críticas.
* Prompts profissionais costumam combinar as duas abordagens.

---

## 8. Perguntas simuladas

### 1. Por que ser específico melhora os resultados com Claude?

A) Porque impede Claude de receber mensagens do usuário
B) Porque reduz ambiguidade e dá um alvo mais claro para a resposta
C) Porque elimina totalmente a necessidade de avaliação
D) Porque força Claude a sempre responder com código

**Resposta correta: B.**

**Explicação:** Instruções específicas reduzem interpretações possíveis e ajudam Claude a produzir respostas mais alinhadas ao objetivo.

---

### 2. Quais são os dois tipos principais de diretrizes citados?

A) API keys e model names
B) Output Quality Guidelines e Process Steps
C) Temperature e max_tokens
D) JSON e Python

**Resposta correta: B.**

**Explicação:** O conteúdo diferencia diretrizes de qualidade da saída e passos de processo.

---

### 3. O que são Output Quality Guidelines?

A) Instruções que definem características da resposta final
B) Regras para instalar o SDK
C) Etapas internas de autenticação da API
D) Métricas de cobrança da Anthropic

**Resposta correta: A.**

**Explicação:** Elas controlam tamanho, estrutura, formato, elementos obrigatórios, tom e estilo.

---

### 4. O que são Process Steps?

A) Passos que orientam Claude a abordar a tarefa de forma sistemática
B) Campos obrigatórios de uma API key
C) Tokens gerados no streaming
D) Códigos de erro da API

**Resposta correta: A.**

**Explicação:** Process steps indicam como Claude deve trabalhar, analisar ou raciocinar antes de responder.

---

### 5. Quando é recomendado usar Output Quality Guidelines?

A) Quase sempre
B) Apenas em prompts de imagem
C) Nunca em aplicações profissionais
D) Somente quando não há restrições

**Resposta correta: A.**

**Explicação:** O conteúdo recomenda usar diretrizes de qualidade em praticamente todos os prompts.

---

### 6. Quando os Process Steps são especialmente úteis?

A) Em tarefas complexas, troubleshooting e tomada de decisão
B) Apenas para criar API keys
C) Apenas para respostas de uma palavra
D) Somente para reduzir custo de tokens

**Resposta correta: A.**

**Explicação:** Passos ajudam quando Claude precisa considerar múltiplos fatores ou seguir uma análise sistemática.

---

### 7. No exemplo do plano alimentar, qual foi o impacto de adicionar diretrizes específicas?

A) A pontuação caiu para zero
B) A pontuação subiu de 3.92 para 7.86
C) A avaliação deixou de funcionar
D) Claude passou a ignorar restrições alimentares

**Resposta correta: B.**

**Explicação:** A especificidade mais que dobrou a qualidade medida pela avaliação.

---

### 8. Qual destas é uma boa diretriz específica para um plano alimentar?

A) “Make it good.”
B) “List all portion sizes in grams.”
C) “Say something about food.”
D) “Be nice.”

**Resposta correta: B.**

**Explicação:** Essa diretriz é concreta e verificável, ajudando a controlar a qualidade da saída.

---

### 9. Qual é um risco de prompts específicos demais?

A) Podem ficar rígidos ou conter regras conflitantes
B) Sempre geram respostas vazias
C) Impedem o uso de datasets
D) Apagam automaticamente o histórico

**Resposta correta: A.**

**Explicação:** Especificidade excessiva ou mal planejada pode criar rigidez, conflitos e dificuldade de seguir todas as regras.

---

### 10. Qual é a melhor descrição de um prompt profissional?

A) Um prompt vago que deixa Claude decidir tudo
B) Um prompt que combina diretrizes de saída e, quando necessário, passos de processo
C) Um prompt que sempre omite restrições
D) Um prompt que nunca deve ser avaliado

**Resposta correta: B.**

**Explicação:** Prompts profissionais costumam controlar tanto o formato da saída quanto o processo de análise quando a tarefa é complexa.
