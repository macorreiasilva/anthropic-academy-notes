# Skills versus outras funcionalidades do Claude Code

## 1. Ideia central

A aula compara **Skills** com outros recursos do Claude Code: `CLAUDE.md`, subagents, hooks e MCP servers.

A ideia principal é que cada recurso resolve um tipo diferente de problema. Skills não substituem tudo. Elas são melhores para **conhecimento específico de uma tarefa**, carregado sob demanda. Já outros recursos servem para padrões globais, automações, isolamento de execução ou integrações externas.

O ponto central para certificação é saber **quando usar cada recurso** e entender que eles podem ser combinados.

---

## 2. Principais conceitos

### `CLAUDE.md` vs Skills

O arquivo `CLAUDE.md` é carregado em **todas as conversas**. Ele serve para instruções que devem estar sempre ativas no projeto.

Use `CLAUDE.md` para regras permanentes, como:

- Padrões gerais do projeto.
- Restrições que sempre devem ser obedecidas.
- Preferências de framework.
- Estilo de código.
- Regras como “nunca modificar o schema do banco de dados”.

Exemplo:

```
Always use TypeScript strict mode.
```

Já as **Skills** são carregadas **sob demanda**. Elas entram no contexto apenas quando Claude identifica que a solicitação combina com a descrição da skill.

Use Skills para:

- Conhecimento específico de uma tarefa.
- Procedimentos detalhados usados apenas às vezes.
- Checklists que não precisam estar em todas as conversas.
- Especialização ativada automaticamente quando relevante.

Exemplo: uma checklist de revisão de PR só precisa ser carregada quando o usuário pedir uma revisão de PR.

---

### Skills vs Subagents

Skills adicionam conhecimento à **conversa atual**. Quando uma skill é ativada, suas instruções entram no contexto existente.

Subagents funcionam de maneira diferente: eles executam tarefas em um **contexto separado e isolado**. O subagent recebe uma tarefa, trabalha de forma independente e devolve o resultado para a conversa principal.

Use Subagents quando:

- Você quer delegar uma tarefa para outro contexto de execução.
- A tarefa deve ficar isolada da conversa principal.
- O subagent precisa de acesso diferente a ferramentas.
- Você quer separar trabalho delegado do raciocínio principal.

Use Skills quando:

- Você quer melhorar o conhecimento de Claude na conversa atual.
- A especialização deve influenciar a tarefa em andamento.
- As instruções da skill são relevantes ao longo da conversa.

Em resumo: **Skills aumentam o conhecimento do Claude na conversa atual; Subagents delegam trabalho para um contexto separado.**

---

### Skills vs Hooks

Hooks são acionados por **eventos**.

Exemplos de eventos:

- Claude salva um arquivo.
- Claude chama determinada ferramenta.
- Uma ação específica acontece no ambiente.

Um hook pode, por exemplo:

- Rodar um linter sempre que Claude salvar um arquivo.
- Validar uma entrada antes de uma chamada de ferramenta.
- Executar uma verificação automática após uma modificação.

Skills são acionadas por **requisições**. Elas ativam quando Claude reconhece que o pedido do usuário combina com a descrição da skill.

Use Hooks para:

- Operações automáticas em eventos.
- Validação antes de chamadas específicas de ferramentas.
- Efeitos colaterais automatizados das ações de Claude.
- Rotinas que devem acontecer sempre que certo evento ocorrer.

Use Skills para:

- Conhecimento que orienta como Claude deve responder.
- Guidelines que afetam o raciocínio.
- Procedimentos que dependem do tipo de solicitação feita pelo usuário.

Em resumo: **Hooks são event-driven; Skills são request-driven.**

---

### Skills vs MCP servers

MCP servers pertencem a outra categoria.

Eles fornecem **ferramentas externas e integrações** para Claude Code. Enquanto skills são instruções e conhecimento, MCP servers conectam Claude a sistemas, APIs, bases de dados, ferramentas ou serviços externos.

Use MCP servers quando precisar de:

- Integração com ferramentas externas.
- Acesso a sistemas fora do Claude Code.
- APIs, bancos de dados, serviços ou plataformas conectadas.
- Expansão das capacidades práticas de Claude por meio de ferramentas.

Em resumo: **Skills dizem como Claude deve pensar ou agir em uma tarefa; MCP servers fornecem ferramentas e integrações externas.**

---

### Combinando recursos

A aula reforça que não se deve forçar tudo dentro de skills. Cada recurso tem uma especialidade.

Uma configuração típica pode incluir:

```
CLAUDE.md — padrões sempre ativos do projeto
Skills — conhecimento específico carregado sob demanda
Hooks — automações acionadas por eventos
Subagents — execução isolada de tarefas delegadas
MCP servers — ferramentas e integrações externas
```

A melhor prática é combinar os recursos conforme a necessidade.

---

## 3. Pontos importantes para certificação

