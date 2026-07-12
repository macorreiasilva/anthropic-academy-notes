# Avaliação de prompt

```
006_controlling_output.ipynb
```

## 1. Ideia central

O conteúdo explica a diferença entre **prompt engineering** e **prompt evaluation**.

A ideia central é que **escrever um bom prompt não é suficiente** para criar aplicações confiáveis com Claude. Depois de criar o prompt, é necessário **avaliar sistematicamente se ele funciona bem em diferentes cenários**.

Em resumo:

* **Prompt engineering** ajuda a escrever prompts melhores.
* **Prompt evaluation** ajuda a medir se esses prompts realmente funcionam.

Para aplicações sérias em produção, a avaliação de prompts é essencial porque usuários reais costumam fornecer entradas inesperadas.

---

## 2. Principais conceitos

### Prompt engineering

**Prompt engineering** é o conjunto de técnicas usadas para criar prompts mais claros, eficazes e controlados.

Exemplos citados no conteúdo:

* **multishot prompting**;
* estruturação com **XML tags**;
* outras boas práticas de escrita de prompts.

Essas técnicas ajudam Claude a entender:

* o que o usuário quer;
* qual formato de resposta deve usar;
* quais restrições seguir;
* qual papel ou comportamento assumir.

Prompt engineering responde à pergunta:

**“Como escrevo um prompt melhor?”**

---

### Prompt evaluation

**Prompt evaluation** é o processo de testar e medir a qualidade de um prompt de forma mais sistemática.

Em vez de apenas “achar” que o prompt está bom, você cria formas de avaliar se ele funciona corretamente.

A avaliação pode incluir:

* testar contra respostas esperadas;
* comparar versões diferentes do mesmo prompt;
* revisar saídas em busca de erros;
* medir desempenho em vários casos de teste.

Prompt evaluation responde à pergunta:

**“Como sei que esse prompt funciona bem?”**

---

### Diferença entre escrever e avaliar prompts

A diferença principal é:

| Conceito               | Foco                                      |
| ---------------------- | ----------------------------------------- |
| **Prompt engineering** | Criar prompts melhores                    |
| **Prompt evaluation**  | Medir se os prompts funcionam bem         |
| **Prompt iteration**   | Melhorar o prompt com base nos resultados |

Um erro comum é investir muito em escrever o prompt, mas pouco em testá-lo.

---

### Três caminhos depois de escrever um prompt

Depois de criar um prompt, o conteúdo apresenta três opções comuns.

### Opção 1: testar uma vez e considerar suficiente

Essa é a abordagem mais frágil.

Você testa o prompt uma única vez, ele parece funcionar, e então decide usá-lo.

O risco é alto porque um único teste não representa a diversidade de entradas reais que usuários podem enviar.

---

### Opção 2: testar algumas vezes e ajustar casos específicos

Essa abordagem é melhor que a primeira, mas ainda limitada.

Você testa o prompt com alguns exemplos, encontra um ou dois problemas e faz ajustes.

O problema é que usuários reais ainda podem fornecer entradas muito diferentes daquelas que você testou.

Essa abordagem pode dar uma falsa sensação de segurança.

---

### Opção 3: usar um pipeline de avaliação

Essa é a abordagem recomendada para aplicações confiáveis.

Você cria um conjunto de testes e executa o prompt contra vários casos. Depois, mede os resultados com métricas ou critérios definidos.

Essa abordagem exige mais trabalho e pode gerar mais custo, mas oferece muito mais confiança.

Benefícios:

* identificar falhas antes da produção;
* comparar versões de prompts objetivamente;
* melhorar prompts com base em dados;
* construir aplicações de IA mais robustas.

---

### Armadilhas comuns em testes de prompt

O conteúdo destaca que muitos engenheiros caem nas opções 1 e 2.

Isso acontece porque parece natural testar rapidamente um prompt e assumir que ele está bom.

O problema é que, em produção, usuários interagem com o sistema de formas difíceis de prever.

Um prompt que funciona em testes simples pode falhar quando recebe:

* entradas ambíguas;
* perguntas incompletas;
* instruções contraditórias;
* casos extremos;
* formatos inesperados;
* tentativas de burlar o comportamento esperado.

---

### Abordagem evaluation-first

A abordagem **evaluation-first** significa tratar avaliação como parte central do desenvolvimento de prompts, não como algo opcional.

Em vez de depender de intuição, você usa dados.

