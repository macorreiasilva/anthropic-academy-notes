# Criando sua primeira skill

## 1. Ideia central

A aula explica como **criar a primeira skill no Claude Code** e como o Claude descobre, carrega e usa essas skills.

A ideia principal é que uma skill é uma **pasta com um arquivo `SKILL.md`**, contendo metadados e instruções. O Claude Code carrega inicialmente apenas o **nome** e a **descrição** da skill. Depois, quando o usuário faz um pedido compatível com a descrição, Claude pode ativar essa skill e seguir suas instruções.

O exemplo usado na aula é a criação de uma skill pessoal chamada `pr-description`, usada para escrever descrições de pull requests em um formato consistente.

---

## 2. Principais conceitos

### Estrutura básica de uma skill

Uma skill é um diretório que contém um arquivo obrigatório:

```
SKILL.md
```

Esse arquivo possui duas partes principais:

1. **Frontmatter**, com metadados.
2. **Instruções**, com o comportamento esperado.

Exemplo:

```
---
name: pr-description
description: Writes pull request descriptions. Use when creating a PR, writing a PR, or when the user asks to summarize changes for a pull request.
---

When writing a PR description:

1. Run `git diff main...HEAD` to see all changes on this branch
2. Write a description following this format:
...
```

O `name` identifica a skill.

A `description` informa quando Claude deve usá-la.

O conteúdo abaixo do frontmatter contém as instruções que Claude seguirá.

---

### Criando uma skill pessoal

A aula mostra como criar uma skill pessoal para descrições de PR.

Primeiro, cria-se um diretório dentro da pasta de skills pessoais:

```
mkdir-p ~/.claude/skills/pr-description
```

Depois, cria-se o arquivo:

```
~/.claude/skills/pr-description/SKILL.md
```

Como essa é uma **personal skill**, ela fica disponível em todos os projetos do usuário.

---

### Exemplo demonstrado na aula

A skill `pr-description` ensina Claude a escrever descrições de pull requests seguindo um template fixo:

```
## What

One sentence explaining what this PR does.

## Why

Brief context on why this change is needed

## Changes

- Bullet points of specific changes made
- Group related changes together
- Mention any files deleted or renamed
```

Esse exemplo demonstra o conceito de **padronização de tarefas repetitivas**.

Em vez de sempre pedir a Claude para escrever uma descrição de PR no mesmo formato, o usuário cria uma skill uma vez. Depois, quando pedir algo como “write a PR description for my changes”, Claude pode usar automaticamente essa skill.

---

### Testando uma skill

Depois de criar uma skill, é necessário **reiniciar o Claude Code**, porque as skills são carregadas no início da sessão.

Para testar, o usuário pode:

1. Criar ou modificar arquivos em uma branch.
2. Pedir: “write a PR description for my changes”.
3. Claude identifica a skill relevante.
4. Claude usa o template definido na skill.
5. Claude gera a descrição do PR com o formato esperado.

---

### Como o matching de skills funciona

Quando Claude Code inicia, ele verifica os locais onde skills podem existir, mas carrega apenas:

- O nome da skill.
- A descrição da skill.

Ele **não carrega todo o conteúdo da skill imediatamente**.

Quando o usuário envia uma solicitação, Claude compara a mensagem com as descrições das skills disponíveis usando **semantic matching**, ou correspondência semântica.

Exemplo:

```
"explain what this function does"
```

Esse pedido poderia corresponder a uma skill descrita como:

```
"explain code with visual diagrams"
```

Mesmo que as palavras não sejam idênticas, a intenção é parecida.

---

### Confirmação antes de carregar a skill

Quando Claude encontra uma skill correspondente, ele pede confirmação antes de carregar o conteúdo completo da skill no contexto.

Isso é importante porque:

- Dá visibilidade ao usuário.
- Evita carregar instruções sem consentimento.
- Ajuda o usuário a saber qual contexto está sendo usado.

Após a confirmação, Claude lê o `SKILL.md` completo e segue as instruções.

---

### Prioridade em conflitos de nomes

Se duas skills tiverem o mesmo nome, Claude segue uma ordem de prioridade clara:

1. **Enterprise**
2. **Personal**
3. **Project**
4. **Plugins**

Ou seja, se houver uma skill chamada `code-review` definida pela empresa e outra skill pessoal com o mesmo nome, a skill de **Enterprise** vence.

Essa hierarquia permite que organizações imponham padrões globais, enquanto ainda permitem personalizações individuais e por projeto.

---

### Atualizando e removendo skills

Para atualizar uma skill:

```
Edite o arquivo SKILL.md
```

Para remover uma skill:

```
Delete o diretório da skill
```

Após qualquer alteração, é necessário:

```
Reiniciar o Claude Code
```

Sem reiniciar, as mudanças podem não ser reconhecidas.

