# A ferramenta de edição de texto

```
notebooks\19_the_text_editor_tool.ipynb
```


## 1. Ideia central

O conteúdo apresenta a **Text Editor Tool**, uma ferramenta integrada ao Claude que permite trabalhar com arquivos e diretórios.

A ideia central é:

**Claude já conhece o schema da ferramenta de edição de texto, mas você ainda precisa implementar a função que executa as ações reais no sistema de arquivos.**

Ou seja, Claude sabe **como pedir** para visualizar, criar ou editar arquivos, mas a sua aplicação precisa fornecer o código que realmente realiza essas operações.

---

## 2. Principais conceitos

### O que é a Text Editor Tool

A **Text Editor Tool** é uma ferramenta embutida no Claude que permite operações típicas de um editor de texto.

Ela pode dar a Claude a capacidade de:

* visualizar conteúdo de arquivos ou diretórios;
* visualizar intervalos específicos de linhas em um arquivo;
* substituir texto em um arquivo;
* criar novos arquivos;
* inserir texto em linhas específicas;
* desfazer edições recentes.

Isso aproxima Claude de um assistente capaz de trabalhar como um desenvolvedor dentro de um projeto de código.

---

### Schema embutido, implementação externa

Um ponto muito importante é a diferença entre **schema** e **implementação**.

Com ferramentas customizadas comuns, você precisa criar:

1. a função Python;
2. o JSON schema da ferramenta.

Com a Text Editor Tool, a situação é diferente:

* Claude já tem conhecimento do **schema completo** da ferramenta;
* você ainda precisa implementar a **função real** que executa as operações.

A imagem mostra essa ideia:

```text
text_editor_tool_schema  →  função real que você precisa escrever
```

Ou seja, Claude sabe como solicitar ações como “ver arquivo” ou “substituir texto”, mas o sistema precisa ter uma função capaz de interpretar essa solicitação e modificar o arquivo de fato.

---

### Capacidades principais da ferramenta

A Text Editor Tool permite:

| Capacidade                      | O que significa                              |
| ------------------------------- | -------------------------------------------- |
| View file or directory contents | Ver o conteúdo de arquivos ou diretórios     |
| View specific range of lines    | Ler apenas um intervalo específico de linhas |
| Replace text in a file          | Substituir um trecho de texto                |
| Create new files                | Criar arquivos novos                         |
| Insert text at specific lines   | Inserir texto em linhas específicas          |
| Undo recent edits               | Desfazer edições recentes                    |

Essas operações são úteis para construir aplicações em que Claude precisa analisar, modificar ou gerar arquivos.

---

### Schema stub

Mesmo que Claude conheça o schema completo internamente, ainda é necessário enviar um pequeno **schema stub** na chamada.

Exemplo do conteúdo:

```python
def get_text_edit_schema(model):
    if model.startswith("claude-3-7-sonnet"):
        return {
            "type": "text_editor_20250124",
            "name": "str_replace_editor",
        }
    elif model.startswith("claude-3-5-sonnet"):
        return {
            "type": "text_editor_20241022",
            "name": "str_replace_editor",
        }
```

Esse pequeno schema informa a Claude que a ferramenta de edição de texto está disponível.

Depois disso, Claude expande internamente essa referência para a especificação completa da ferramenta.

---

### Versões da ferramenta

O conteúdo destaca que a string de versão da ferramenta depende do modelo usado.

Exemplos:

```python
"text_editor_20250124"
```

para alguns modelos Claude 3.7 Sonnet.

E:

```python
"text_editor_20241022"
```

para alguns modelos Claude 3.5 Sonnet.

O ponto importante para prova é: **a versão do schema da ferramenta pode variar conforme o modelo**, então é necessário consultar a documentação apropriada ao implementar.

---

### Exemplo prático: abrir e resumir um arquivo

Se você pedir:

```text
Open the ./main.py file and summarize its contents.
```

Claude pode:

1. chamar a Text Editor Tool para visualizar `main.py`;
2. ler o conteúdo do arquivo;
3. responder com um resumo.

Esse exemplo mostra uso de leitura.

---

### Exemplo prático: editar e criar arquivos

Se você pedir:

