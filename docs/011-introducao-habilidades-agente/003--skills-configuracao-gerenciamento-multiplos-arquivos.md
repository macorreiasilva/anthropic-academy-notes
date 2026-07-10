# Skills de configuração e gerenciamento de múltiplos arquivos

# Resumo do conteúdo

## 1. Ideia central

A aula explica como configurar skills mais avançadas no Claude Code, indo além do básico `name` e `description`.

O foco está em quatro ideias principais:

1. Como usar campos de metadados no `SKILL.md`.
2. Como escrever boas descrições para melhorar o acionamento da skill.
3. Como restringir ferramentas com `allowed-tools`.
4. Como organizar skills maiores usando múltiplos arquivos, referências, scripts e assets.

A mensagem central é: uma skill deve ser **específica, segura, eficiente no uso de contexto e fácil de manter**.

---

## 2. Principais conceitos

### Campos de metadados da skill

O padrão aberto de agent skills permite vários campos no frontmatter do `SKILL.md`.

Os principais são:

```
---
name: codebase-onboarding
description: Helps new developers understand the system works.
allowed-tools: Read, Grep, Glob, Bash
model: sonnet
---
```

### `name`

Campo obrigatório.

Identifica a skill.

Regras importantes:

- Deve usar apenas letras minúsculas, números e hífens.
- Deve ter no máximo 64 caracteres.
- Deve corresponder ao nome do diretório da skill.

Exemplo:

```
codebase-onboarding
```

---

### `description`

Campo obrigatório.

É o campo mais importante para o matching da skill, porque Claude usa a descrição para decidir quando deve ativá-la.

A descrição deve responder duas perguntas:

1. **O que a skill faz?**
2. **Quando Claude deve usá-la?**

Exemplo ruim:

```
Helps with docs.
```

Esse exemplo é vago demais.

Exemplo melhor:

```
Writes API documentation from code comments. Use when creating, updating, or reviewing API docs.
```

Esse exemplo é mais claro porque explica a função e o momento de uso.

---

### `allowed-tools`

Campo opcional.

Permite restringir quais ferramentas Claude pode usar quando a skill está ativa.

Exemplo:

```
allowed-tools: Read, Grep, Glob, Bash
```

Nesse caso, Claude pode usar ferramentas de leitura, busca e terminal, mas não pode editar ou escrever arquivos livremente sem permissão.

Esse campo é útil para:

- Fluxos somente leitura.
- Tarefas sensíveis à segurança.
- Auditorias.
- Revisões de código.
- Onboarding em bases de código.
- Situações em que a equipe quer impor limites claros.

Se `allowed-tools` for omitido, a skill não adiciona restrições extras e Claude usa o modelo normal de permissões.

---

### `model`

Campo opcional.

Permite especificar qual modelo Claude deve usar para a skill.

Exemplo:

```
model: sonnet
```

Isso pode ser útil quando uma skill exige um tipo específico de raciocínio, custo, desempenho ou comportamento esperado.

---

### Progressive disclosure

**Progressive disclosure** significa manter apenas as instruções essenciais no `SKILL.md` e colocar materiais adicionais em arquivos separados.

A ideia é evitar que Claude carregue informações demais no contexto sem necessidade.

Em vez de colocar tudo em um único arquivo enorme, a skill pode ser organizada assim:

```
my-skill/
  SKILL.md
  scripts/
  references/
  assets/
```

Onde:

- `scripts/` contém código executável.
- `references/` contém documentação complementar.
- `assets/` contém imagens, templates ou outros arquivos de apoio.

O `SKILL.md` funciona como um guia principal, dizendo a Claude quando deve consultar cada arquivo adicional.

---

### Limite recomendado para `SKILL.md`

A aula recomenda manter o `SKILL.md` com menos de 500 linhas.

Se ele passar disso, provavelmente há conteúdo que deveria ser movido para arquivos separados em:

```
references/
scripts/
assets/
```

Isso melhora:

- Eficiência de contexto.
- Manutenção.
- Organização.
- Clareza das instruções.

---

### Scripts em skills

Scripts podem ser incluídos na pasta da skill e executados por Claude.

A grande vantagem é que Claude **não precisa carregar o conteúdo do script no contexto**. Ele executa o script e apenas a saída gerada consome tokens.

Isso é útil para:

- Validação de ambiente.
- Transformações de dados.
- Operações repetitivas.
- Tarefas em que código testado é mais confiável do que código gerado dinamicamente.
- Padronização de processos.

Ponto importante: no `SKILL.md`, é melhor instruir Claude a **executar o script**, não a lê-lo.

Exemplo conceitual:

```
Run scripts/check-environment.sh to validate the project setup. Do not read the script unless debugging is required.
```

---

## 3. Pontos importantes para certificação

- `name` e `description` são campos obrigatórios no frontmatter.
- `allowed-tools` e `model` são campos opcionais.
- `description` é o campo mais importante para ativação da skill.
- Uma boa `description` responde:
    - O que a skill faz?
    - Quando Claude deve usá-la?
