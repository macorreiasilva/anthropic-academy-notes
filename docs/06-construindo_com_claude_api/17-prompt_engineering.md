# Engenharia de prompt

```

```

## 1. Ideia central

O conteúdo explica **prompt engineering** como um processo de melhoria contínua de prompts.

A ideia central é que criar um prompt bom não acontece de uma vez. Você começa com uma versão simples, avalia o desempenho, aplica técnicas de melhoria, mede novamente e repete esse ciclo até obter resultados mais confiáveis.

O processo segue esta lógica:

**definir objetivo → escrever prompt inicial → avaliar → melhorar → reavaliar → repetir**

O exemplo usado é um prompt que gera **planos alimentares de um dia para atletas**, levando em conta altura, peso, objetivo e restrições alimentares.

---

## 2. Principais conceitos

### Prompt engineering

**Prompt engineering** é o processo de melhorar um prompt para obter respostas mais confiáveis, úteis e alinhadas ao objetivo.

Ele envolve:

* clareza nas instruções;
* definição de contexto;
* critérios de saída;
* estruturação do prompt;
* testes;
* iteração com base em resultados.

O foco não é apenas “escrever um prompt bonito”, mas criar um prompt que funcione bem de forma consistente.

---

### Processo iterativo de melhoria

O conteúdo apresenta um ciclo de cinco etapas:

1. **Set a goal**: definir o que o prompt deve realizar.
2. **Write an initial prompt**: criar uma primeira versão simples.
3. **Evaluate the prompt**: testar o prompt com critérios definidos.
4. **Apply prompt engineering techniques**: aplicar técnicas para melhorar o resultado.
5. **Re-evaluate**: medir novamente para verificar se houve melhoria.

As etapas 4 e 5 são repetidas até que o prompt atinja um desempenho satisfatório.

O ponto importante é que cada mudança deve ser avaliada de forma mensurável.

---

### Exemplo prático: plano alimentar para atletas

O caso de uso do exemplo é criar um prompt que gere um **plano alimentar compacto e conciso de um dia para um atleta**.

O prompt deve considerar:

* altura do atleta;
* peso;
* objetivo;
* restrições alimentares.

Exemplo de entradas:

```python
{
    "height": "Athlete's height in cm",
    "weight": "Athlete's weight in kg",
    "goal": "Goal of the athlete",
    "restrictions": "Dietary restrictions of the athlete"
}
```

O objetivo é gerar uma resposta útil, personalizada e estruturada.

---

### PromptEvaluator

A aula apresenta uma classe chamada `PromptEvaluator`, usada para automatizar parte do processo de avaliação.

Exemplo:

```python
evaluator = PromptEvaluator(max_concurrent_tasks=5)
```

Essa classe ajuda em tarefas como:

* gerar datasets de teste;
* executar avaliações;
* usar um modelo avaliador;
* produzir relatórios com pontuação e justificativas.

O parâmetro `max_concurrent_tasks` controla quantas tarefas podem rodar ao mesmo tempo.

---

### Concorrência e rate limits

O parâmetro:

```python
max_concurrent_tasks=5
```

define o nível de concorrência da avaliação.

Quanto maior o número, mais avaliações podem ser executadas simultaneamente. Isso pode acelerar o processo, mas também pode causar erros de **rate limit** se a conta ou API não permitir tantas chamadas ao mesmo tempo.

A recomendação da aula é começar com valores baixos, como:

```python
max_concurrent_tasks=3
```

Depois, aumentar conforme a cota da API permitir.

---

### Geração automática de dataset

O sistema de avaliação pode gerar casos de teste automaticamente.

Exemplo:

```python
dataset = evaluator.generate_dataset(
    task_description="Write a compact, concise 1 day meal plan for a single athlete",
    prompt_inputs_spec={
        "height": "Athlete's height in cm",
        "weight": "Athlete's weight in kg", 
        "goal": "Goal of the athlete",
        "restrictions": "Dietary restrictions of the athlete"
    },
    output_file="dataset.json",
    num_cases=3
)
```

Aqui, o avaliador cria um dataset com diferentes combinações de entradas.

Durante o desenvolvimento, a recomendação é usar poucos casos, como **2 ou 3**, para acelerar o ciclo de iteração.

Depois, na validação final, o número de casos pode ser aumentado.

---

### Prompt inicial

O conteúdo recomenda começar com um prompt simples, mesmo que ele seja fraco.

Exemplo:

```python
def run_prompt(prompt_inputs):
    prompt = f"""
What should this person eat?

- Height: {prompt_inputs["height"]}
- Weight: {prompt_inputs["weight"]}
- Goal: {prompt_inputs["goal"]}
- Dietary restrictions: {prompt_inputs["restrictions"]}
"""
    
    messages = []
    add_user_message(messages, prompt)
    return chat(messages)
```

Esse prompt provavelmente terá resultados ruins, mas isso é esperado. Ele serve como **baseline**, ou seja, uma referência inicial para medir melhorias futuras.

---

### Critérios extras de avaliação

Ao rodar a avaliação, é possível passar critérios adicionais para o modelo avaliador.

Exemplo:

