# Compartilhando habilidades

## 1. Ideia central

A aula explica como **compartilhar skills no Claude Code** e como elas se comportam em diferentes contextos: repositórios Git, plugins, ambientes enterprise e subagents.

A ideia principal é que skills ficam mais poderosas quando deixam de ser apenas uma configuração pessoal e passam a padronizar fluxos de trabalho em equipes ou organizações inteiras.

Uma skill de revisão de PR, por exemplo, pode ajudar individualmente. Mas, quando compartilhada com toda a equipe, ela cria um padrão consistente para revisão de código.

---

## 2. Principais conceitos

### Compartilhando skills pelo repositório

A forma mais simples de compartilhar skills é colocá-las dentro do repositório, no diretório:

```
.claude/skills
```

Quando esse diretório é versionado com Git, qualquer pessoa que clonar o repositório recebe as skills automaticamente.

Esse método é ideal para:

- Padrões de código da equipe.
- Workflows específicos do projeto.
- Skills que dependem da estrutura do codebase.
- Templates internos do repositório.
- Regras de revisão ou documentação específicas daquele projeto.

A pasta `.claude` pode conter:

```
.claude/
  agents/
  hooks/
  skills/
  settings/
```

Ou seja, ela pode centralizar customizações compartilhadas do projeto.

---

### Compartilhando skills por plugins

Plugins permitem distribuir skills entre vários projetos e equipes, especialmente quando elas não são específicas de um único repositório.

Em um projeto de plugin, você pode criar um diretório de skills com estrutura parecida com a pasta `.claude`.

Cada skill continua tendo sua própria pasta com um arquivo:

```
SKILL.md
```

Depois de publicar o plugin em um marketplace, outros usuários podem descobrir, instalar e usar essas skills no Claude Code.

Esse método é indicado quando a skill pode ser útil para uma comunidade mais ampla, como:

- Skills genéricas de documentação.
- Skills de revisão frontend.
- Skills de acessibilidade.
- Skills de performance.
- Skills de boas práticas de segurança.
- Skills reutilizáveis entre diferentes times ou organizações.

---

### Deploy enterprise por managed settings

Administradores podem implantar skills em toda a organização usando **managed settings**.

Essas skills têm a maior prioridade na hierarquia.

A ordem de prioridade é:

```
Enterprise → Personal → Project → Plugins
```

Isso significa que, se uma skill enterprise tiver o mesmo nome que uma skill pessoal, de projeto ou de plugin, a versão enterprise vence.

Esse método é ideal para padrões obrigatórios, como:

- Requisitos de segurança.
- Fluxos de compliance.
- Políticas corporativas.
- Práticas de código obrigatórias.
- Regras de governança.
- Procedimentos que precisam ser iguais para toda a organização.

A palavra-chave aqui é **must**: use enterprise quando o padrão precisa ser obrigatório.

---

### `strictKnownMarketplaces`

Managed settings também podem controlar de onde plugins podem ser instalados.

Exemplo:

```
{
  "strictKnownMarketplaces": [
    {
      "source":"github",
      "repo":"acme-corp/approved-plugins"
    },
    {
      "source":"npm",
      "package":"@acme-corp/compliance-plugins"
    }
  ]
}
```

Esse tipo de configuração ajuda organizações a limitar plugins a fontes aprovadas, reduzindo riscos de segurança e compliance.

---

### Skills e subagents

Um ponto importante da aula é que **subagents não enxergam automaticamente as suas skills**.

Quando você delega uma tarefa para um subagent, ele começa com um contexto novo e limpo.

Existem diferenças importantes:

- Built-in agents, como **Explorer**, **Plan** e **Verify**, não conseguem acessar skills.
- Custom subagents podem usar skills, mas somente quando elas são explicitamente listadas.
- Skills em subagents são carregadas quando o subagent inicia.
- No subagent, as skills não são carregadas sob demanda como na conversa principal.

---

