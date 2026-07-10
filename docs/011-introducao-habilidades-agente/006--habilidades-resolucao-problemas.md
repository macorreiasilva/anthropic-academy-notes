# Habilidades de resolução de problemas

# Resumo do conteúdo

## 1. Ideia central

A aula explica como fazer **troubleshooting de skills no Claude Code** quando elas não funcionam como esperado.

Os problemas mais comuns caem em algumas categorias:

- A skill não é ativada.
- A skill não aparece ou não carrega.
- Claude usa a skill errada.
- Há conflito de prioridade entre skills com o mesmo nome.
- Skills de plugins não aparecem.
- A skill carrega, mas falha durante a execução.

A ideia principal é começar pelo **validador de skills**, depois verificar descrição, estrutura de arquivos, conflitos de nome, permissões, dependências e caminhos.

---

## 2. Principais conceitos

### Skills validator

O primeiro passo recomendado é usar a ferramenta de validação de skills, chamada na aula de **agent skills verifier command**.

Ela verifica problemas estruturais antes que você perca tempo investigando outras causas.

O validador pode identificar erros como:

- Estrutura incorreta de diretórios.
- Problemas no arquivo `SKILL.md`.
- Erros de frontmatter ou YAML.
- Arquivos posicionados no lugar errado.
- Possíveis falhas que impedem a skill de carregar.

A recomendação da aula é: **comece sempre pelo validador**.

---

### Skill não é ativada

Quando a skill existe e passa na validação, mas Claude não a usa quando esperado, o problema quase sempre está na `description`.

Claude usa **semantic matching**, ou seja, ele compara o pedido do usuário com o significado da descrição da skill.

Se não houver sobreposição suficiente entre o pedido e a descrição, a skill não será ativada.

Exemplo de pedidos que talvez precisem estar refletidos na descrição:

```
help me profile this
why is this slow?
make this faster
```

Se a skill é sobre performance, mas a descrição não inclui frases ou conceitos parecidos com esses pedidos, Claude pode não ativá-la.

A solução é adicionar **trigger phrases**, ou seja, frases de acionamento que se parecem com a forma real como os usuários fazem pedidos.

---

### Skill não carrega

Se a skill não aparece quando você pergunta quais skills estão disponíveis, o problema pode estar na estrutura.

Requisitos importantes:

```
A skill deve estar dentro de um diretório nomeado.
O arquivo deve se chamar exatamente SKILL.md.
```

Erro comum:

```
~/.claude/skills/SKILL.md
```

Esse arquivo está direto na raiz de `skills`, o que está errado.

Estrutura correta:

```
~/.claude/skills/my-skill/SKILL.md
```

Outro detalhe importante: o nome do arquivo precisa ser exatamente:

```
SKILL.md
```

Com `SKILL` em letras maiúsculas e `md` em minúsculas.

---

### Uso de `claude --debug`

Quando uma skill não carrega, a aula recomenda rodar:

```
claude--debug
```

Esse comando pode mostrar erros de carregamento. O ideal é procurar mensagens que mencionem o nome da skill.

Muitas vezes, o próprio log de debug aponta diretamente para o problema.

---

### Skill errada sendo usada

Se Claude usa a skill errada ou parece confuso entre duas skills, provavelmente as descrições estão parecidas demais.

A solução é tornar as descrições mais distintas.

Descrições específicas ajudam Claude a:

- Saber quando usar uma skill.
- Saber quando não usar uma skill.
- Evitar conflitos com skills semelhantes.

Exemplo de descrições muito genéricas:

```
Reviews code.
```

```
Checks code quality.
```

Essas descrições podem causar confusão.

Descrições melhores seriam mais específicas:

```
Reviews frontend React components for accessibility and UI consistency.
```

```
Reviews backend API changes for security, validation, and error handling.
```

---

### Conflitos de prioridade

Se uma skill pessoal está sendo ignorada, talvez exista uma skill de prioridade maior com o mesmo nome.

A hierarquia de prioridade é:

```
Enterprise → Personal → Project → Plugins
```

Exemplo:

Se existir uma skill enterprise chamada:

```
code-review
```

E você também criar uma skill pessoal chamada:

```
code-review
```

A skill enterprise vence sempre.

Soluções possíveis:

- Renomear sua skill para algo mais específico.
- Conversar com o administrador responsável pela skill enterprise.

