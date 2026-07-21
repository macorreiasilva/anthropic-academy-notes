# Visão geral do projeto

## 1. Ideia central

O conteúdo apresenta um **projeto prático de uso de ferramentas com Claude**: criar um sistema em que Claude consiga interpretar pedidos de lembrete em linguagem natural e acionar ferramentas para calcular datas e registrar lembretes.

Exemplo de objetivo:

```text
Set a reminder for my doctor's appointment. It's a week from Thursday.
```

Resposta esperada:

```text
OK, I will remind you.
```

A ideia principal é mostrar que, quando Claude tem limitações naturais, podemos **estender suas capacidades usando tools**, em vez de tentar resolver tudo apenas com prompt.

---

## 2. Principais conceitos

### Projeto de lembretes

O projeto consiste em ensinar Claude a lidar com pedidos de lembrete para datas futuras.

O usuário pode pedir algo como:

```text
Remind me in a week.
```

ou:

```text
Set a reminder for my doctor's appointment. It's a week from Thursday.
```

Para responder corretamente, Claude precisa entender a intenção do usuário, calcular a data futura e registrar o lembrete no sistema.

---

### Por que isso é desafiador

Embora pareça simples, o projeto revela três limitações importantes.

#### 1. Consciência limitada de tempo

Claude pode saber a data atual dentro do contexto da conversa ou da chamada, mas pode não saber o **horário exato atual**.

Para lembretes, isso é importante porque “em uma hora”, “amanhã de manhã” ou “na próxima quinta” dependem de uma referência temporal precisa.

---

#### 2. Problemas com cálculo de datas

Claude pode cometer erros em cálculos temporais, especialmente quando precisa somar dias, semanas ou interpretar expressões como:

* “a week from Thursday”;
* “in 10 days”;
* “next Friday”;
* “two months from now”;
* “tomorrow afternoon”.

Mesmo que Claude consiga raciocinar sobre datas, cálculos de tempo devem ser confiáveis em uma aplicação real.

Por isso, é melhor usar uma ferramenta programática para calcular datas.

---

#### 3. Claude não tem capacidade nativa de criar lembretes

Claude não possui, por padrão, um mecanismo interno para realmente salvar um lembrete no sistema.

Ele pode dizer “vou te lembrar”, mas sem uma ferramenta conectada a um backend, calendário, banco de dados ou sistema de notificações, nenhum lembrete real será criado.

Esse é um exemplo clássico de diferença entre **responder sobre uma tarefa** e **executar uma tarefa**.

---

### Tools como solução

A solução é criar ferramentas customizadas que Claude pode solicitar quando precisar.

O projeto propõe três ferramentas:

| Ferramenta                    | Função                                               |
| ----------------------------- | ---------------------------------------------------- |
| **Get current date time**     | Obter a data e hora atual com precisão               |
| **Add duration to date time** | Somar uma duração a uma data/hora de forma confiável |
| **Set a reminder**            | Registrar o lembrete no sistema                      |

Essas tools cobrem as lacunas de Claude:

* falta de tempo exato;
* risco de erro em cálculo temporal;
* ausência de mecanismo nativo de lembrete.

---

### Fluxo esperado do sistema

O fluxo do projeto pode ser entendido assim:

```text
Usuário pede um lembrete
↓
Claude identifica que precisa saber a data/hora atual
↓
Claude usa a tool de data/hora atual
↓
Claude calcula a data futura usando a tool de soma de duração
↓
Claude chama a tool para criar o lembrete
↓
Claude responde ao usuário confirmando
```

Exemplo:

```text
Usuário: Remind me about my doctor's appointment a week from Thursday.
Claude: preciso calcular a data exata.
Tool 1: obtém data/hora atual.
Tool 2: soma a duração ou resolve a data futura.
Tool 3: cria o lembrete.
Claude: OK, I will remind you.
```

---

## 3. Pontos importantes para certificação