### Custom subagents com skills

Para criar um subagent customizado com skills, é preciso definir um arquivo markdown em:

```
.claude/agents
```

Esse arquivo pode conter um campo `skills` no frontmatter.

Exemplo:

```
---
name: frontend-security-accessibility-reviewer
description: "Use this agent when you need to review frontend code for accessibility..."
tools: Bash, Glob, Grep, Read, WebFetch, WebSearch, Skill...
model: sonnet
color: blue
skills: accessibility-audit, performance-check
---
```

Nesse exemplo, o subagent carrega duas skills:

```
accessibility-audit
performance-check
```

Quando uma tarefa é delegada para esse subagent, ele aplica essas skills em suas revisões.

Antes disso, é necessário garantir que as skills existam no diretório:

```
.claude/skills
```

---

### Quando usar subagents com skills

Esse padrão funciona bem quando você quer combinar:

1. Delegação isolada.
2. Especialização específica.
3. Consistência em tarefas repetitivas.

Exemplos:

- Um subagent de revisão frontend com skills de acessibilidade e performance.
- Um subagent backend com skills de segurança de API e arquitetura.
- Um subagent de documentação com skills de escrita técnica.
- Um subagent de QA com skills de checklist de testes.

---

## 3. Pontos importantes para certificação

- Project skills ficam em:

```
.claude/skills
```

- Project skills são compartilhadas automaticamente via Git.
- Quem clona o repositório recebe as skills do projeto.
- Plugins permitem distribuir skills entre múltiplos repositórios e para a comunidade.
- Plugins são melhores quando a skill não é específica de um único projeto.
- Enterprise managed settings permitem implantar skills em toda a organização.
- Skills enterprise têm a maior prioridade.
- Enterprise é ideal para padrões obrigatórios de segurança, compliance e governança.
- `strictKnownMarketplaces` pode restringir marketplaces ou fontes de plugins aprovadas.
- Subagents não veem skills automaticamente.
- Built-in agents como Explorer, Plan e Verify não conseguem acessar skills.
- Custom subagents podem usar skills, mas elas precisam ser listadas explicitamente no campo `skills`.
- Skills em custom subagents são carregadas quando o subagent inicia.
- No subagent, skills não são ativadas sob demanda como na conversa principal.
- Para usar skills em um custom subagent, as skills devem existir em `.claude/skills`.
- A escolha do método de compartilhamento depende do escopo:
    - Projeto específico → repositório.
    - Reutilização ampla → plugin.
    - Padrão obrigatório organizacional → enterprise.

---

## 4. Termos-chave

### Project skill

Skill armazenada em `.claude/skills` dentro do repositório. É compartilhada automaticamente com quem clona o projeto.

### Plugin

Forma de distribuir skills e funcionalidades para múltiplos projetos ou usuários, geralmente via marketplace.

### Marketplace

Local onde plugins podem ser publicados, descobertos e instalados por outros usuários.

### Enterprise managed settings

Configurações gerenciadas por administradores para aplicar padrões em toda a organização.

### Enterprise skill

Skill implantada pela organização, com prioridade máxima sobre skills pessoais, de projeto e de plugins.

### `strictKnownMarketplaces`

Configuração que limita os marketplaces ou fontes de plugins permitidos pela organização.

### Subagent

Agente que executa tarefas em contexto separado. Não recebe skills automaticamente.

### Built-in agents

Agentes nativos do Claude Code, como Explorer, Plan e Verify. Segundo a aula, eles não podem acessar skills.

### Custom subagent

Subagent definido pelo usuário em `.claude/agents`. Pode usar skills se elas forem explicitamente listadas.

### `skills` field

Campo no frontmatter de um custom subagent usado para listar quais skills serão carregadas.

### `.claude/agents`

Diretório onde ficam definições de custom subagents.

### `.claude/skills`

Diretório onde ficam skills do projeto.

---

## 5. Boas práticas

