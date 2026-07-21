# Estrutura com tags XML

## 1. Ideia central

O conteúdo explica como usar **XML tags** para dar estrutura aos prompts, principalmente quando eles contêm muito contexto, dados longos, código, documentação ou várias variáveis interpoladas.

A ideia central é:

**Use XML tags para separar partes distintas do prompt e deixar claro para Claude o que cada bloco representa.**

Isso ajuda Claude a diferenciar instruções, dados, código, documentação e exemplos.

---

## 2. Principais conceitos

### Por que estruturar prompts

Quando um prompt mistura muitas informações, Claude pode ter dificuldade para entender quais partes são:

* instruções;
* dados de entrada;
* documentação;
* código;
* exemplos;
* requisitos de saída.

Nas imagens, o exemplo mostra um prompt pedindo um relatório sobre queda de 30% nas vendas. Sem tags, os registros aparecem apenas como:

```text
{sales_records}
```

Com XML tags, o mesmo conteúdo fica delimitado:

```xml
<sales_records>
{sales_records}
</sales_records>
```

Isso deixa claro que aquele bloco contém os registros de vendas que devem ser analisados.

---

### XML tags como delimitadores

XML tags funcionam como marcadores de início e fim.

Exemplo:

```xml
<sales_records>
...dados de vendas...
</sales_records>
```

A tag de abertura indica onde o bloco começa:

```xml
<sales_records>
```

A tag de fechamento indica onde ele termina:

```xml
</sales_records>
```

Esses limites ajudam Claude a entender melhor a estrutura do prompt.

---

### Exemplo: código e documentação

A terceira imagem mostra um exemplo muito importante.

Na versão **“Not Great”**, o prompt mistura código e documentação no mesmo bloco, sem separação clara. Isso torna difícil saber o que é o código do usuário e o que é documentação de referência.

Na versão **“Better”**, o prompt usa tags:

```xml
<my_code>
from datavortex import Pipeline, DataSource

def process_data(input_file, output_file):
    pipeline = Pipeline()
    source = DataSource.from_csv(input_file)
</my_code>

<docs>
# Creating a data source from data vortex
csv_source = DataSource.from_csv("data.csv")
</docs>
```

Agora Claude consegue identificar:

* o bloco `<my_code>` como o código que precisa ser depurado;
* o bloco `<docs>` como a documentação que deve ser usada como referência.

Esse exemplo demonstra que XML tags reduzem ambiguidade em prompts técnicos.

---

### Nomes personalizados de tags

Você não precisa usar tags XML oficiais. Pode criar nomes que façam sentido para o conteúdo.

Exemplos bons:

```xml
<sales_records>
```

```xml
<athlete_information>
```

```xml
<my_code>
```

```xml
<docs>
```

O nome da tag deve ser **descritivo**. Por exemplo:

```xml
<sales_records>
```

é melhor do que:

```xml
<data>
```

porque informa exatamente que tipo de dado está dentro do bloco.

---

### Quando usar XML tags

XML tags são mais úteis quando o prompt inclui:

* muito contexto;
* dados longos;
* documentação;
* código;
* diferentes tipos de conteúdo;
* múltiplas variáveis interpoladas;
* instruções complexas;
* exemplos e requisitos de saída.

Mesmo em prompts menores, elas podem ajudar, mas o benefício fica maior conforme o prompt cresce em complexidade.

---

## 3. Pontos importantes para certificação

* XML tags são usadas para **estruturar prompts**.
* Elas ajudam a separar partes distintas do prompt.
* Funcionam como **delimitadores** para Claude.
* São especialmente úteis quando há muito contexto ou muitos dados.
* Ajudam a distinguir instruções de dados.
* Ajudam a separar código de documentação.
* Tags personalizadas são permitidas.
* Tags descritivas são melhores que tags genéricas.
* `<sales_records>` é melhor que `<data>`.
* `<my_code>` e `<docs>` ajudam Claude a entender a diferença entre código e documentação.
* XML tags não executam código nem mudam a API; elas apenas organizam o prompt.
* Quanto mais complexo o prompt, mais valiosas as XML tags se tornam.

---

## 4. Termos-chave

**XML tags**
Marcadores usados para delimitar e nomear seções dentro de um prompt.

**Delimiter**
Elemento que separa claramente o início e o fim de uma seção.

**Content boundaries**
Limites entre diferentes blocos de conteúdo no prompt.

**Prompt structure**
Organização interna do prompt para facilitar a compreensão pelo modelo.

**Interpolated data**
Dados inseridos dinamicamente em um prompt, como `{sales_records}`.

**Custom tags**
Tags criadas pelo próprio usuário para organizar conteúdo, como `<my_code>` ou `<docs>`.

**Descriptive tag names**
Nomes de tags que explicam claramente o conteúdo do bloco.

**Context**
Informações fornecidas ao modelo para orientar a resposta.

**Documentation block**
Seção contendo documentação de referência.

**Code block**
Seção contendo código a ser analisado, corrigido ou explicado.

---

## 5. Boas práticas

Use XML tags quando o prompt tiver várias partes diferentes.

Prefira nomes claros e específicos:

```xml
<athlete_information>
```

em vez de:

```xml
<info>
```

Separe blocos de conteúdo com função diferente:

```xml
<instructions>
...
</instructions>

<data>
...
</data>

<output_requirements>
...
</output_requirements>
```

Para código e documentação, use algo como:

```xml
<my_code>
...
</my_code>

<docs>
...
</docs>
```

Para dados de vendas:

```xml
<sales_records>
...
</sales_records>
```

Depois dos blocos, escreva a tarefa de forma clara e direta.

Exemplo:

```xml
<sales_records>
{sales_records}
</sales_records>

Write a one-page decision report explaining why sales dropped 30% last quarter. Follow the analysis steps below.
```

---

## 6. Limitações, riscos e cuidados

XML tags ajudam Claude a entender a estrutura, mas não garantem sozinhas uma resposta perfeita.

Elas devem ser combinadas com outras técnicas de prompt engineering, como:

* ser claro e direto;
* ser específico;
* definir critérios de saída;
* incluir passos de processo;
* avaliar o prompt com um dataset.

Também é importante evitar tags genéricas demais. Por exemplo:

```xml
<data>
```

pode ser menos útil do que:

```xml
<sales_records>
```

Outro cuidado é fechar corretamente as tags. Tags abertas, duplicadas ou mal organizadas podem aumentar a confusão em vez de reduzir.

---

## 7. Resumo para revisão rápida

* XML tags organizam prompts.
* Elas separam partes distintas do conteúdo.
* São úteis como delimitadores para Claude.
* Ajudam em prompts com muito contexto.
* Ajudam a separar dados de instruções.
* Ajudam a separar código de documentação.
* Tags personalizadas são permitidas.
* Nomes específicos são melhores que nomes genéricos.
* Use `<sales_records>` em vez de `<data>`.
* Use `<my_code>` e `<docs>` para prompts de debugging.
* XML tags são mais valiosas em prompts longos e complexos.
* Elas melhoram clareza, mas não substituem boas instruções.

---

## 8. Perguntas simuladas

### 1. Para que servem XML tags em prompts?

A) Para executar código dentro da API
B) Para separar e identificar partes distintas do prompt
C) Para aumentar automaticamente o número de tokens
D) Para substituir a API key

**Resposta correta: B.**

**Explicação:** XML tags funcionam como delimitadores que ajudam Claude a entender a estrutura do prompt.

---

### 2. Quando XML tags são mais úteis?

A) Quando o prompt contém muito contexto ou tipos diferentes de conteúdo
B) Apenas quando o prompt tem uma única frase
C) Somente para gerar imagens
D) Apenas quando não há dados no prompt

**Resposta correta: A.**

**Explicação:** Tags são especialmente úteis com dados longos, documentação, código e múltiplas variáveis.

---

### 3. Qual tag é mais clara para registros de vendas?

A) `<data>`
B) `<text>`
C) `<sales_records>`
D) `<stuff>`

**Resposta correta: C.**

**Explicação:** `<sales_records>` descreve exatamente o conteúdo do bloco.

---

### 4. Em um prompt de debugging com código e documentação, qual estrutura é melhor?

A) Misturar código e documentação sem separação
B) Usar `<my_code>` para o código e `<docs>` para a documentação
C) Colocar tudo dentro de uma única tag `<data>`
D) Não informar qual parte é código

**Resposta correta: B.**

**Explicação:** Essa estrutura cria limites claros entre o código a ser depurado e a documentação de referência.

---

### 5. XML tags precisam ser tags oficiais?

A) Sim, apenas tags oficiais funcionam
B) Não, você pode criar tags personalizadas e descritivas
C) Sim, somente tags HTML padrão são aceitas
D) Não, mas Claude ignora todas elas

**Resposta correta: B.**

**Explicação:** Você pode criar tags como `<athlete_information>`, `<sales_records>`, `<my_code>` e `<docs>`.

---

### 6. O que significa usar XML tags como delimitadores?

A) Marcar onde uma seção começa e termina
B) Criptografar o conteúdo do prompt
C) Transformar texto em código executável
D) Remover a necessidade de instruções

**Resposta correta: A.**

**Explicação:** Tags de abertura e fechamento delimitam blocos de conteúdo.

---

### 7. Qual é uma limitação das XML tags?

A) Elas organizam o prompt, mas não garantem que Claude seguirá tudo perfeitamente
B) Elas impedem Claude de ler o conteúdo
C) Elas só funcionam com JSON
D) Elas tornam impossível usar variáveis interpoladas

**Resposta correta: A.**

**Explicação:** Tags ajudam na estrutura, mas ainda precisam ser combinadas com instruções claras e avaliação.

---

### 8. No exemplo dos registros de vendas, por que usar `<sales_records>`?

A) Para indicar claramente que o bloco contém dados de vendas
B) Para esconder os dados de Claude
C) Para gerar automaticamente um gráfico
D) Para impedir que Claude analise os registros

**Resposta correta: A.**

**Explicação:** A tag ajuda Claude a identificar que aquele bloco é o conjunto de registros que deve ser analisado.

---

### 9. Qual prática é recomendada ao escolher nomes de tags?

A) Usar nomes curtos e vagos sempre
B) Usar nomes específicos e descritivos
C) Usar nomes aleatórios
D) Nunca fechar as tags

**Resposta correta: B.**

**Explicação:** Tags descritivas comunicam melhor a função de cada seção.

---

### 10. Qual frase resume melhor esta técnica?

A) XML tags servem para estruturar prompts e reduzir ambiguidade entre blocos de conteúdo.
B) XML tags substituem a necessidade de prompt engineering.
C) XML tags fazem Claude armazenar memória permanente.
D) XML tags são usadas apenas para streaming.

**Resposta correta: A.**

**Explicação:** A principal função das XML tags é tornar o prompt mais organizado e compreensível para Claude.