- `CLAUDE.md` é carregado em **todas as conversas**.
- Skills são carregadas **sob demanda**, quando a solicitação combina com a skill.
- Use `CLAUDE.md` para padrões globais e sempre ativos.
- Use Skills para expertise específica de tarefas.
- Skills adicionam conhecimento ao contexto atual.
- Subagents executam tarefas em um contexto separado e isolado.
- Use Subagents para delegação de trabalho e isolamento.
- Hooks são acionados por eventos, como salvar arquivos ou chamar ferramentas.
- Skills são acionadas por solicitações do usuário.
- Hooks são úteis para automações e validações recorrentes.
- MCP servers fornecem ferramentas externas e integrações.
- MCP servers não são o mesmo tipo de recurso que skills.
- Os recursos podem e devem ser combinados.
- A escolha correta depende do tipo de problema:
    - Regra global: `CLAUDE.md`
    - Expertise específica: Skill
    - Trabalho delegado e isolado: Subagent
    - Automação por evento: Hook
    - Integração externa: MCP server

---

## 4. Termos-chave

### Skill

Recurso que adiciona conhecimento específico à conversa atual quando a solicitação do usuário é relevante.

### `CLAUDE.md`

Arquivo com instruções sempre carregadas em todas as conversas de um projeto.

### Subagent

Agente que executa uma tarefa em contexto separado e isolado, retornando depois o resultado.

### Hook

Automação acionada por eventos específicos, como salvar arquivo ou chamar uma ferramenta.

### MCP server

Servidor que fornece ferramentas externas e integrações para Claude Code.

### Always-on instructions

Instruções que estão sempre ativas. Normalmente pertencem ao `CLAUDE.md`.

### On-demand loading

Carregamento sob demanda. Característica das skills, que só entram no contexto quando relevantes.

### Event-driven

Acionado por eventos. Característica dos hooks.

### Request-driven

Acionado pelo pedido do usuário. Característica das skills.

### Isolated execution context

Contexto separado de execução, usado por subagents para trabalhar de forma independente da conversa principal.

---

## 5. Boas práticas

- Use `CLAUDE.md` apenas para instruções que sempre devem valer.
- Não coloque checklists ou procedimentos raramente usados no `CLAUDE.md`; transforme-os em skills.
- Use skills para tarefas recorrentes, mas específicas.
- Use subagents quando quiser delegar trabalho sem misturar tudo na conversa principal.
- Use hooks para automações que devem ocorrer em eventos previsíveis.
- Use MCP servers quando precisar conectar Claude a sistemas externos.
- Combine recursos em vez de tentar resolver tudo com uma única abordagem.
- Revise seu `CLAUDE.md` periodicamente para identificar conteúdo que poderia virar skill.
- Escolha o recurso com base no gatilho:
    - Sempre ativo: `CLAUDE.md`
    - Pedido específico: Skill
    - Evento do sistema: Hook
    - Delegação isolada: Subagent
    - Ferramenta externa: MCP

---

## 6. Limitações, riscos e cuidados

- Colocar informações demais no `CLAUDE.md` pode poluir o contexto de todas as conversas.
- Usar skills para regras que sempre devem valer pode ser arriscado, porque skills só carregam quando ativadas.
- Usar hooks para conhecimento ou guidelines não é adequado, pois hooks são automações por evento.
- Usar subagents quando o conhecimento deveria permanecer na conversa principal pode fragmentar o fluxo.
- Usar skills como substitutas de MCP servers não funciona quando o problema exige integração externa real.
- Escolher o recurso errado pode gerar complexidade desnecessária.
- Skills são úteis, mas não devem ser usadas para tudo.
- `CLAUDE.md`, skills, hooks, subagents e MCP servers podem interagir; é importante evitar instruções conflitantes entre eles.

---

## 7. Resumo para revisão rápida

- `CLAUDE.md` carrega em toda conversa.
- Skills carregam sob demanda.
- Use `CLAUDE.md` para padrões globais do projeto.
- Use Skills para conhecimento específico de tarefas.
- Skills entram no contexto atual.
- Subagents trabalham em contexto separado.
- Use Subagents para tarefas delegadas e isoladas.
- Hooks são acionados por eventos.
- Skills são acionadas por pedidos do usuário.
- MCP servers fornecem ferramentas externas e integrações.
- Não force tudo em skills.
- Combine os recursos conforme o problema.
- Regra prática:
    - Sempre ativo → `CLAUDE.md`
    - Tarefa específica → Skill
    - Delegação isolada → Subagent
    - Evento automático → Hook
    - Integração externa → MCP server

---

## 8. Perguntas simuladas

### Pergunta 1

Qual é a principal diferença entre `CLAUDE.md` e Skills?

A. `CLAUDE.md` só funciona com plugins, enquanto Skills funcionam offline.

B. `CLAUDE.md` é carregado em toda conversa, enquanto Skills carregam sob demanda.

C. Skills são sempre carregadas, enquanto `CLAUDE.md` precisa ser chamado manualmente.