- Se a skill não estiver sendo ativada corretamente, a descrição pode precisar de palavras-chave mais alinhadas à forma como o usuário faz pedidos.
- `allowed-tools` restringe as ferramentas disponíveis quando a skill está ativa.
- `allowed-tools` é útil para fluxos read-only e sensíveis à segurança.
- Se `allowed-tools` for omitido, Claude segue o modelo normal de permissões.
- Skills compartilham a janela de contexto com a conversa.
- O conteúdo do `SKILL.md` é carregado no contexto quando a skill é ativada.
- Progressive disclosure evita carregar informações desnecessárias.
- O `SKILL.md` deve conter apenas instruções essenciais.
- Arquivos de apoio devem ficar em diretórios como:
    - `scripts/`
    - `references/`
    - `assets/`
- Boa prática: manter `SKILL.md` com menos de 500 linhas.
- Scripts podem ser executados sem que seu conteúdo seja carregado no contexto.
- Apenas a saída do script consome tokens.
- Para scripts, a instrução correta é dizer a Claude para executar o script, não para ler o script.

---

## 4. Termos-chave

### `name`

Campo obrigatório que identifica a skill. Deve usar letras minúsculas, números e hífens.

### `description`

Campo obrigatório que informa o que a skill faz e quando Claude deve usá-la.

### `allowed-tools`

Campo opcional que restringe quais ferramentas Claude pode usar quando a skill está ativa.

### `model`

Campo opcional que especifica qual modelo Claude deve usar para executar a skill.

### Frontmatter

Bloco inicial do `SKILL.md`, delimitado por `---`, onde ficam os metadados da skill.

### Semantic matching

Processo pelo qual Claude compara a solicitação do usuário com a descrição da skill para decidir se deve ativá-la.

### Progressive disclosure

Técnica de organização em que apenas o conteúdo essencial fica no `SKILL.md`, enquanto detalhes adicionais ficam em arquivos separados.

### Context window

Janela de contexto disponível para Claude processar a conversa e as instruções carregadas.

### `scripts/`

Diretório usado para armazenar scripts executáveis dentro de uma skill.

### `references/`

Diretório usado para documentação complementar ou material de referência.

### `assets/`

Diretório usado para imagens, templates ou arquivos de apoio.

### Read-only workflow

Fluxo em que Claude pode apenas ler, buscar ou analisar informações, sem modificar arquivos.

---

## 5. Boas práticas

- Escreva descrições claras e específicas.
- Inclua na descrição tanto a função da skill quanto o momento de uso.
- Use palavras-chave parecidas com as frases que você realmente usa ao pedir tarefas.
- Mantenha o `name` curto, descritivo e compatível com o nome do diretório.
- Use `allowed-tools` quando quiser limitar riscos.
- Use restrições de ferramentas em fluxos de auditoria, revisão, onboarding ou análise.
- Não coloque todo o conteúdo da skill no `SKILL.md`.
- Use progressive disclosure para organizar skills maiores.
- Coloque documentação detalhada em `references/`.
- Coloque scripts em `scripts/`.
- Coloque templates, imagens e arquivos de apoio em `assets/`.
- Mantenha o `SKILL.md` com menos de 500 linhas.
- Instrua Claude a carregar arquivos adicionais apenas quando necessário.
- Para scripts, peça que Claude execute o script, em vez de ler seu conteúdo.
- Prefira scripts para operações repetitivas, sensíveis ou que precisam ser consistentes.

---

## 6. Limitações, riscos e cuidados

- Uma descrição vaga pode impedir Claude de ativar a skill corretamente.
- Uma descrição ampla demais pode fazer Claude ativar a skill em situações erradas.
- Um `SKILL.md` muito longo consome muito contexto e dificulta manutenção.
- Colocar muitas referências diretamente no `SKILL.md` reduz a eficiência da conversa.
- `allowed-tools` mal configurado pode bloquear ferramentas necessárias para a tarefa.
- Omitir `allowed-tools` significa que a skill não adicionará restrições extras.
- Scripts devem ser confiáveis, pois podem executar operações no ambiente.
- Mesmo que scripts economizem contexto, sua saída ainda consome tokens.
- Claude deve ser instruído com clareza sobre quando consultar arquivos extras.
- Arquivos de apoio sem instruções claras podem ser ignorados ou usados no momento errado.
- Em workflows de segurança, é importante restringir ferramentas para evitar alterações acidentais.

---

## 7. Resumo para revisão rápida

- `name` e `description` são obrigatórios.
- `allowed-tools` e `model` são opcionais.
- `description` é essencial para Claude saber quando usar a skill.
- Boa descrição responde: o que faz e quando usar.
- `allowed-tools` limita ferramentas disponíveis durante a skill.
- Restrição de ferramentas é útil para segurança e tarefas somente leitura.
- Sem `allowed-tools`, vale o modelo normal de permissões.
- Progressive disclosure mantém o `SKILL.md` enxuto.
- Boa prática: `SKILL.md` com menos de 500 linhas.
- Use `references/` para documentação extra.
- Use `scripts/` para código executável.
- Use `assets/` para imagens, templates e dados auxiliares.
- Scripts podem ser executados sem carregar o código no contexto.
- Apenas a saída do script consome tokens.
- Para scripts, diga a Claude para executar, não ler.

