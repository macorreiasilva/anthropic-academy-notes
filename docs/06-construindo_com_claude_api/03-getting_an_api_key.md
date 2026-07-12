# Como obter uma chave de API

## 1. Ideia central

O conteúdo explica como **criar uma API key no Console da Anthropic (**[https://platform.claude.com/dashboard](https://platform.claude.com/dashboard)).

A API key é a credencial usada para autenticar chamadas à API da Anthropic. Sem ela, sua aplicação não consegue enviar requisições para os modelos Claude.

A sequência geral é:

**Anthropic Console → Get API Keys → Create Key → escolher workspace e nome → copiar a chave**

O ponto mais importante é: **a chave só aparece uma vez**, então ela deve ser copiada e armazenada com segurança imediatamente.

---

## 2. Principais conceitos

### 1. Anthropic API Console

O **Anthropic API Console** é a interface web onde o usuário gerencia recursos da API, incluindo criação e administração de chaves.

É nele que você acessa sua conta Anthropic e cria uma API key para usar nos próximos passos do curso.

---

### 2. Get API Keys

Depois de entrar no console, o usuário deve clicar no botão **“Get API Keys”**.

Esse botão leva à área de gerenciamento de chaves da API.

---

### 3. Create Key

Na página de API keys, o próximo passo é clicar em **“Create Key”**.

Esse botão inicia o processo de criação de uma nova chave.

---

### 4. Workspace

Durante a criação da chave, é necessário escolher um **workspace**.

No exemplo do curso, o workspace escolhido é:

**Default**

O workspace ajuda a organizar recursos, projetos e credenciais dentro da conta.

---

### 5. Nome da chave

Também é necessário dar um nome à API key.

No exemplo, o nome usado é:

**Anthropic Course**

Esse nome não é a chave em si. Ele serve apenas para identificação, especialmente quando há várias chaves criadas.

---

### 6. Copiar a chave

Depois de criar a chave, ela aparece em uma janela pop-up.

Esse é um ponto crítico:

**A API key só será exibida uma vez.**

Por isso, o usuário deve copiá-la imediatamente e guardá-la em um local seguro.

Se a janela for fechada sem copiar a chave, a recomendação do curso é:

**deletar a chave antiga e gerar uma nova.**

---

## 3. Pontos importantes para certificação

- A API key é usada para **autenticar chamadas à API da Anthropic**.
- A chave deve ser criada no **Anthropic API Console**.
- O botão **Get API Keys** leva à área de gerenciamento de chaves.
- O botão **Create Key** cria uma nova chave.
- A chave deve estar associada a um **workspace**.
- O nome da chave serve para **organização e identificação**, não para autenticação.
- A API key **só aparece uma vez**.
- Se o usuário fechar a janela sem copiar a chave, deve **deletar a chave antiga e criar outra**.
- A API key deve ser tratada como **segredo**.
- A chave não deve ser colocada em código público, front-end, repositórios GitHub ou prints compartilhados.

---

## 4. Termos-chave

**API key**

Chave secreta usada para autenticar uma aplicação ao chamar a API da Anthropic.

**Anthropic API Console**

Painel web onde o usuário gerencia acesso, chaves, configurações e recursos da API.

**Get API Keys**

Botão ou área do console usada para acessar o gerenciamento de chaves.

**Create Key**

Ação usada para gerar uma nova API key.

**Workspace**

Espaço organizacional dentro da conta Anthropic. Pode separar projetos, ambientes ou equipes.

**Default workspace**

Workspace padrão usado no exemplo do curso.

**Key name**

Nome dado à chave para facilitar sua identificação.

**Secret**

Informação sensível que deve ser protegida, como uma API key.

**Credential**

Credencial de acesso usada para provar identidade ou autorização em um sistema.

---

## 5. Boas práticas

- Copiar a API key assim que ela for exibida.
- Guardar a chave em um local seguro.
- Usar nomes descritivos para as chaves, como **Anthropic Course**, **Production App** ou **Test Environment**.
- Criar chaves diferentes para ambientes diferentes, como desenvolvimento, teste e produção.
- Nunca publicar a chave em repositórios de código.
- Nunca colocar a chave no front-end.
- Nunca compartilhar a chave em capturas de tela, mensagens ou documentos públicos.
- Revogar ou deletar uma chave se houver suspeita de vazamento.
- Se a chave for perdida, criar uma nova em vez de tentar recuperar a antiga.

---

## 6. Limitações, riscos e cuidados

O principal cuidado é que a **API key só aparece uma vez**.

Isso significa que, depois de fechar a janela, você não conseguirá visualizar a mesma chave novamente. Se não tiver copiado, será necessário apagar a chave antiga e criar outra.

Outro risco importante é o **vazamento da API key**. Quem tiver acesso à chave pode fazer chamadas à API em nome da sua conta, o que pode gerar custos, abuso ou exposição de dados.

Também é importante lembrar que o nome da chave, como **Anthropic Course**, não protege nem autentica a chamada. Ele é apenas um rótulo administrativo.

A API key deve ser armazenada como variável de ambiente ou em um gerenciador de segredos, não diretamente no código.

---

## 7. Resumo para revisão rápida

- Entre no **Anthropic API Console**.
- Clique em **Get API Keys**.
- Clique em **Create Key**.
- Escolha o workspace **Default**.
- Dê um nome à chave, como **Anthropic Course**.
- Copie a chave imediatamente.
- A API key aparece **somente uma vez**.
- Se fechar a janela sem copiar, delete a chave e crie outra.
- A API key é uma credencial secreta.
- Nunca exponha a chave no front-end ou em repositórios públicos.

---

## 8. Perguntas simuladas

### 1. Para que serve uma API key da Anthropic?

A) Para escolher automaticamente o melhor modelo Claude

B) Para autenticar chamadas feitas à API da Anthropic

