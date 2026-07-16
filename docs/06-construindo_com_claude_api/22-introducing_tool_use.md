# Introdução ao uso de ferramentas

## 1. Ideia central

O conteúdo introduz o conceito de **tool use**, ou uso de ferramentas, no Claude.

A ideia central é que Claude, por padrão, responde com base no que aprendeu durante o treinamento e **não acessa automaticamente informações externas, atuais ou em tempo real**.

Com **tools**, Claude pode solicitar dados externos de forma estruturada, e o seu servidor pode buscar essas informações em APIs, bancos de dados ou outros sistemas. Depois, Claude usa esses dados para gerar uma resposta final mais precisa e atualizada.

Em resumo:

**Tool use transforma Claude de um modelo baseado apenas em conhecimento interno em um assistente capaz de trabalhar com dados externos e atuais.**

---

## 2. Principais conceitos

### O problema sem ferramentas

Sem ferramentas, Claude não consegue acessar dados atuais.

Por exemplo, se o usuário perguntar:

```text
What's the weather in San Francisco, California?
```

Claude não tem como saber o clima atual, porque esse tipo de informação muda constantemente.

Nesse caso, ele teria que responder algo como:

```text
I don't have access to up-to-date weather information.
```

Isso limita a experiência do usuário quando a aplicação precisa de dados em tempo real.

---

### O que são tools

**Tools** são funções externas que sua aplicação disponibiliza para Claude usar indiretamente.

Elas podem buscar informações em:

* APIs externas;
* bancos de dados;
* sistemas internos;
* serviços de clima;
* plataformas financeiras;
* sistemas de estoque;
* calendários;
* CRMs;
* mecanismos de busca;
* ferramentas próprias do seu backend.

Importante: Claude não executa a ferramenta sozinho. Ele **pede** os dados necessários, e o **seu servidor** executa o código para buscar esses dados.

---

### Como o tool use funciona

O fluxo segue um padrão de ida e volta entre Claude e o servidor.

A imagem mostra quatro etapas principais:

1. **Pergunta inicial com instruções de ferramenta**
   O servidor envia a pergunta do usuário para Claude junto com instruções sobre como obter dados extras.

2. **Claude solicita dados adicionais**
   Claude percebe que precisa de informação externa e pede dados específicos.

3. **Servidor busca os dados**
   O servidor executa código para buscar as informações solicitadas, por exemplo chamando uma API de clima.

4. **Claude gera a resposta final**
   O servidor envia os dados de volta para Claude, e Claude usa essas informações para responder ao usuário.

O fluxo visual é:

```text
Servidor → Claude: pergunta + instruções de como obter dados extras
Claude → Servidor: solicitação de dados específicos
Servidor: executa código para buscar os dados
Servidor → Claude: envia os dados obtidos
Claude → Servidor: resposta final enriquecida com os dados
```

---

### Exemplo: clima atual

O exemplo usado é uma pergunta sobre o clima em San Francisco.

Sem tools, Claude não consegue responder com precisão sobre o clima atual.

Com tools, o fluxo seria:

1. Usuário pergunta o clima atual.
2. Claude identifica que precisa de dados em tempo real.
3. Claude solicita dados de clima para San Francisco.
4. O servidor chama uma API de previsão do tempo.
5. O servidor envia os dados obtidos para Claude.
6. Claude responde com base nas informações atualizadas.

Esse exemplo demonstra que tools são úteis quando a resposta depende de informação externa ou dinâmica.

---

### Papel do servidor

O servidor é essencial no tool use.

Ele é responsável por:

* receber a solicitação de Claude;
* decidir se a ferramenta pode ser executada;
* chamar APIs externas;
* consultar bancos de dados;
* retornar os dados para Claude;
* controlar segurança, permissões e validação.

Claude não deve ter acesso livre e direto a sistemas externos. A aplicação precisa intermediar esse acesso.

---

### Resposta final aumentada com dados externos

Depois que o servidor retorna os dados, Claude gera uma resposta final combinando:

* a pergunta original do usuário;
* os dados externos recuperados;
* o contexto da conversa;
* as instruções do sistema ou da aplicação.

Esse tipo de resposta é chamado de resposta **aumentada** ou **enriquecida** por dados externos.