Na prática, renomear costuma ser o caminho mais simples.

---

### Skills de plugins não aparecem

Se você instalou um plugin, mas as skills dele não aparecem, a aula recomenda:

1. Limpar o cache.
2. Reiniciar o Claude Code.
3. Reinstalar o plugin.

Se ainda assim as skills não aparecerem, o problema pode estar na estrutura do plugin.

Nesse caso, o validador de skills é especialmente útil.

---

### Erros em tempo de execução

Às vezes, a skill carrega corretamente, mas falha durante a execução.

Causas comuns:

#### Dependências ausentes

Se a skill usa pacotes externos, eles precisam estar instalados.

Boa prática: documentar as dependências na descrição ou nas instruções da skill para que Claude saiba o que é necessário.

#### Problemas de permissão

Scripts precisam ter permissão de execução.

Exemplo:

```
chmod+x script.sh
```

Sem essa permissão, o script pode falhar mesmo que a skill esteja correta.

#### Separadores de caminho

A aula recomenda usar **forward slashes** em todos os caminhos, inclusive no Windows.

Use:

```
scripts/check.sh
```

Evite:

```
scripts\check.sh
```

---

## 3. Pontos importantes para certificação

- O primeiro passo no troubleshooting deve ser usar o **skills validator**.
- O validador identifica problemas estruturais antes de investigar causas mais complexas.
- Se a skill não ativa, o problema geralmente está na `description`.
- Claude usa **semantic matching** para associar pedidos às skills.
- A `description` deve refletir a forma real como os usuários fazem pedidos.
- Use **trigger phrases** na descrição para melhorar o acionamento.
- Se a skill não carrega, verifique se `SKILL.md` está dentro de um diretório nomeado.
- `SKILL.md` não deve ficar diretamente na raiz da pasta `skills`.
- O nome do arquivo precisa ser exatamente `SKILL.md`.
- Se a skill não aparece, use `claude --debug` para procurar erros de carregamento.
- Se a skill errada é usada, as descrições provavelmente estão parecidas demais.
- Conflitos de prioridade seguem a hierarquia:

```
Enterprise → Personal → Project → Plugins
```

- Uma skill enterprise com o mesmo nome sobrescreve uma skill pessoal.
- Para resolver shadowing, geralmente é melhor renomear a skill.
- Se skills de plugin não aparecem, limpe cache, reinicie e reinstale.
- Erros em runtime geralmente envolvem dependências, permissões ou caminhos.
- Scripts precisam de permissão de execução com `chmod +x`.
- Caminhos devem usar forward slashes, mesmo no Windows.

---

## 4. Termos-chave

### Troubleshooting

Processo de identificar e corrigir problemas quando uma skill não funciona como esperado.

### Skills validator

Ferramenta que verifica se a estrutura da skill está correta e ajuda a encontrar erros antes de testes manuais.

### Semantic matching

Correspondência semântica entre a solicitação do usuário e a descrição da skill.

### Description

Campo do frontmatter usado por Claude para decidir quando ativar uma skill.

### Trigger phrases

Frases ou palavras que refletem como os usuários normalmente pedem uma tarefa e ajudam Claude a ativar a skill correta.

### `SKILL.md`

Arquivo principal da skill. Precisa ter esse nome exato e ficar dentro de um diretório nomeado.

### Skills root

Diretório raiz onde ficam as pastas das skills. O arquivo `SKILL.md` não deve ficar diretamente nele.

### `claude --debug`

Comando usado para ver logs e erros de carregamento no Claude Code.

### Skill priority

Hierarquia que define qual skill vence quando há duas ou mais skills com o mesmo nome.

### Shadowing

Situação em que uma skill de maior prioridade impede outra skill com o mesmo nome de ser usada.

### Plugin skill

Skill distribuída por meio de um plugin.

### Runtime error

Erro que acontece durante a execução da skill, mesmo que ela tenha carregado corretamente.

### Dependency

Pacote, biblioteca ou ferramenta externa necessária para a skill funcionar.

### File permission

Permissão do sistema operacional que permite ou bloqueia a execução de arquivos, como scripts.

### Path separator

Separador usado em caminhos de arquivos. A recomendação é usar `/`, mesmo no Windows.

---

## 5. Boas práticas