---

## 3. Pontos importantes para certificação

- Uma skill é um **diretório** contendo um arquivo `SKILL.md`.
- O arquivo `SKILL.md` contém **frontmatter** com `name` e `description`.
- A `description` é usada como critério de correspondência para decidir quando a skill deve ser ativada.
- Claude Code carrega no startup apenas o **nome** e a **descrição** das skills.
- O conteúdo completo da skill só é carregado depois de uma correspondência e confirmação.
- O matching é feito de forma **semântica**, não apenas por palavras exatas.
- Claude pede confirmação antes de carregar o conteúdo completo da skill.
- Skills pessoais ficam em:

```
~/.claude/skills
```

- O diretório da skill geralmente deve ter o mesmo nome da skill.
- Para atualizar uma skill, edite o `SKILL.md`.
- Para remover uma skill, delete o diretório da skill.
- Sempre reinicie o Claude Code após criar, editar ou remover uma skill.
- Em caso de conflito de nomes, a prioridade é:

```
Enterprise → Personal → Project → Plugins
```

- Skills Enterprise têm a maior prioridade.
- Plugins têm a menor prioridade.
- Para evitar conflitos, use nomes descritivos, como `frontend-review` ou `backend-review`, em vez de apenas `review`.

---

## 4. Termos-chave

### Skill

Diretório com instruções e recursos que ensinam Claude Code a executar melhor uma tarefa específica.

### `SKILL.md`

Arquivo principal da skill. Contém os metadados no frontmatter e as instruções da skill.

### Frontmatter

Bloco inicial delimitado por `---`, usado para declarar metadados como `name` e `description`.

### `name`

Campo que define o nome da skill. Ele identifica a skill dentro do Claude Code.

### `description`

Campo que explica quando a skill deve ser usada. É o principal critério de matching.

### Semantic matching

Processo em que Claude compara a intenção do pedido do usuário com a descrição da skill, mesmo que as palavras usadas sejam diferentes.

### Personal skill

Skill armazenada no diretório pessoal do usuário, disponível em todos os projetos.

### Project skill

Skill armazenada dentro de um repositório, disponível para todos que trabalham naquele projeto.

### Enterprise skill

Skill gerenciada pela organização. Tem prioridade máxima em caso de conflitos.

### Plugin skill

Skill fornecida por plugins instalados. Tem a menor prioridade na hierarquia.

### Context

Espaço de informações carregadas na conversa. Skills ajudam a economizar contexto porque são carregadas apenas quando relevantes.

---

## 5. Boas práticas

- Use nomes claros e específicos para as skills.
- Escreva descrições objetivas, pois elas determinam quando Claude ativará a skill.
- Evite nomes genéricos como:

```
review
```

Prefira nomes mais específicos, como:

```
frontend-review
backend-review
pr-description
```

- Coloque tarefas pessoais repetitivas em skills pessoais.
- Coloque padrões compartilhados da equipe em project skills.
- Use enterprise skills para regras organizacionais que precisam prevalecer.
- Mantenha o `SKILL.md` simples, direto e acionável.
- Reinicie Claude Code sempre que criar, editar ou remover uma skill.
- Teste a skill com uma solicitação real parecida com a situação em que ela deverá ser usada.
- Use skills para tarefas com formato recorrente, como:
    - Descrições de PR.
    - Revisões de código.
    - Mensagens de commit.
    - Templates de documentação.
    - Checklists de QA.
    - Explicações técnicas padronizadas.

---

## 6. Limitações, riscos e cuidados

- Claude não carrega imediatamente o conteúdo completo da skill; ele carrega apenas nome e descrição no início.
- Se a descrição for mal escrita, Claude pode não identificar a skill correta.
- Se a descrição for ampla demais, a skill pode ser sugerida em situações inadequadas.
- Alterações no `SKILL.md` não entram em vigor até reiniciar o Claude Code.
- Skills com o mesmo nome podem gerar conflitos.
- A hierarquia de prioridade pode fazer com que uma skill pessoal ou de projeto seja ignorada se existir uma skill enterprise com o mesmo nome.
- Nomes genéricos aumentam o risco de colisão.
- Project skills precisam ser mantidas com cuidado, pois afetam todos os membros que clonam o repositório.
- Enterprise skills podem limitar customizações individuais, já que têm prioridade máxima.
- O usuário deve prestar atenção à confirmação antes de carregar uma skill, para entender qual contexto será adicionado.

---

## 7. Resumo para revisão rápida