---

## 3. Pontos importantes para certificação

* Claude não acessa automaticamente dados em tempo real.
* Tool use permite que Claude trabalhe com informações externas.
* Tools conectam Claude a APIs, bancos de dados e sistemas externos.
* Claude solicita os dados; o servidor executa a ferramenta.
* O servidor é responsável por buscar os dados extras.
* O fluxo envolve múltiplas etapas entre servidor e Claude.
* Tool use é útil para clima, preços, estoque, dados internos, eventos atuais e consultas dinâmicas.
* A resposta final é gerada depois que Claude recebe os dados externos.
* Tool use melhora a capacidade de Claude em aplicações reais.
* Tools devem ser controladas com segurança, permissões e validação.
* Claude não deve executar livremente ações externas sem controle do servidor.
* Tool use transforma Claude em um assistente dinâmico, não apenas em uma base estática de conhecimento.

---

## 4. Termos-chave

**Tool use**
Capacidade de Claude solicitar e usar dados ou ações externas por meio de ferramentas controladas pela aplicação.

**Tool**
Função externa disponibilizada pela aplicação, como buscar clima, consultar banco de dados ou chamar uma API.

**External data**
Informação que vem de fora do modelo, como dados atuais, registros internos ou respostas de APIs.

**Real-time data**
Dados atualizados em tempo real ou quase real, como clima, preços, status de pedidos ou cotações.

**Server**
Backend da aplicação que executa ferramentas, busca dados e controla o acesso a sistemas externos.

**Data retrieval**
Processo de buscar dados em uma fonte externa.

**API**
Interface usada para consultar ou interagir com outro sistema.

**Dynamic response**
Resposta gerada com base em dados atualizados ou variáveis.

**Tool request**
Solicitação feita por Claude indicando quais dados ou ação externa são necessários.

**Augmented response**
Resposta final enriquecida com dados externos recuperados por uma ferramenta.

---

## 5. Boas práticas

Use tool use quando a resposta depender de dados que Claude não possui internamente.

Bons casos de uso incluem:

* clima atual;
* preços de ações;
* status de pedidos;
* informações de banco de dados;
* dados de usuários;
* disponibilidade em calendário;
* consultas a sistemas internos;
* eventos recentes;
* resultados de busca;
* cálculos ou execuções específicas.

O servidor deve ser responsável por executar a ferramenta, não o cliente diretamente.

Também é importante definir ferramentas com descrições claras, para que Claude saiba:

* quando usar a ferramenta;
* quais dados pedir;
* quais parâmetros fornecer;
* que tipo de retorno esperar.

Além disso, valide os dados antes de usá-los e aplique controles de segurança, principalmente quando a ferramenta puder consultar dados sensíveis ou executar ações.

---

## 6. Limitações, riscos e cuidados

Tool use aumenta o poder da aplicação, mas também aumenta a responsabilidade.

O principal cuidado é que Claude não deve ter acesso irrestrito a sistemas externos. O servidor precisa validar:

* se a ferramenta pode ser chamada;
* se os parâmetros são seguros;
* se o usuário tem permissão;
* se a ação é permitida;
* se os dados retornados são confiáveis.

Outro risco é retornar dados incorretos ou desatualizados de uma API externa. Claude pode formular uma resposta convincente com base nesses dados, então a qualidade da fonte externa é importante.

Também é necessário cuidar de privacidade. Se ferramentas acessam dados pessoais, registros internos ou informações sensíveis, a aplicação deve limitar o acesso e registrar o uso adequadamente.

Tool use também pode aumentar latência, porque a resposta final depende de chamadas adicionais para sistemas externos.

---

## 7. Resumo para revisão rápida

* Claude não acessa dados atuais automaticamente.
* Tools permitem buscar informações externas.
* O servidor executa as ferramentas.
* Claude solicita os dados necessários.
* O servidor busca os dados em APIs, bancos ou sistemas.
* Claude usa os dados retornados para responder.
* Exemplo clássico: clima atual.
* Sem tools, Claude não sabe o clima em tempo real.
* Com tools, o servidor busca o clima e Claude responde com base nele.
* Tool use permite respostas dinâmicas e atualizadas.
* É essencial controlar segurança, permissões e validação.
* Tool use conecta Claude ao mundo externo.