```python
results = evaluator.run_evaluation(
    run_prompt_function=run_prompt,
    dataset_file="dataset.json",
    extra_criteria="""
The output should include:
- Daily caloric total
- Macronutrient breakdown  
- Meals with exact foods, portions, and timing
"""
)
```

Esses critérios indicam ao avaliador o que a resposta deve conter.

No exemplo, uma boa resposta deve incluir:

* total calórico diário;
* divisão de macronutrientes;
* refeições com alimentos exatos;
* porções;
* horários.

Isso torna a avaliação mais alinhada ao objetivo real da aplicação.

---

### Análise dos resultados

Depois de rodar a avaliação, o sistema retorna:

* uma pontuação numérica;
* um relatório HTML detalhado;
* justificativas do modelo avaliador para cada nota.

Uma pontuação inicial baixa, como **2.3 de 10**, é normal em uma primeira tentativa.

O objetivo não é acertar de primeira, mas melhorar progressivamente.

O relatório ajuda a entender:

* onde o prompt falhou;
* quais critérios não foram atendidos;
* quais mudanças podem melhorar a próxima versão;
* se uma alteração realmente aumentou a qualidade.

---

### Fazer uma mudança por vez

Uma recomendação importante é alterar o prompt de forma controlada.

Em vez de mudar muitas coisas ao mesmo tempo, o ideal é:

1. fazer uma alteração;
2. rodar a avaliação;
3. observar o impacto;
4. manter se melhorou;
5. descartar ou ajustar se piorou.

Isso ajuda a entender quais técnicas realmente trouxeram valor.

---

## 3. Pontos importantes para certificação

* **Prompt engineering é um processo iterativo**, não uma ação única.
* O ciclo envolve definir objetivo, escrever prompt, avaliar, melhorar e reavaliar.
* A melhoria deve ser guiada por métricas, não apenas por intuição.
* O prompt inicial serve como **baseline**.
* Uma pontuação inicial baixa é normal e útil para comparação.
* O dataset de avaliação deve representar os inputs esperados.
* Durante desenvolvimento, poucos casos de teste aceleram a iteração.
* Na validação final, o dataset deve ser ampliado.
* `PromptEvaluator` automatiza geração de dataset, avaliação e relatórios.
* `max_concurrent_tasks` controla concorrência e pode causar rate limit se for alto demais.
* Critérios extras ajudam o grader a avaliar o que realmente importa.
* Relatórios detalhados ajudam a diagnosticar falhas do prompt.
* A melhor prática é fazer **uma mudança por vez** e medir o impacto.

---

## 4. Termos-chave

**Prompt engineering**
Processo de melhorar prompts para obter respostas mais confiáveis e alinhadas ao objetivo.

**Iterative refinement**
Melhoria gradual por meio de ciclos de teste, ajuste e nova avaliação.

**Goal**
Objetivo que o prompt deve alcançar.

**Initial prompt**
Primeira versão do prompt, usada como ponto de partida.

**Baseline**
Resultado inicial usado como referência para comparar melhorias.

**PromptEvaluator**
Classe usada para automatizar geração de datasets e avaliação de prompts.

**Evaluation pipeline**
Fluxo que testa prompts, coleta respostas e calcula pontuações.

**Dataset**
Conjunto de casos de teste usados para avaliar o prompt.

**Prompt inputs**
Variáveis de entrada usadas pelo prompt, como altura, peso, objetivo e restrições.

**Extra criteria**
Critérios adicionais usados pelo avaliador para julgar a qualidade da resposta.

**Model grading**
Uso de um modelo para avaliar respostas geradas.

**HTML report**
Relatório visual detalhado com resultados, notas e justificativas.

**Concurrency**
Execução de múltiplas tarefas ao mesmo tempo.

**Rate limit**
Limite de chamadas permitido pela API em determinado período.

**Iteration**
Ciclo de melhoria baseado em avaliação e ajuste.

---

## 5. Boas práticas

* Comece com um prompt simples para criar uma baseline.
* Defina claramente o objetivo antes de tentar melhorar o prompt.
* Use um dataset de avaliação para medir desempenho.
* Durante o desenvolvimento, use poucos casos de teste para iterar rápido.
* Aumente o número de casos na validação final.
* Use critérios extras para alinhar a avaliação ao objetivo real.
* Comece com baixa concorrência para evitar rate limits.
* Analise o relatório detalhado, não apenas a nota média.
* Faça uma mudança por vez no prompt.
* Reavalie depois de cada alteração.
* Mantenha mudanças que geram melhoria mensurável.
* Use os resultados da avaliação para guiar a próxima iteração.

---

## 6. Limitações, riscos e cuidados

Um erro comum é tentar melhorar o prompt sem avaliação estruturada. Isso pode gerar mudanças que parecem melhores, mas que não melhoram de fato o desempenho.

Outro risco é usar um dataset pequeno demais na validação final. Dois ou três casos são úteis para desenvolvimento rápido, mas não são suficientes para confiar em produção.

Também é importante tomar cuidado com **rate limits**. Aumentar muito `max_concurrent_tasks` pode acelerar a avaliação, mas também pode gerar erros se a API bloquear excesso de chamadas simultâneas.