* Tools são usadas para estender capacidades de Claude.
* Claude não deve depender apenas de raciocínio interno para tarefas que exigem precisão operacional.
* Cálculos de datas podem ser um ponto fraco de modelos de linguagem.
* Lembretes exigem execução real em algum sistema externo.
* Claude não cria lembretes por conta própria sem uma tool ou integração.
* O servidor/aplicação é responsável por implementar as ferramentas.
* Cada tool deve resolver uma limitação específica.
* O projeto usa três tools: obter data/hora atual, somar duração e criar lembrete.
* Tools permitem transformar linguagem natural em ações estruturadas.
* A abordagem correta é: identificar a lacuna do modelo e criar uma ferramenta para preenchê-la.
* Esse projeto demonstra o princípio de **não tentar resolver limitações do modelo apenas com prompt** quando uma tool pode resolver melhor.

---

## 4. Termos-chave

**Tool use**
Uso de ferramentas externas para permitir que Claude acesse dados ou execute ações fora do modelo.

**Custom tool**
Ferramenta criada especificamente pela aplicação para resolver uma necessidade do sistema.

**Current date time**
Data e hora atual, usada como referência para calcular lembretes futuros.

**Date calculation**
Cálculo envolvendo datas, horários, durações e intervalos.

**Duration**
Período de tempo, como uma semana, três dias ou duas horas.

**Reminder**
Alerta ou notificação programada para uma data futura.

**Natural language request**
Pedido feito em linguagem comum, como “remind me next Thursday”.

**Backend/server**
Sistema responsável por executar código, chamar ferramentas e persistir lembretes.

**Capability gap**
Lacuna entre o que Claude consegue fazer naturalmente e o que a aplicação precisa que ele faça.

**Tool calling**
Processo em que Claude solicita a execução de uma ferramenta com parâmetros específicos.

---

## 5. Boas práticas

Crie tools para tarefas que exigem precisão, dados externos ou execução real.

Para lembretes, é melhor separar responsabilidades em ferramentas pequenas:

```text
get_current_datetime
add_duration_to_datetime
set_reminder
```

Essa separação torna o sistema mais fácil de testar, depurar e manter.

Também é uma boa prática começar pela ferramenta mais simples e evoluir gradualmente. No projeto, isso significa começar por **obter a data e hora atual** antes de implementar cálculo de duração e criação de lembrete.

Outra boa prática é não confiar apenas no raciocínio do modelo para cálculos de data. Use código determinístico para operações temporais.

Além disso, confirme ações importantes com clareza. Depois de criar o lembrete, Claude deve responder de forma simples, por exemplo:

```text
OK, I will remind you.
```

---

## 6. Limitações, riscos e cuidados

O principal risco é Claude calcular uma data errada se o sistema depender apenas do raciocínio do modelo.

Por exemplo, expressões como “a week from Thursday” podem ser ambíguas dependendo da data atual e da interpretação de “from Thursday”.

Outro risco é Claude afirmar que criou um lembrete quando nenhuma ação real foi executada. Isso deve ser evitado. A confirmação só deve acontecer depois que a tool `set_reminder` for chamada com sucesso.

Também há cuidados de privacidade e segurança. Lembretes podem conter informações sensíveis, como consultas médicas, compromissos pessoais ou dados profissionais. O sistema deve armazenar e processar essas informações com segurança.

Além disso, é importante lidar com ambiguidades. Se o usuário disser “remind me tomorrow”, talvez seja necessário perguntar o horário, dependendo do comportamento desejado da aplicação.

---

## 7. Resumo para revisão rápida

* O projeto ensina Claude a criar lembretes.
* A tarefa parece simples, mas exige tools.
* Claude pode não saber o horário exato.
* Claude pode errar cálculos de data.
* Claude não cria lembretes nativamente.
* Tools resolvem essas limitações.
* Tool 1: obter data/hora atual.
* Tool 2: somar duração a uma data/hora.
* Tool 3: criar o lembrete.
* O servidor executa as tools.
* Claude usa linguagem natural para decidir quando pedir ferramentas.
* A resposta final só deve confirmar o lembrete após a execução da tool.
* O princípio central é: use tools para preencher lacunas reais do modelo.