O processo pode ser entendido assim:

1. Escreva uma primeira versão do prompt.
2. Rode o prompt em uma base de testes.
3. Compare as respostas com critérios esperados.
4. Identifique falhas.
5. Ajuste o prompt.
6. Rode a avaliação novamente.
7. Compare a nova versão com a anterior.

Essa abordagem torna o processo de melhoria mais objetivo.

---

## 3. Pontos importantes para certificação

* **Prompt engineering** e **prompt evaluation** são conceitos diferentes.
* Prompt engineering foca em **criar prompts melhores**.
* Prompt evaluation foca em **medir a eficácia dos prompts**.
* Testar um prompt apenas uma vez é arriscado.
* Testar apenas alguns casos também pode ser insuficiente.
* Usuários reais fornecem entradas inesperadas.
* Um prompt que parece bom em testes simples pode falhar em produção.
* Avaliação sistemática aumenta a confiabilidade da aplicação.
* Um pipeline de avaliação permite comparar versões de prompts.
* Avaliações ajudam a encontrar erros antes que usuários encontrem.
* A abordagem baseada em métricas é mais confiável que julgamento subjetivo.
* Avaliação exige mais trabalho e custo, mas reduz riscos em produção.

---

## 4. Termos-chave

**Prompt engineering**
Conjunto de técnicas para escrever prompts mais claros, específicos e eficazes.

**Prompt evaluation**
Processo de medir o desempenho de prompts usando testes, critérios e métricas.

**Multishot prompting**
Técnica em que vários exemplos são incluídos no prompt para orientar o comportamento do modelo.

**XML tags**
Tags usadas para estruturar partes do prompt, como contexto, instruções, exemplos ou formato de saída.

**Evaluation pipeline**
Fluxo automatizado para testar prompts contra vários casos e medir resultados.

**Expected answers**
Respostas esperadas usadas como referência para comparar a saída do modelo.

**Prompt versioning**
Prática de manter diferentes versões de um prompt para comparação e melhoria.

**Edge case**
Caso incomum ou extremo que pode fazer o prompt falhar.

**Production**
Ambiente real em que usuários finais interagem com a aplicação.

**Objective metrics**
Métricas mensuráveis usadas para avaliar desempenho de forma menos subjetiva.

**Iteration**
Processo de melhorar o prompt em ciclos sucessivos de teste e ajuste.

---

## 5. Boas práticas

* Não considerar um prompt pronto após apenas um teste.
* Criar conjuntos de exemplos para avaliar o comportamento do prompt.
* Testar entradas comuns e também casos extremos.
* Comparar diferentes versões do mesmo prompt.
* Usar critérios claros para julgar se a resposta está correta.
* Automatizar a avaliação sempre que possível.
* Iterar com base em resultados mensuráveis, não apenas em opinião.
* Avaliar prompts antes de colocá-los em produção.
* Incluir casos de uso reais ou próximos da realidade nos testes.
* Revisar saídas para encontrar erros, inconsistências e falhas de formato.

---

## 6. Limitações, riscos e cuidados

O maior risco é assumir que um prompt está bom porque funcionou em poucos exemplos.

Isso pode levar a falhas em produção, especialmente quando usuários fornecem entradas inesperadas.

Outro risco é avaliar de forma subjetiva demais. Se a equipe apenas “olha a resposta” e decide se gostou, pode ser difícil comparar versões ou saber se houve melhoria real.

Também há um trade-off: pipelines de avaliação exigem mais tempo, infraestrutura e custo. Porém, para aplicações importantes, esse investimento reduz riscos e aumenta a confiabilidade.

É importante lembrar que nenhuma avaliação cobre todos os casos possíveis. Ainda assim, uma boa avaliação cobre mais cenários do que testes manuais simples.

---

## 7. Resumo para revisão rápida

* Escrever um prompt bom é só o começo.
* **Prompt engineering** ajuda a criar prompts melhores.
* **Prompt evaluation** mede se o prompt funciona bem.
* Testar uma vez é arriscado.
* Testar alguns exemplos é melhor, mas ainda limitado.
* Usuários reais geram entradas inesperadas.
* Avaliação sistemática aumenta confiança.
* Um pipeline de avaliação compara versões do prompt.
* Métricas objetivas ajudam a melhorar com mais segurança.
* A melhor abordagem é iterar: testar, medir, ajustar e testar novamente.

---