- Use `.claude/skills` para skills específicas do projeto.
- Versione project skills com Git para padronizar o trabalho da equipe.
- Use plugins para skills reutilizáveis em vários projetos.
- Use enterprise managed settings para padrões obrigatórios.
- Evite usar enterprise para preferências opcionais; reserve para regras que realmente precisam ser impostas.
- Use nomes claros para evitar conflitos entre skills enterprise, pessoais, de projeto e plugins.
- Antes de usar uma skill em um subagent, confirme que ela existe em `.claude/skills`.
- Liste explicitamente as skills no campo `skills` dos custom subagents.
- Crie subagents especializados para áreas diferentes, como frontend, backend, segurança, documentação ou QA.
- Combine subagents com skills quando quiser consistência em trabalho delegado.
- Use `strictKnownMarketplaces` quando a organização precisar controlar fontes de plugins por segurança ou compliance.

---

## 6. Limitações, riscos e cuidados

- Project skills só são compartilhadas se forem commitadas no repositório.
- Skills via plugin podem ser genéricas demais se não forem bem projetadas.
- Enterprise skills podem sobrescrever skills pessoais, de projeto ou plugin com o mesmo nome.
- Conflitos de nomes podem causar comportamento inesperado se a equipe não entender a ordem de prioridade.
- Built-in agents não conseguem acessar skills.
- Subagents customizados não carregam skills automaticamente.
- Se o campo `skills` não for definido no subagent, ele não usará aquelas skills.
- Skills em subagents são carregadas no início do subagent, não sob demanda.
- Plugins de fontes não confiáveis podem representar risco; por isso managed settings e `strictKnownMarketplaces` são importantes.
- Organizações devem distinguir entre padrões obrigatórios e preferências flexíveis para não exagerar no uso de enterprise settings.

---

## 7. Resumo para revisão rápida

- Skills podem ser compartilhadas por repositório, plugins ou enterprise settings.
- Project skills ficam em `.claude/skills`.
- Skills em `.claude/skills` são compartilhadas via Git.
- Plugins distribuem skills entre projetos e comunidades.
- Enterprise managed settings aplicam skills para toda a organização.
- Enterprise skills têm prioridade máxima.
- Use enterprise para segurança, compliance e padrões obrigatórios.
- `strictKnownMarketplaces` controla fontes permitidas de plugins.
- Subagents não veem skills automaticamente.
- Built-in agents não acessam skills.
- Custom subagents podem usar skills se listadas no campo `skills`.
- Skills em subagents carregam no início, não sob demanda.
- Para custom subagents com skills, use `.claude/agents` e `.claude/skills`.
- Escolha o método de compartilhamento conforme o escopo:
    - Repositório → equipe/projeto.
    - Plugin → reutilização ampla.
    - Enterprise → obrigação organizacional.

---

## 8. Perguntas simuladas

### Pergunta 1

Qual é a forma mais simples de compartilhar skills com uma equipe em um projeto?

A. Criar uma skill apenas no diretório pessoal.

B. Colocar as skills em `.claude/skills` e versionar pelo Git.

C. Enviar o conteúdo da skill manualmente por chat.

D. Criar apenas um hook.

**Resposta correta: B**

**Explicação:** Skills em `.claude/skills` são compartilhadas automaticamente com quem clona o repositório.

---

### Pergunta 2

Onde ficam as project skills?

A. `~/.claude/skills`

B. `.claude/skills`

C. `.claude/agents`

D. `/plugins/global/skills`

**Resposta correta: B**

**Explicação:** Project skills ficam dentro do repositório no diretório `.claude/skills`.

---

### Pergunta 3

Quando faz mais sentido distribuir uma skill por plugin?

A. Quando ela é específica de um único codebase.

B. Quando ela pode ser útil em vários projetos ou para uma comunidade maior.

C. Quando ela deve sobrescrever padrões enterprise.

D. Quando ela só deve funcionar para um usuário local.