---

## 8. Perguntas simuladas

### 1. Qual é o objetivo do projeto apresentado?

A) Ensinar Claude a gerar imagens
B) Ensinar Claude a criar lembretes para datas futuras usando tools
C) Ensinar Claude a treinar um novo modelo
D) Ensinar Claude a esconder API keys

**Resposta correta: B.**

**Explicação:** O projeto mostra como usar ferramentas para permitir que Claude interprete pedidos de lembrete e acione ações externas.

---

### 2. Por que criar lembretes é desafiador para Claude?

A) Porque Claude nunca entende linguagem natural
B) Porque exige tempo exato, cálculo de datas e uma ação externa real
C) Porque lembretes só podem ser criados em JSON
D) Porque Claude não aceita datas em prompts

**Resposta correta: B.**

**Explicação:** O sistema precisa saber o momento atual, calcular datas futuras e registrar o lembrete em algum sistema.

---

### 3. Qual limitação está relacionada a “Claude might know the current date, but not the exact time”?

A) Consciência limitada de tempo
B) Falha de streaming
C) Erro de markdown
D) Falta de XML tags

**Resposta correta: A.**

**Explicação:** Claude pode não ter acesso preciso ao horário atual, o que é essencial para lembretes.

---

### 4. Por que é útil ter uma tool para adicionar duração a uma data?

A) Porque cálculos temporais podem ser feitos de forma mais confiável por código
B) Porque Claude não consegue receber texto
C) Porque isso substitui a API key
D) Porque tools só funcionam com datas

**Resposta correta: A.**

**Explicação:** Operações de data e hora exigem precisão, e código determinístico é mais confiável que raciocínio textual do modelo.

---

### 5. Qual tool realmente registra o lembrete?

A) Get current date time
B) Add duration to date time
C) Set a reminder
D) Generate prompt

**Resposta correta: C.**

**Explicação:** A tool `set_reminder` é responsável por criar o lembrete no sistema.

---

### 6. O que Claude deve fazer quando recebe “remind me in a week”?

A) Inventar uma data sem referência
B) Usar ferramentas para obter data atual, calcular a data futura e criar o lembrete
C) Responder que lembretes nunca são possíveis
D) Ignorar o pedido

**Resposta correta: B.**

**Explicação:** O fluxo correto envolve usar tools para resolver as partes que exigem precisão e ação externa.

---

### 7. Qual princípio geral o projeto demonstra?

A) Sempre resolver limitações do modelo com prompts mais longos
B) Estender capacidades do modelo com tools quando houver limitações reais
C) Evitar qualquer integração externa
D) Usar apenas respostas estáticas

**Resposta correta: B.**

**Explicação:** O projeto mostra que tools são a forma adequada de preencher lacunas de capacidade, como cálculo preciso e execução de ações.

---

### 8. Quem executa o código das tools?

A) O servidor da aplicação
B) O usuário manualmente em papel
C) Claude diretamente, sem controle
D) O arquivo README

**Resposta correta: A.**

**Explicação:** Claude solicita a ferramenta, mas o servidor executa o código e retorna o resultado.

---

### 9. Qual é um risco se Claude confirmar um lembrete sem chamar uma tool?

A) O usuário pode acreditar que o lembrete foi criado quando não foi
B) O prompt fica mais curto
C) A resposta vira XML automaticamente
D) O modelo deixa de funcionar

**Resposta correta: A.**

**Explicação:** A aplicação deve confirmar ações apenas depois de executá-las de verdade.

---

### 10. Qual conjunto de tools melhor corresponde ao projeto?

A) Translate text, generate image, summarize document
B) Get current date time, add duration to date time, set a reminder
C) Create API key, delete server, train model
D) Validate JSON, compile regex, stream response

**Resposta correta: B.**

**Explicação:** Essas três ferramentas resolvem as limitações específicas do sistema de lembretes.