```text
Open the ./main.py file and write out a function to calculate pi to the 5th digit. Then create a ./test.py file to test your implementation.
```

Claude pode:

1. abrir `main.py`;
2. substituir ou editar o conteúdo;
3. criar uma função para calcular π até a 5ª casa;
4. criar `test.py`;
5. escrever testes para validar a implementação.

Esse exemplo mostra uso mais avançado: leitura, edição e criação de arquivos.

---

## 3. Pontos importantes para certificação

* A Text Editor Tool é uma ferramenta integrada ao Claude.
* Ela permite trabalhar com arquivos e diretórios.
* Claude conhece o schema da ferramenta, mas a aplicação precisa implementar a função real.
* A ferramenta pode visualizar arquivos, editar texto, criar arquivos e desfazer edições.
* Mesmo com schema embutido, é necessário enviar um pequeno schema stub.
* O campo `type` do schema muda conforme a versão/modelo.
* O nome usado no exemplo é `"str_replace_editor"`.
* A ferramenta permite construir experiências parecidas com editores de código assistidos por IA.
* A implementação deve controlar acesso, permissões e segurança no sistema de arquivos.
* Claude não deve ter acesso irrestrito a todos os arquivos do sistema.
* Essa ferramenta é útil quando sua aplicação precisa dar a Claude capacidade programática de manipular arquivos.

---

## 4. Termos-chave

**Text Editor Tool**
Ferramenta integrada ao Claude para visualizar, criar e editar arquivos e diretórios.

**Schema**
Descrição estruturada da ferramenta, usada por Claude para entender como solicitar ações.

**Schema stub**
Pequena especificação enviada na chamada para ativar a ferramenta embutida.

**Implementation**
Código real escrito pela aplicação para executar as operações solicitadas pela ferramenta.

**`str_replace_editor`**
Nome da ferramenta de edição de texto mostrado no exemplo.

**Tool version string**
String que identifica a versão da ferramenta, como `text_editor_20250124`.

**View file**
Operação de visualizar o conteúdo de um arquivo.

**Replace text**
Operação de substituir um trecho de texto em um arquivo.

**Insert text**
Operação de inserir texto em uma linha específica.

**Undo edits**
Capacidade de desfazer alterações recentes.

**File system**
Sistema de arquivos onde diretórios e arquivos são armazenados.

---

## 5. Boas práticas

Use a Text Editor Tool quando quiser que Claude interaja com arquivos de forma estruturada.

Boas práticas importantes:

* limitar quais diretórios Claude pode acessar;
* validar caminhos de arquivos;
* impedir acesso a arquivos sensíveis;
* registrar alterações feitas;
* permitir desfazer edições;
* revisar alterações antes de aplicar em produção;
* tratar erros com mensagens claras;
* testar a implementação da ferramenta separadamente;
* manter o schema e a função alinhados;
* consultar a versão correta da ferramenta para o modelo usado.

Também é recomendável começar com operações simples, como leitura de arquivos, antes de permitir alterações destrutivas ou criação automática de arquivos.

---

## 6. Limitações, riscos e cuidados

A Text Editor Tool é poderosa, mas exige cuidado.

O principal risco é dar a Claude acesso amplo demais ao sistema de arquivos.

Sem controles, Claude poderia tentar:

* ler arquivos sensíveis;
* modificar arquivos importantes;
* sobrescrever código;
* criar arquivos em locais indevidos;
* alterar configurações críticas.

Por isso, a aplicação deve controlar permissões, validar caminhos e restringir o ambiente.

Outro cuidado é que o schema embutido não substitui a implementação. Se você não escrever a função que processa as solicitações da ferramenta, Claude pode pedir uma ação, mas nada será executado.

Também é importante lembrar que diferentes modelos podem exigir versões diferentes do schema stub. Usar uma versão incorreta pode causar falhas ou comportamento inesperado.

---

## 7. Resumo para revisão rápida

* Text Editor Tool é embutida no Claude.
* Permite ler, criar e editar arquivos.
* Claude conhece o schema, mas você implementa a função real.
* Pode visualizar arquivos e diretórios.
* Pode visualizar intervalos de linhas.
* Pode substituir texto.
* Pode criar arquivos.
* Pode inserir texto em linhas específicas.
* Pode desfazer edições recentes.
* É necessário enviar um schema stub.
* A versão do schema depende do modelo.
* Use controles de segurança para limitar acesso a arquivos.
* A ferramenta é útil para aplicações que precisam de edição programática de arquivos.