---

## 8. Perguntas simuladas

### 1. Qual problema o tool use resolve?

A) A falta de acesso automático de Claude a dados externos e atuais
B) A necessidade de escrever prompts claros
C) O limite de caracteres em arquivos Markdown
D) A criação automática de API keys

**Resposta correta: A.**

**Explicação:** Tool use permite que Claude solicite dados externos, como clima atual, preços ou informações de bancos de dados.

---

### 2. Sem tools, o que Claude provavelmente responderia a uma pergunta sobre clima atual?

A) Uma previsão exata em tempo real
B) Que não tem acesso a informações atualizadas de clima
C) Que vai consultar automaticamente a internet
D) Que o servidor não é necessário

**Resposta correta: B.**

**Explicação:** Por padrão, Claude não acessa dados em tempo real.

---

### 3. Quem executa o código para buscar dados externos?

A) O próprio Claude diretamente
B) O servidor da aplicação
C) O navegador do usuário final sem controle
D) O prompt de sistema sozinho

**Resposta correta: B.**

**Explicação:** Claude solicita os dados, mas o servidor executa a ferramenta e retorna as informações.

---

### 4. Qual é a ordem geral do fluxo de tool use?

A) Claude executa código → usuário aprova → servidor ignora
B) Pergunta inicial → Claude solicita dados → servidor busca dados → Claude responde com dados enriquecidos
C) Usuário envia API key → Claude salva permanentemente → ferramenta some
D) Servidor responde sem consultar Claude

**Resposta correta: B.**

**Explicação:** Tool use envolve uma troca estruturada entre Claude e servidor para obter dados externos e gerar a resposta final.

---

### 5. Qual é um bom exemplo de uso de tools?

A) Perguntar a definição de uma palavra comum
B) Perguntar o clima atual em uma cidade
C) Pedir para escrever um poema simples
D) Pedir para resumir um texto já fornecido

**Resposta correta: B.**

**Explicação:** Clima atual depende de dados externos e em tempo real, tornando tools apropriadas.

---

### 6. O que significa dizer que a resposta final é “augmented by extra data”?

A) A resposta foi enriquecida com dados externos recuperados por uma ferramenta
B) A resposta foi reduzida para economizar tokens
C) A resposta ignorou a pergunta original
D) A resposta foi gerada sem contexto

**Resposta correta: A.**

**Explicação:** Claude usa os dados externos fornecidos pelo servidor para gerar uma resposta mais completa e atualizada.

---

### 7. Qual benefício NÃO é diretamente associado ao tool use?

A) Acesso a dados em tempo real
B) Integração com sistemas externos
C) Respostas dinâmicas
D) Garantia de que toda resposta será sempre correta

**Resposta correta: D.**

**Explicação:** Tools melhoram acesso a dados, mas não garantem perfeição. Ainda é preciso validar fontes e respostas.

---

### 8. Por que o servidor é importante no tool use?

A) Porque controla a execução das ferramentas, valida permissões e busca dados externos
B) Porque substitui totalmente Claude
C) Porque impede qualquer uso de APIs
D) Porque elimina a necessidade de prompts

**Resposta correta: A.**

**Explicação:** O servidor é a camada segura que executa ferramentas e controla acesso a sistemas externos.

---

### 9. Qual é um risco ao usar tools?

A) Aumentar responsabilidades de segurança e privacidade
B) Impedir Claude de responder a qualquer pergunta
C) Remover a necessidade de validar dados
D) Garantir automaticamente respostas sem erro

**Resposta correta: A.**

**Explicação:** Tools podem acessar dados sensíveis ou executar ações, então exigem validação, permissões e controle.

---

### 10. Qual frase resume melhor tool use?

A) Tool use permite que Claude solicite dados externos, que o servidor busca e devolve para gerar respostas mais atualizadas.
B) Tool use faz Claude lembrar automaticamente todas as conversas.
C) Tool use é apenas outro nome para temperature.
D) Tool use serve apenas para formatar respostas em XML.

**Resposta correta: A.**

**Explicação:** O conceito central é conectar Claude a dados ou sistemas externos por meio de ferramentas controladas pela aplicação.
