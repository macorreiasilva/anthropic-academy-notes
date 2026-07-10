# O que são skills?

# Resumo do conteúdo

## 1. Ideia central

A aula explica o que são **Skills no Claude Code**: pastas com instruções reutilizáveis que ajudam o Claude a executar tarefas específicas com mais precisão.

A ideia principal é evitar repetir sempre as mesmas instruções. Em vez de explicar várias vezes como revisar PRs, escrever commits, seguir padrões de documentação ou aplicar diretrizes da equipe, você cria uma **skill** uma vez. Depois, o Claude pode descobri-la e usá-la automaticamente quando a tarefa for relevante.

---

## 2. Principais conceitos

### O que é uma Skill

Uma **skill** é uma pasta que contém instruções e recursos para ensinar o Claude Code a realizar uma tarefa específica.

Cada skill possui um arquivo chamado:

```
SKILL.md
```

Esse arquivo contém:

1. **Frontmatter**, com nome e descrição da skill.
2. **Instruções**, com o conteúdo que Claude deve seguir.

Exemplo:

```
---
name: pr-review
description: Reviews pull requests for code quality. Use when reviewing PRs or checking code changes.
---

Below the frontmatter, you write the actual instructions.
```

Nesse exemplo, a skill ensina Claude a revisar pull requests com foco em qualidade de código.

---

### Como Claude decide usar uma Skill

Claude usa principalmente a **descrição da skill** para decidir se ela é relevante.

Quando você faz uma solicitação, como:

> “Review this PR”
> 

Claude compara esse pedido com as descrições das skills disponíveis. Se encontrar uma correspondência, ele ativa a skill apropriada.

Ou seja, a descrição funciona como um “gatilho semântico” para o Claude saber quando aquela skill deve ser usada.

---

### Onde as Skills ficam armazenadas

Existem dois tipos principais de skills:

#### Personal skills

Ficam no diretório pessoal do usuário:

```
~/.claude/skills
```

No Windows:

```
C:/Users/<your-user>/.claude/skills
```

Essas skills acompanham o usuário em todos os projetos.

Exemplos de uso:

- Estilo pessoal de mensagens de commit.
- Forma preferida de documentação.
- Como você gosta que o código seja explicado.

#### Project skills

Ficam dentro do repositório do projeto:

```
.claude/skills
```

Essas skills são compartilhadas com todos que clonam o repositório, pois podem ser versionadas junto com o código.

Exemplos de uso:

- Padrões de código da equipe.
- Guidelines de marca.
- Fontes, cores e padrões visuais.
- Templates de documentação do projeto.

---

### Skills vs. CLAUDE.md vs. Slash Commands

A aula compara três formas de personalizar o Claude Code.

### CLAUDE.md

O arquivo `CLAUDE.md` é carregado em **todas as conversas**.

Use quando a instrução deve estar sempre ativa.

Exemplo:

```
Sempre use TypeScript em strict mode.
```

### Skills

Skills são carregadas **sob demanda**, apenas quando combinam com a tarefa.

Isso evita ocupar o contexto desnecessariamente.

Exemplo:

Uma checklist de revisão de PR só precisa ser carregada quando você pedir uma revisão de PR.

### Slash commands

Slash commands precisam ser chamados explicitamente pelo usuário.

Exemplo hipotético:

```
/review-pr
```

Já as skills não exigem chamada manual. Claude as aplica automaticamente quando reconhece a situação.

---

## 3. Pontos importantes para certificação

- **Skills são pastas de instruções e recursos** usadas pelo Claude Code para melhorar tarefas específicas.
- Cada skill vive em um arquivo chamado **SKILL.md**.
- O arquivo `SKILL.md` deve conter um **frontmatter** com pelo menos:
    - `name`
    - `description`
- Claude usa a **description** para decidir quando ativar uma skill.
- Skills são carregadas **on demand**, ou seja, apenas quando são relevantes.
- Skills são diferentes de `CLAUDE.md`, pois `CLAUDE.md` é carregado em toda conversa.
- Skills são diferentes de slash commands, pois não precisam ser chamadas explicitamente.
- Skills pessoais ficam em:

```
~/.claude/skills
```

- Skills de projeto ficam em:

```
.claude/skills
```

- No Windows, skills pessoais ficam em:

```
C:/Users/<your-user>/.claude/skills
```

- Uma boa regra prática: se você está explicando a mesma coisa repetidamente para Claude, provavelmente isso deveria virar uma skill.

---

## 4. Termos-chave

### Skill

Conjunto de instruções e recursos que ensina Claude Code a executar melhor uma tarefa específica.

### SKILL.md

Arquivo principal de uma skill. Contém o frontmatter e as instruções da skill.

### Frontmatter

Bloco inicial em formato estruturado, delimitado por `---`, usado para definir metadados como `name` e `description`.

### Name

Nome da skill. Identifica a skill, por exemplo:

```
pr-review
```

### Description

Descrição usada por Claude para decidir quando a skill deve ser ativada.

### Personal skills

Skills pessoais do usuário, disponíveis em todos os projetos.

### Project skills

Skills específicas de um projeto, armazenadas dentro do repositório e compartilhadas com a equipe.

### CLAUDE.md

Arquivo de configuração/instruções carregado em todas as conversas com Claude Code.

### Slash commands

Comandos invocados manualmente pelo usuário. Diferem das skills porque não são ativados automaticamente.

### On demand

Carregamento sob demanda. Significa que a skill só é carregada quando Claude reconhece que ela é relevante.

---

## 5. Boas práticas