---

## 8. Perguntas simuladas

### 1. O que é a Text Editor Tool?

A) Uma ferramenta para gerar imagens
B) Uma ferramenta integrada ao Claude para trabalhar com arquivos e diretórios
C) Um parâmetro de temperature
D) Um tipo de API key

**Resposta correta: B.**

**Explicação:** A Text Editor Tool permite que Claude solicite operações de leitura, criação e edição de arquivos.

---

### 2. Qual destas ações a Text Editor Tool pode realizar?

A) Visualizar conteúdo de arquivos
B) Alterar o preço da API
C) Treinar um novo modelo
D) Criar uma API key automaticamente

**Resposta correta: A.**

**Explicação:** A ferramenta pode visualizar arquivos e diretórios, além de editar e criar arquivos.

---

### 3. O que significa dizer que o schema é embutido no Claude?

A) Claude já conhece a especificação da ferramenta
B) Claude executa automaticamente qualquer arquivo do sistema
C) A aplicação não precisa fazer nenhuma implementação
D) O schema fica salvo no `.env`

**Resposta correta: A.**

**Explicação:** Claude conhece o formato da ferramenta, mas ainda precisa que a aplicação implemente as ações reais.

---

### 4. Mesmo com schema embutido, o que a aplicação ainda precisa fornecer?

A) A implementação real da ferramenta
B) Um novo modelo treinado do zero
C) Uma resposta em XML
D) Um arquivo sem permissões

**Resposta correta: A.**

**Explicação:** Claude sabe como solicitar a ação, mas o servidor precisa executar a operação no sistema de arquivos.

---

### 5. Qual é a função do schema stub?

A) Ativar ou declarar a ferramenta embutida na chamada à API
B) Apagar arquivos automaticamente
C) Substituir a função real
D) Armazenar lembretes

**Resposta correta: A.**

**Explicação:** O schema stub informa a Claude que a Text Editor Tool está disponível.

---

### 6. No exemplo, qual é o nome da ferramenta?

A) `weather_tool`
B) `str_replace_editor`
C) `reminder_creator`
D) `json_validator`

**Resposta correta: B.**

**Explicação:** O schema stub usa `"name": "str_replace_editor"`.

---

### 7. Por que a versão do schema importa?

A) Porque ela pode variar conforme o modelo Claude usado
B) Porque define o nome do usuário
C) Porque substitui o arquivo `.env`
D) Porque impede qualquer edição

**Resposta correta: A.**

**Explicação:** O conteúdo mostra versões diferentes, como `text_editor_20250124` e `text_editor_20241022`, dependendo do modelo.

---

### 8. Qual é um exemplo de uso da Text Editor Tool?

A) Abrir `main.py` e resumir seu conteúdo
B) Escolher automaticamente um plano de assinatura
C) Fazer login no console da Anthropic
D) Criar uma chave secreta pública

**Resposta correta: A.**

**Explicação:** Claude pode usar a ferramenta para visualizar um arquivo e depois explicar o conteúdo.

---

### 9. Qual é um risco ao usar a Text Editor Tool?

A) Claude pode receber acesso amplo demais ao sistema de arquivos se não houver controle
B) Claude deixa de conseguir responder texto
C) A API key fica obrigatoriamente pública
D) A ferramenta impede criação de arquivos

**Resposta correta: A.**

**Explicação:** É essencial restringir permissões e validar caminhos para evitar acesso ou edição indevida.

---

### 10. Qual frase resume melhor a Text Editor Tool?

A) Claude já sabe solicitar operações de edição de texto, mas sua aplicação precisa executar essas operações com segurança.
B) Claude treina automaticamente um novo modelo sempre que edita um arquivo.
C) A Text Editor Tool substitui todos os prompts.
D) A ferramenta serve apenas para consultar clima atual.

**Resposta correta: A.**

**Explicação:** O schema é conhecido por Claude, mas a implementação real e segura fica sob responsabilidade da aplicação.