---

## 8. Perguntas simuladas

### Pergunta 1

Quais campos são obrigatórios no frontmatter de uma skill?

A. `model` e `allowed-tools`

B. `name` e `description`

C. `scripts` e `assets`

D. `version` e `author`

**Resposta correta: B**

**Explicação:** Os campos obrigatórios são `name` e `description`. Os campos `allowed-tools` e `model` são opcionais.

---

### Pergunta 2

Qual é o campo mais importante para Claude decidir quando ativar uma skill?

A. `model`

B. `allowed-tools`

C. `description`

D. `assets`

**Resposta correta: C**

**Explicação:** Claude usa a `description` para fazer o matching entre o pedido do usuário e a skill disponível.

---

### Pergunta 3

Uma boa descrição de skill deve responder quais perguntas?

A. Quem criou a skill e qual versão ela usa.

B. O que a skill faz e quando Claude deve usá-la.

C. Quais arquivos foram modificados e quem aprovou.

D. Qual linguagem de programação será usada e qual sistema operacional está ativo.

**Resposta correta: B**

**Explicação:** A descrição deve explicar claramente a função da skill e em quais situações ela deve ser usada.

---

### Pergunta 4

Para que serve o campo `allowed-tools`?

A. Para definir o nome da skill.

B. Para definir quais ferramentas Claude pode usar quando a skill está ativa.

C. Para carregar todos os arquivos da skill automaticamente.

D. Para apagar ferramentas antigas do Claude Code.

**Resposta correta: B**

**Explicação:** `allowed-tools` restringe as ferramentas disponíveis durante a execução da skill.

---

### Pergunta 5

Quando `allowed-tools` é especialmente útil?

A. Em fluxos sensíveis à segurança ou somente leitura.

B. Apenas para mudar o nome da skill.

C. Apenas para carregar imagens.

D. Quando não há necessidade de controle de permissões.

**Resposta correta: A**

**Explicação:** Restringir ferramentas é útil quando você quer impedir alterações acidentais ou limitar ações em tarefas sensíveis.

---

### Pergunta 6

O que acontece se `allowed-tools` for omitido?

A. A skill deixa de funcionar.

B. Claude usa seu modelo normal de permissões.

C. Claude só pode ler arquivos.

D. Claude não pode ativar a skill.

**Resposta correta: B**

**Explicação:** Sem `allowed-tools`, a skill não impõe restrições extras às ferramentas.

---

### Pergunta 7

O que significa progressive disclosure no contexto de skills?

A. Carregar todas as informações possíveis no `SKILL.md`.

B. Manter instruções essenciais no `SKILL.md` e mover detalhes para arquivos separados.

C. Impedir Claude de usar scripts.

D. Substituir `description` por `allowed-tools`.

**Resposta correta: B**

**Explicação:** Progressive disclosure organiza a skill de forma que Claude só leia arquivos adicionais quando necessário.

---

### Pergunta 8

Qual é a recomendação de tamanho para o `SKILL.md`?

A. Menos de 50 linhas.

B. Exatamente 1.000 linhas.

C. Menos de 500 linhas.

D. Mais de 2.000 linhas.

**Resposta correta: C**

**Explicação:** A aula recomenda manter o `SKILL.md` com menos de 500 linhas para economizar contexto e facilitar manutenção.

---

### Pergunta 9

Qual diretório é recomendado para documentação complementar de uma skill?

A. `references/`

B. `bin/`

C. `tmp/`

D. `node_modules/`

**Resposta correta: A**

**Explicação:** O diretório `references/` é usado para materiais de referência e documentação adicional.

---

### Pergunta 10

Qual é uma vantagem de usar scripts dentro de skills?

A. O conteúdo inteiro do script sempre entra no contexto.

B. O script pode executar e apenas sua saída consumir tokens.

C. Scripts substituem a necessidade de `SKILL.md`.

D. Scripts impedem Claude de usar qualquer ferramenta.

**Resposta correta: B**

**Explicação:** Scripts podem ser executados sem carregar seu conteúdo no contexto; apenas o resultado da execução consome tokens.

---

### Pergunta 11

Qual é a instrução recomendada ao usar scripts em uma skill?

A. Pedir para Claude ler todo o script antes de executá-lo.

B. Pedir para Claude executar o script, não ler seu conteúdo.

C. Copiar o script inteiro para o `SKILL.md`.

D. Colocar o script dentro do frontmatter.

**Resposta correta: B**

**Explicação:** Para economizar contexto, a skill deve instruir Claude a executar o script, e não a carregar seu conteúdo.

---

### Pergunta 12

Qual destas opções representa uma boa estrutura para uma skill multifile?

A.

```
my-skill/
  SKILL.md
  scripts/
  references/
  assets/
```

B.

```
my-skill/
  README.md
  package-lock.json
```

C.

```
my-skill/
  CLAUDE.md
```

D.

```
my-skill/
  only-one-large-file.md
```

**Resposta correta: A**

**Explicação:** Uma skill multifile bem estruturada pode ter `SKILL.md`, `scripts/`, `references/` e `assets/`.