- Comece sempre pelo validador de skills.
- Valide a skill antes de compartilhá-la com a equipe.
- Escreva descrições específicas e alinhadas com pedidos reais dos usuários.
- Inclua trigger phrases na `description`.
- Teste a skill com várias formas de pedir a mesma tarefa.
- Use descrições distintas para skills com funções parecidas.
- Evite nomes genéricos que possam conflitar com skills enterprise, pessoais, de projeto ou plugins.
- Coloque `SKILL.md` sempre dentro de um diretório nomeado.
- Verifique se o arquivo se chama exatamente `SKILL.md`.
- Use `claude --debug` quando a skill não aparecer.
- Documente dependências necessárias para scripts ou ferramentas externas.
- Dê permissão de execução para scripts usados pela skill.
- Use forward slashes nos caminhos.
- Reinicie Claude Code após alterações relevantes.
- Antes de publicar skills via repositório ou plugin, rode validação e teste cenários comuns.

---

## 6. Limitações, riscos e cuidados

- Uma skill pode estar correta estruturalmente, mas ainda assim não ativar por causa de uma descrição fraca.
- Descrições muito genéricas aumentam o risco de Claude escolher a skill errada.
- Skills com nomes iguais podem ser sobrescritas por versões de maior prioridade.
- Uma skill pessoal pode ser ignorada se houver uma enterprise skill com o mesmo nome.
- Plugins podem não exibir skills se houver problemas de cache, instalação ou estrutura.
- Scripts podem falhar por falta de dependências externas.
- Scripts sem permissão de execução podem causar erro em runtime.
- Caminhos com separadores inconsistentes podem quebrar compatibilidade, especialmente entre sistemas operacionais.
- Resolver problemas sem usar o validador pode levar a perda de tempo com hipóteses erradas.
- Compartilhar skills sem validação pode espalhar erros para toda a equipe.

---

## 7. Resumo para revisão rápida

- Comece troubleshooting com o skills validator.
- Se a skill não ativa, revise a `description`.
- Use trigger phrases parecidas com pedidos reais.
- Se a skill não carrega, verifique estrutura e nome do arquivo.
- `SKILL.md` deve estar dentro de uma pasta nomeada.
- O nome deve ser exatamente `SKILL.md`.
- Use `claude --debug` para encontrar erros de carregamento.
- Se a skill errada é usada, torne as descrições mais distintas.
- Conflitos seguem a prioridade:

```
Enterprise → Personal → Project → Plugins
```

- Enterprise vence personal, project e plugin em conflito de nome.
- Se plugin skills não aparecem, limpe cache, reinicie e reinstale.
- Se há erro em runtime, cheque dependências, permissões e caminhos.
- Scripts precisam de `chmod +x`.
- Use `/` como separador de caminho, mesmo no Windows.

---

## 8. Perguntas simuladas

### Pergunta 1

Qual deve ser o primeiro passo ao investigar uma skill que não funciona?

A. Renomear a skill imediatamente.

B. Usar o skills validator.

C. Apagar todas as skills pessoais.

D. Criar um novo plugin.

**Resposta correta: B**

**Explicação:** A aula recomenda começar pelo validador, pois ele identifica problemas estruturais antes de outras investigações.

---

### Pergunta 2

Se uma skill existe e passa na validação, mas não é ativada, qual é a causa mais provável?

A. O nome do sistema operacional.

B. A descrição da skill.

C. A cor do subagent.

D. A falta de um MCP server.

**Resposta correta: B**

**Explicação:** Quando a skill não ativa, o problema geralmente está na `description`, pois Claude usa esse campo para matching.

---

### Pergunta 3

O que são trigger phrases?

A. Scripts executáveis dentro da skill.

B. Frases que refletem como os usuários realmente fazem pedidos e ajudam a ativar a skill.

C. Permissões de arquivo para plugins.

D. Campos obrigatórios no YAML.

**Resposta correta: B**

**Explicação:** Trigger phrases aumentam a chance de Claude reconhecer quando uma skill deve ser usada.

---

### Pergunta 4

Qual estrutura está correta para uma skill?

A.

```
~/.claude/skills/SKILL.md
```

B.

```
~/.claude/skills/my-skill/SKILL.md
```

C.

```
~/.claude/SKILL.md
```

D.

```
~/.claude/skills/my-skill/skill.md
```