C) Para substituir o prompt do usuário

D) Para aumentar a janela de contexto do modelo

**Resposta correta: B.**

**Explicação:** A API key é uma credencial usada para identificar e autenticar quem está chamando a API.

---

### 2. Onde a API key é criada?

A) Diretamente no código do front-end

B) No Anthropic API Console

C) Dentro da resposta do modelo Claude

D) No navegador do usuário final

**Resposta correta: B.**

**Explicação:** A chave é criada no painel da Anthropic, chamado Anthropic API Console.

---

### 3. Qual botão leva à área de gerenciamento de chaves?

A) Start Chat

B) Get API Keys

C) Run Model

D) New Prompt

**Resposta correta: B.**

**Explicação:** O botão **Get API Keys** leva à seção onde as chaves podem ser gerenciadas.

---

### 4. O que o botão “Create Key” faz?

A) Envia uma mensagem para Claude

B) Cria uma nova API key

C) Apaga todas as chaves existentes

D) Abre a documentação do curso

**Resposta correta: B.**

**Explicação:** O botão **Create Key** inicia a criação de uma nova chave de API.

---

### 5. No exemplo do curso, qual workspace deve ser usado?

A) Production

B) Claude Workspace

C) Default

D) Admin Only

**Resposta correta: C.**

**Explicação:** O exemplo orienta criar a chave no workspace **Default**.

---

### 6. Qual nome é usado para a chave no exemplo?

A) Claude Secret

B) Anthropic Course

C) Main API

D) Test Key 01

**Resposta correta: B.**

**Explicação:** O curso usa o nome **Anthropic Course** para identificar a chave.

---

### 7. O que acontece se você fechar a janela sem copiar a API key?

A) Você pode visualizar a mesma chave novamente depois

B) A chave é enviada automaticamente por e-mail

C) Você deve deletar a chave antiga e gerar uma nova

D) A chave continua visível no histórico do navegador

**Resposta correta: C.**

**Explicação:** A chave só aparece uma vez. Se a janela for fechada sem copiar, a solução é apagar a chave e criar outra.

---

### 8. Qual é uma boa prática de segurança com API keys?

A) Colocar a chave no código do front-end

B) Compartilhar a chave com todos os usuários do app

C) Armazenar a chave com segurança e não expô-la publicamente

D) Usar a mesma chave em todos os projetos públicos

**Resposta correta: C.**

**Explicação:** A API key deve ser tratada como segredo e protegida contra exposição.

---

### 9. O nome da API key serve principalmente para quê?

A) Definir o preço da API

B) Identificar a chave no console

C) Criptografar a resposta do modelo

D) Escolher automaticamente entre Opus, Sonnet e Haiku

**Resposta correta: B.**

**Explicação:** O nome da chave é um rótulo administrativo para facilitar a organização.

---

### 10. Qual afirmação é mais correta sobre a API key?

A) Ela pode ser pública porque não gera custos

B) Ela deve ficar escondida e protegida

C) Ela substitui o campo `model` na requisição

D) Ela só é usada para chamadas feitas pelo navegador

**Resposta correta: B.**

**Explicação:** A API key é uma credencial sensível e deve ser protegida como segredo.