## 8. Perguntas simuladas

### 1. Qual é a principal diferença entre prompt engineering e prompt evaluation?

A) Prompt engineering mede custo; prompt evaluation escolhe o modelo
B) Prompt engineering cria prompts melhores; prompt evaluation mede se eles funcionam bem
C) Prompt engineering só serve para JSON; prompt evaluation só serve para imagens
D) Não há diferença entre os dois conceitos

**Resposta correta: B.**

**Explicação:** Prompt engineering é focado em técnicas de escrita de prompts. Prompt evaluation é focado em testar e medir a eficácia desses prompts.

---

### 2. Qual das opções representa uma técnica de prompt engineering citada no conteúdo?

A) Backup automático de banco de dados
B) Multishot prompting
C) Criação de API key
D) Streaming de resposta

**Resposta correta: B.**

**Explicação:** Multishot prompting é uma técnica de engenharia de prompts que usa múltiplos exemplos para orientar o modelo.

---

### 3. O que é prompt evaluation?

A) O processo de medir a eficácia de prompts por meio de testes
B) O processo de instalar o SDK da Anthropic
C) O ato de enviar uma API key ao Claude
D) A geração automática de modelos novos

**Resposta correta: A.**

**Explicação:** Prompt evaluation envolve testar prompts, comparar resultados e revisar erros.

---

### 4. Por que testar um prompt apenas uma vez é arriscado?

A) Porque Claude nunca responde na primeira tentativa
B) Porque um único teste não cobre entradas inesperadas de usuários reais
C) Porque prompts só funcionam depois de dez execuções
D) Porque a API bloqueia prompts testados uma vez

**Resposta correta: B.**

**Explicação:** Usuários podem fornecer entradas muito diferentes do teste inicial, fazendo o prompt falhar em produção.

---

### 5. Qual é a principal vantagem de um pipeline de avaliação?

A) Eliminar completamente todos os erros possíveis
B) Medir o desempenho do prompt em vários casos e comparar versões objetivamente
C) Evitar o uso de mensagens do usuário
D) Fazer Claude armazenar memória permanente

**Resposta correta: B.**

**Explicação:** Um pipeline de avaliação permite testar o prompt sistematicamente e usar métricas para orientar melhorias.

---

### 6. O que geralmente acontece quando uma aplicação com prompt pouco testado vai para produção?

A) Usuários podem fornecer entradas inesperadas que quebram o comportamento esperado
B) O modelo automaticamente corrige todos os erros
C) A API impede qualquer resposta incorreta
D) A temperature sempre vira 0 automaticamente

**Resposta correta: A.**

**Explicação:** Em produção, a variedade de entradas é muito maior do que em testes manuais simples.

---

### 7. Qual opção é mais recomendada para aplicações sérias?

A) Testar o prompt uma vez e publicar
B) Testar o prompt apenas com um caso feliz
C) Rodar o prompt em um pipeline de avaliação e iterar com base em métricas
D) Nunca modificar o prompt depois da primeira versão

**Resposta correta: C.**

**Explicação:** A avaliação sistemática aumenta a confiabilidade e permite melhorias baseadas em dados.

---

### 8. O que significa iterar em um prompt?

A) Apagar todas as versões anteriores sem testar
B) Melhorar o prompt em ciclos de teste, análise e ajuste
C) Usar sempre o mesmo prompt sem mudanças
D) Trocar a API key a cada execução

**Resposta correta: B.**

**Explicação:** Iteração envolve revisar resultados, ajustar o prompt e testar novamente.

---

### 9. Qual é uma limitação da avaliação de prompts?

A) Ela não ajuda em nada
B) Ela exige mais tempo, infraestrutura e possivelmente custo
C) Ela impede o uso de XML tags
D) Ela só funciona para prompts de tradução

**Resposta correta: B.**

**Explicação:** Avaliar sistematicamente dá mais confiança, mas exige investimento maior do que testes manuais simples.

---

### 10. Qual é o objetivo principal da abordagem evaluation-first?

A) Encontrar problemas durante o desenvolvimento, antes que usuários os encontrem em produção
B) Esconder erros dos usuários sem corrigi-los
C) Fazer o modelo sempre responder com JSON
D) Remover a necessidade de prompts

**Resposta correta: A.**

**Explicação:** A abordagem evaluation-first busca identificar falhas cedo, comparar versões e construir aplicações de IA mais robustas.