**Resposta correta: B**

**Explicação:** O arquivo `SKILL.md` deve ficar dentro de um diretório nomeado da skill.

---

### Pergunta 5

Qual nome de arquivo é correto para uma skill?

A. `skill.md`

B. `Skill.md`

C. `SKILL.md`

D. `skills.md`

**Resposta correta: C**

**Explicação:** O arquivo deve se chamar exatamente `SKILL.md`, com `SKILL` em maiúsculas e `md` em minúsculas.

---

### Pergunta 6

Qual comando pode ajudar a encontrar erros de carregamento de skills?

A. `claude --debug`

B. `claude --delete-skills`

C. `skills --force-run`

D. `git skill-check`

**Resposta correta: A**

**Explicação:** `claude --debug` mostra logs que podem indicar por que uma skill não carregou.

---

### Pergunta 7

Se Claude está usando a skill errada, qual é a causa mais provável?

A. O arquivo `SKILL.md` tem permissões de execução.

B. As descrições das skills estão parecidas demais.

C. O plugin foi instalado corretamente.

D. O modelo usado é sempre enterprise.

**Resposta correta: B**

**Explicação:** Quando descrições são muito semelhantes, Claude pode confundir qual skill deve ativar.

---

### Pergunta 8

Qual é a melhor forma de reduzir confusão entre skills parecidas?

A. Tornar as descrições mais distintas e específicas.

B. Remover o campo `description`.

C. Colocar todas as instruções no `CLAUDE.md`.

D. Usar nomes de arquivo diferentes de `SKILL.md`.

**Resposta correta: A**

**Explicação:** Descrições específicas ajudam Claude a escolher a skill correta e evitam conflitos.

---

### Pergunta 9

Se existe uma skill enterprise chamada `code-review` e uma skill pessoal também chamada `code-review`, qual será usada?

A. A pessoal, porque pertence ao usuário.

B. A de projeto, se existir.

C. A enterprise, porque tem maior prioridade.

D. A de plugin, porque é instalada externamente.

**Resposta correta: C**

**Explicação:** Enterprise tem a maior prioridade na hierarquia de skills.

---

### Pergunta 10

Qual é a ordem correta de prioridade de skills?

A. Plugins → Project → Personal → Enterprise

B. Enterprise → Personal → Project → Plugins

C. Project → Enterprise → Plugins → Personal

D. Personal → Plugins → Project → Enterprise

**Resposta correta: B**

**Explicação:** A prioridade correta é Enterprise, depois Personal, depois Project e por último Plugins.

---

### Pergunta 11

O que fazer se skills de um plugin instalado não aparecem?

A. Ignorar, pois plugins nunca podem conter skills.

B. Limpar cache, reiniciar Claude Code e reinstalar o plugin.

C. Mover o plugin para `CLAUDE.md`.

D. Remover todas as skills enterprise.

**Resposta correta: B**

**Explicação:** A aula recomenda limpar cache, reiniciar e reinstalar. Se persistir, verificar a estrutura com o validador.

---

### Pergunta 12

Uma skill carrega, mas falha ao executar um script. Qual causa é plausível?

A. A descrição está longa demais, necessariamente.

B. Falta de permissão de execução no script.

C. O arquivo se chama exatamente `SKILL.md`.

D. A skill está dentro de um diretório nomeado.

**Resposta correta: B**

**Explicação:** Scripts precisam de permissão de execução. Em sistemas Unix-like, isso pode ser corrigido com `chmod +x`.

---

### Pergunta 13

Qual separador de caminho a aula recomenda usar?

A. Barra invertida `\` em todos os sistemas.

B. Dois pontos `:`.

C. Barra normal `/`, inclusive no Windows.

D. Espaço em branco.

**Resposta correta: C**

**Explicação:** A recomendação é usar forward slashes em todos os caminhos, mesmo no Windows.

---

### Pergunta 14

Qual destas opções melhor resume o checklist de troubleshooting?

A. Verificar descrição, estrutura, conflitos, plugin cache, dependências, permissões e caminhos.

B. Sempre apagar a skill e recriá-la sem validar.

C. Usar apenas enterprise skills.

D. Nunca usar scripts dentro de skills.

**Resposta correta: A**

**Explicação:** O troubleshooting cobre ativação, carregamento, skill errada, prioridade, plugins e erros em runtime.