- Escreva descrições claras, pois Claude usa a `description` para decidir quando usar a skill.
- Use skills para instruções repetitivas e específicas.
- Coloque padrões pessoais em `~/.claude/skills`.
- Coloque padrões da equipe ou do projeto em `.claude/skills`.
- Versione project skills junto com o código para que toda a equipe compartilhe os mesmos padrões.
- Não coloque tudo em `CLAUDE.md`; use skills quando a instrução só for necessária em situações específicas.
- Use `CLAUDE.md` para regras globais que devem valer sempre.
- Use slash commands quando quiser uma ação explicitamente invocada pelo usuário.
- Crie skills para tarefas como:
    - Revisão de PRs.
    - Formato de commits.
    - Checklists de debugging.
    - Templates de documentação.
    - Diretrizes de marca.
    - Padrões de design.

---

## 6. Limitações, riscos e cuidados

- Se a descrição da skill for vaga, Claude pode não ativá-la no momento correto.
- Se a descrição for ampla demais, Claude pode ativar a skill em situações inadequadas.
- Skills não substituem instruções globais quando uma regra precisa valer sempre; nesses casos, `CLAUDE.md` pode ser mais adequado.
- Skills dependem da organização correta dos arquivos e diretórios.
- Project skills devem ser mantidas atualizadas, pois afetam toda a equipe.
- Instruções conflitantes entre `CLAUDE.md`, skills e prompts do usuário podem gerar comportamento confuso.
- Colocar informações demais em uma skill pode torná-la menos objetiva.
- Boas skills devem ser específicas, acionáveis e fáceis de Claude aplicar.

---

## 7. Resumo para revisão rápida

- Skills ensinam Claude Code a fazer tarefas específicas.
- Cada skill fica em um arquivo `SKILL.md`.
- O `SKILL.md` contém `name` e `description` no frontmatter.
- Claude usa a `description` para decidir quando ativar a skill.
- Skills são carregadas automaticamente quando relevantes.
- Personal skills ficam em `~/.claude/skills`.
- Project skills ficam em `.claude/skills`.
- Project skills podem ser compartilhadas via controle de versão.
- `CLAUDE.md` é carregado em toda conversa.
- Skills são carregadas apenas sob demanda.
- Slash commands precisam ser chamados manualmente.
- Use skills quando você repete as mesmas instruções várias vezes.

---

## 8. Perguntas simuladas

### Pergunta 1

O que é uma skill no Claude Code?

A. Um comando que sempre precisa ser chamado manualmente pelo usuário.

B. Uma pasta com instruções e recursos que Claude pode descobrir e usar para tarefas específicas.

C. Um arquivo obrigatório carregado em toda conversa.

D. Um plugin externo instalado via navegador.

**Resposta correta: B**

**Explicação:** Skills são pastas com instruções e recursos que Claude Code usa para lidar melhor com tarefas específicas.

---

### Pergunta 2

Qual arquivo principal define uma skill?

A. `CLAUDE.md`

B. `README.md`

C. `SKILL.md`

D. `COMMAND.md`

**Resposta correta: C**

**Explicação:** Cada skill vive em um arquivo chamado `SKILL.md`, que contém frontmatter e instruções.

---

### Pergunta 3

Qual campo Claude usa principalmente para decidir quando ativar uma skill?

A. `author`

B. `version`

C. `description`

D. `repository`

**Resposta correta: C**

**Explicação:** Claude compara a solicitação do usuário com as descrições das skills disponíveis para decidir qual usar.

---

### Pergunta 4

Onde ficam as personal skills em sistemas Unix/macOS?

A. `.claude/skills` dentro do repositório

B. `/etc/claude/skills`

C. `~/.claude/skills`

D. `~/Documents/claude/skills`

**Resposta correta: C**

**Explicação:** Skills pessoais ficam em `~/.claude/skills` e acompanham o usuário em todos os projetos.

---

### Pergunta 5

Onde ficam as project skills?

A. `~/.claude/skills`

B. `.claude/skills` dentro do repositório

C. `C:/Claude/System/skills`

D. Dentro do arquivo `CLAUDE.md`

**Resposta correta: B**

**Explicação:** Skills de projeto ficam em `.claude/skills` na raiz do repositório e podem ser compartilhadas com a equipe.

---

### Pergunta 6

Qual é a principal diferença entre skills e `CLAUDE.md`?

A. Skills são sempre carregadas; `CLAUDE.md` nunca é carregado.

B. `CLAUDE.md` é carregado em toda conversa; skills são carregadas sob demanda.

C. Skills só funcionam no Windows.

D. `CLAUDE.md` serve apenas para revisão de PR.

**Resposta correta: B**

**Explicação:** `CLAUDE.md` entra no contexto de todas as conversas, enquanto skills são ativadas apenas quando relevantes.

---

### Pergunta 7

Qual é a principal diferença entre skills e slash commands?

A. Skills precisam ser chamadas manualmente; slash commands são automáticos.

B. Skills são automáticas quando relevantes; slash commands exigem invocação explícita.

C. Slash commands ficam dentro de `SKILL.md`.

D. Skills não podem conter instruções.

**Resposta correta: B**

**Explicação:** Slash commands dependem de comando manual, enquanto skills são aplicadas automaticamente quando Claude reconhece a situação.

---

### Pergunta 8

Quando faz sentido criar uma skill?

A. Quando uma instrução é usada apenas uma vez.

B. Quando você precisa repetir a mesma orientação várias vezes para Claude.

C. Quando você quer desativar Claude Code.

D. Quando não há nenhum padrão a seguir.

**Resposta correta: B**

**Explicação:** A regra prática da aula é: se você se pega explicando a mesma coisa repetidamente, isso provavelmente deveria virar uma skill.