Outro ponto importante: uma pontuação numérica não deve ser analisada isoladamente. O relatório detalhado e o reasoning do grader ajudam a entender por que o prompt falhou.

Também é preciso evitar mudar várias partes do prompt ao mesmo tempo. Se a pontuação melhorar ou piorar, você não saberá qual alteração causou o efeito.

---

## 7. Resumo para revisão rápida

* Prompt engineering melhora prompts de forma iterativa.
* O ciclo é: objetivo, prompt inicial, avaliação, melhoria e reavaliação.
* O prompt inicial serve como baseline.
* Avaliação mostra se mudanças realmente melhoraram o resultado.
* O exemplo gera planos alimentares para atletas.
* Inputs usados: altura, peso, objetivo e restrições alimentares.
* `PromptEvaluator` ajuda a gerar dataset e rodar avaliações.
* `max_concurrent_tasks` controla concorrência.
* Comece com baixa concorrência para evitar rate limits.
* Use poucos casos durante desenvolvimento.
* Use mais casos para validação final.
* Critérios extras orientam o grader.
* Relatórios detalhados mostram falhas e oportunidades de melhoria.
* Faça uma mudança por vez e meça o impacto.

---

## 8. Perguntas simuladas

### 1. O que é prompt engineering?

A) O processo de criar API keys automaticamente
B) O processo de melhorar prompts para obter respostas mais confiáveis
C) Uma técnica usada apenas para gerar imagens
D) Um método para remover datasets de avaliação

**Resposta correta: B.**

**Explicação:** Prompt engineering envolve criar e refinar prompts para melhorar a qualidade e confiabilidade das respostas.

---

### 2. Qual é a ordem correta do processo iterativo apresentado?

A) Avaliar → apagar prompt → gerar API key → publicar
B) Definir objetivo → escrever prompt inicial → avaliar → aplicar técnicas → reavaliar
C) Criar dataset → ignorar resultados → aumentar temperature → finalizar
D) Escrever prompt → publicar direto → revisar depois

**Resposta correta: B.**

**Explicação:** O ciclo começa com o objetivo, passa pelo prompt inicial, avaliação, melhoria e nova avaliação.

---

### 3. Para que serve um prompt inicial simples?

A) Para funcionar como baseline de comparação
B) Para garantir nota 10 imediatamente
C) Para evitar qualquer avaliação
D) Para substituir o dataset

**Resposta correta: A.**

**Explicação:** O prompt inicial estabelece uma referência para medir melhorias futuras.

---

### 4. No exemplo, qual é o objetivo do prompt?

A) Gerar regras de firewall
B) Criar um plano alimentar de um dia para atletas
C) Escrever mensagens de commit
D) Traduzir código Python para JavaScript

**Resposta correta: B.**

**Explicação:** O exemplo trabalha com planos alimentares para atletas considerando dados individuais.

---

### 5. Quais entradas o prompt do exemplo deve considerar?

A) Nome, e-mail, senha e cidade
B) Altura, peso, objetivo e restrições alimentares
C) Modelo, API key, temperature e max_tokens
D) Data, hora, fuso horário e idioma

**Resposta correta: B.**

**Explicação:** O prompt recebe dados do atleta para personalizar o plano alimentar.

---

### 6. Para que serve `max_concurrent_tasks`?

A) Para controlar quantas tarefas rodam simultaneamente
B) Para definir a altura do atleta
C) Para escolher o formato JSON
D) Para limitar a quantidade de refeições

**Resposta correta: A.**

**Explicação:** Esse parâmetro controla a concorrência na execução das avaliações.

---

### 7. Por que começar com baixa concorrência, como 3?

A) Para evitar erros de rate limit
B) Para fazer Claude responder mais criativamente
C) Para impedir geração de dataset
D) Para eliminar a necessidade de avaliação

**Resposta correta: A.**

**Explicação:** Muitas chamadas simultâneas podem ultrapassar limites da API.

---

### 8. Por que usar poucos casos de teste durante o desenvolvimento?

A) Para acelerar o ciclo de iteração
B) Para garantir que o prompt nunca falhe
C) Para impedir comparação entre versões
D) Para reduzir a qualidade da avaliação final

**Resposta correta: A.**

**Explicação:** Poucos casos permitem rodar avaliações rapidamente enquanto o prompt está sendo ajustado.

---

### 9. O que os critérios extras de avaliação ajudam a definir?

A) O que o grader deve considerar como uma boa resposta
B) Qual API key deve ser usada
C) Como apagar o relatório HTML
D) Quantas chamadas serão bloqueadas

**Resposta correta: A.**

**Explicação:** Critérios extras orientam a avaliação de acordo com os requisitos do caso de uso.

---

### 10. Qual é uma boa prática ao melhorar prompts?

A) Fazer várias mudanças grandes ao mesmo tempo
B) Fazer uma mudança por vez e medir o impacto
C) Nunca reavaliar depois de alterar o prompt
D) Ignorar o relatório detalhado e olhar só a média

**Resposta correta: B.**

**Explicação:** Alterações controladas ajudam a identificar quais técnicas realmente melhoram o desempenho.