**Resposta correta: B**

**Explicação:** Plugins são indicados para distribuição ampla, entre repositórios, equipes ou comunidade.

---

### Pergunta 4

Qual tipo de skill tem maior prioridade em conflitos de nome?

A. Project skill

B. Plugin skill

C. Personal skill

D. Enterprise skill

**Resposta correta: D**

**Explicação:** Skills enterprise têm prioridade máxima e sobrescrevem personal, project e plugin skills com o mesmo nome.

---

### Pergunta 5

Qual é a ordem correta de prioridade entre skills?

A. Plugins → Project → Personal → Enterprise

B. Project → Plugins → Enterprise → Personal

C. Enterprise → Personal → Project → Plugins

D. Personal → Project → Plugins → Enterprise

**Resposta correta: C**

**Explicação:** A ordem correta é Enterprise, depois Personal, depois Project e, por fim, Plugins.

---

### Pergunta 6

Para quais situações enterprise managed settings são mais adequadas?

A. Preferências pessoais de escrita.

B. Padrões obrigatórios de segurança, compliance e governança.

C. Testes temporários de um único desenvolvedor.

D. Skills experimentais sem impacto organizacional.

**Resposta correta: B**

**Explicação:** Enterprise managed settings são ideais para requisitos que precisam ser obrigatórios em toda a organização.

---

### Pergunta 7

O que faz a configuração `strictKnownMarketplaces`?

A. Impede o uso de qualquer skill pessoal.

B. Limita de quais marketplaces ou fontes plugins podem ser instalados.

C. Carrega skills automaticamente em subagents built-in.

D. Remove a necessidade de `SKILL.md`.

**Resposta correta: B**

**Explicação:** `strictKnownMarketplaces` controla as fontes aprovadas para instalação de plugins.

---

### Pergunta 8

Subagents veem automaticamente as skills disponíveis na conversa principal?

A. Sim, todos os subagents herdam as skills automaticamente.

B. Não, subagents começam com um contexto limpo.

C. Sim, mas apenas os built-in agents.

D. Sim, desde que a skill esteja em um plugin.

**Resposta correta: B**

**Explicação:** Subagents não recebem skills automaticamente; eles começam em um contexto separado e limpo.

---

### Pergunta 9

Built-in agents como Explorer, Plan e Verify podem acessar skills?

A. Sim, sempre.

B. Sim, se a skill estiver em `.claude/skills`.

C. Não, eles não conseguem acessar skills.

D. Apenas se forem instalados por plugin.

**Resposta correta: C**

**Explicação:** Segundo a aula, built-in agents não podem acessar skills.

---

### Pergunta 10

Como um custom subagent pode usar skills?

A. Basta a skill existir no diretório pessoal do usuário.

B. A skill precisa ser listada explicitamente no campo `skills` do frontmatter do agente.

C. É necessário transformar a skill em hook.

D. O subagent precisa chamar um MCP server.

**Resposta correta: B**

**Explicação:** Custom subagents só usam skills quando elas são explicitamente declaradas no campo `skills`.

---

### Pergunta 11

Quando as skills são carregadas em um custom subagent?

A. Sob demanda, exatamente como na conversa principal.

B. Quando o subagent inicia.

C. Apenas depois de um comando `/skills`.

D. Nunca, pois subagents não podem usar skills.

**Resposta correta: B**

**Explicação:** Em custom subagents, as skills listadas são carregadas no início da execução do subagent.

---

### Pergunta 12

Qual cenário combina melhor com custom subagents usando skills?

A. Uma regra global que deve valer em toda conversa.

B. Um subagent frontend que sempre revisa acessibilidade e performance usando skills específicas.

C. Uma skill pessoal de estilo de escrita usada apenas pelo usuário.

D. Uma integração externa com banco de dados via API.

**Resposta correta: B**

**Explicação:** Custom subagents com skills são úteis para delegar tarefas isoladas com expertise específica e consistente.