D. `CLAUDE.md` é usado apenas para MCP servers.

**Resposta correta: B**

**Explicação:** `CLAUDE.md` contém instruções sempre ativas. Skills são ativadas apenas quando relevantes para a solicitação.

---

### Pergunta 2

Quando é mais adequado usar `CLAUDE.md`?

A. Para uma checklist detalhada de revisão de PR usada ocasionalmente.

B. Para padrões globais do projeto que sempre devem ser aplicados.

C. Para executar uma tarefa em contexto isolado.

D. Para integrar Claude a uma API externa.

**Resposta correta: B**

**Explicação:** `CLAUDE.md` é ideal para regras e padrões que precisam estar presentes em todas as conversas.

---

### Pergunta 3

Quando é mais adequado usar uma Skill?

A. Quando o conhecimento é específico de uma tarefa e só é relevante às vezes.

B. Quando uma ação deve ocorrer sempre que um arquivo for salvo.

C. Quando é necessário conectar Claude a uma ferramenta externa.

D. Quando a tarefa precisa ser executada em contexto completamente separado.

**Resposta correta: A**

**Explicação:** Skills são indicadas para expertise específica carregada sob demanda.

---

### Pergunta 4

Qual é a principal característica de um Subagent?

A. Ele é carregado em todas as conversas.

B. Ele executa tarefas em um contexto separado e isolado.

C. Ele substitui o arquivo `CLAUDE.md`.

D. Ele é acionado sempre que um arquivo é salvo.

**Resposta correta: B**

**Explicação:** Subagents recebem uma tarefa, trabalham em contexto separado e retornam resultados à conversa principal.

---

### Pergunta 5

Qual é a diferença entre Skills e Subagents?

A. Skills adicionam conhecimento à conversa atual; Subagents executam trabalho em contexto isolado.

B. Skills só funcionam com MCP; Subagents só funcionam com hooks.

C. Skills são event-driven; Subagents são always-on.

D. Não há diferença prática.

**Resposta correta: A**

**Explicação:** Skills enriquecem o contexto atual. Subagents delegam trabalho para outro contexto de execução.

---

### Pergunta 6

O que são Hooks no Claude Code?

A. Instruções carregadas sob demanda com base na descrição.

B. Automações acionadas por eventos específicos.

C. Arquivos de documentação global sempre ativos.

D. Servidores externos de integração.

**Resposta correta: B**

**Explicação:** Hooks disparam quando certos eventos acontecem, como salvar arquivos ou chamar ferramentas.

---

### Pergunta 7

Qual exemplo representa melhor o uso de um Hook?

A. Carregar uma checklist de revisão quando o usuário pede review.

B. Rodar um linter toda vez que Claude salva um arquivo.

C. Explicar a arquitetura do projeto em linguagem simples.

D. Criar uma skill de documentação.

**Resposta correta: B**

**Explicação:** Hooks são usados para operações automáticas baseadas em eventos.

---

### Pergunta 8

Qual é a diferença entre Hooks e Skills?

A. Hooks são request-driven; Skills são event-driven.

B. Hooks são event-driven; Skills são request-driven.

C. Ambos são sempre carregados em todas as conversas.

D. Ambos servem apenas para integrações externas.

**Resposta correta: B**

**Explicação:** Hooks disparam por eventos. Skills ativam com base no pedido do usuário.

---

### Pergunta 9

Para que servem MCP servers?

A. Para armazenar instruções em `SKILL.md`.

B. Para fornecer ferramentas externas e integrações.

C. Para substituir todas as skills.

D. Para carregar padrões globais em todas as conversas.

**Resposta correta: B**

**Explicação:** MCP servers conectam Claude a ferramentas externas, APIs, sistemas e integrações.

---

### Pergunta 10

Qual combinação representa uma configuração típica no Claude Code?

A. Apenas Skills para tudo.

B. Apenas Hooks, sem `CLAUDE.md`.

C. `CLAUDE.md` para padrões globais, Skills para expertise específica, Hooks para eventos, Subagents para delegação e MCP para integrações.

D. MCP servers para substituir Subagents e Skills.

**Resposta correta: C**

**Explicação:** Cada recurso tem uma especialidade e pode ser combinado com os outros.

---

### Pergunta 11

Se uma instrução deve valer em todas as conversas do projeto, onde ela deve ficar preferencialmente?

A. Em uma Skill.

B. Em um Hook.

C. Em `CLAUDE.md`.

D. Em um Subagent.

**Resposta correta: C**

**Explicação:** Instruções sempre ativas pertencem ao `CLAUDE.md`.

---

### Pergunta 12

Se uma checklist detalhada só é usada quando o usuário pede revisão de PR, qual recurso é mais adequado?

A. Skill.

B. `CLAUDE.md`.

C. MCP server.

D. Hook.

**Resposta correta: A**

**Explicação:** Uma checklist específica de revisão de PR é conhecimento task-specific e deve carregar sob demanda como skill.