- Skill = diretório com um arquivo `SKILL.md`.
- `SKILL.md` contém frontmatter e instruções.
- Frontmatter inclui `name` e `description`.
- Claude usa a `description` para decidir quando usar a skill.
- Claude Code carrega apenas nomes e descrições no startup.
- O conteúdo completo é carregado apenas após matching e confirmação.
- Matching é semântico, baseado na intenção do pedido.
- Personal skills ficam em `~/.claude/skills`.
- Para criar uma skill, crie um diretório e um arquivo `SKILL.md`.
- Para atualizar, edite o `SKILL.md`.
- Para remover, delete o diretório.
- Sempre reinicie Claude Code após mudanças.
- Prioridade de conflitos:

```
Enterprise → Personal → Project → Plugins
```

- Use nomes específicos para evitar conflitos.
- Skills são ideais para tarefas repetitivas e padronizadas.

---

## 8. Perguntas simuladas

### Pergunta 1

Qual é a estrutura básica de uma skill no Claude Code?

A. Um arquivo `CLAUDE.md` com comandos globais.

B. Um diretório contendo um arquivo `SKILL.md`.

C. Um plugin instalado no navegador.

D. Um comando iniciado com `/`.

**Resposta correta: B**

**Explicação:** Uma skill é um diretório que contém um arquivo `SKILL.md` com metadados e instruções.

---

### Pergunta 2

Quais campos aparecem no frontmatter básico de uma skill?

A. `title` e `author`

B. `name` e `description`

C. `version` e `repository`

D. `command` e `shortcut`

**Resposta correta: B**

**Explicação:** O frontmatter básico contém `name`, que identifica a skill, e `description`, que informa quando ela deve ser usada.

---

### Pergunta 3

O que Claude Code carrega no startup em relação às skills?

A. Todo o conteúdo de todas as skills.

B. Apenas os nomes e descrições das skills.

C. Apenas as skills de projeto.

D. Apenas as skills enterprise.

**Resposta correta: B**

**Explicação:** Claude Code carrega apenas nome e descrição no início, preservando o contexto.

---

### Pergunta 4

Como Claude decide se uma skill deve ser usada?

A. Ele verifica se o usuário digitou exatamente o nome da skill.

B. Ele compara semanticamente o pedido do usuário com a descrição da skill.

C. Ele usa sempre a primeira skill disponível.

D. Ele só usa skills ativadas por slash commands.

**Resposta correta: B**

**Explicação:** O matching é semântico, ou seja, baseado na intenção do pedido e na descrição da skill.

---

### Pergunta 5

O que acontece quando Claude encontra uma skill correspondente?

A. Ele carrega automaticamente todo o conteúdo sem avisar.

B. Ele apaga as outras skills.

C. Ele pede confirmação antes de carregar o conteúdo completo.

D. Ele reinicia o Claude Code.

**Resposta correta: C**

**Explicação:** Claude pede confirmação antes de carregar a skill completa no contexto.

---

### Pergunta 6

Qual é a ordem correta de prioridade quando há skills com o mesmo nome?

A. Project → Personal → Enterprise → Plugins

B. Plugins → Project → Personal → Enterprise

C. Enterprise → Personal → Project → Plugins

D. Personal → Enterprise → Plugins → Project

**Resposta correta: C**

**Explicação:** A hierarquia correta é Enterprise primeiro, depois Personal, Project e, por último, Plugins.

---

### Pergunta 7

O que deve ser feito após editar ou remover uma skill?

A. Criar um novo repositório.

B. Reiniciar o Claude Code.

C. Renomear o arquivo para `CLAUDE.md`.

D. Instalar um plugin.

**Resposta correta: B**

**Explicação:** As mudanças só entram em vigor depois que Claude Code é reiniciado.

---

### Pergunta 8

Por que é recomendado usar nomes descritivos para skills?

A. Para deixar o arquivo menor.

B. Para evitar conflitos de nomes e tornar o uso mais claro.

C. Para impedir o uso de skills enterprise.

D. Para que a skill seja carregada em todas as conversas.

**Resposta correta: B**

**Explicação:** Nomes específicos reduzem colisões e ajudam a identificar melhor a finalidade da skill.

---

### Pergunta 9

No exemplo da aula, qual era a finalidade da skill `pr-description`?

A. Corrigir bugs automaticamente.

B. Criar descrições de pull requests em um formato consistente.

C. Instalar dependências do projeto.

D. Fazer deploy em produção.

**Resposta correta: B**

**Explicação:** A skill ensina Claude a gerar descrições de PR com seções como `What`, `Why` e `Changes`.

---

### Pergunta 10

Qual é uma vantagem de carregar skills sob demanda?

A. Todas as instruções ficam sempre no contexto.

B. O contexto é economizado, pois apenas skills relevantes são carregadas.

C. O usuário não consegue saber quais skills foram usadas.

D. As skills deixam de precisar de descrição.

**Resposta correta: B**

**Explicação:** Como Claude só carrega o conteúdo completo quando a skill é relevante, o contexto é usado de forma mais eficiente.