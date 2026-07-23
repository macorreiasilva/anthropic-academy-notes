# Execução de código e a API de Arquivos

```
notebooks\31-code_execution_and_the_files_api.ipynb
```

## 1. Ideia central

O conteúdo apresenta duas funcionalidades importantes da Anthropic API que funcionam muito bem juntas:

**Files API** e **Code Execution**.

A ideia central é que você pode:

1. Fazer upload de arquivos para Claude usando a **Files API**.
2. Referenciar esses arquivos por um `file_id`.
3. Permitir que Claude use a ferramenta de **execução de código** para analisar, transformar ou gerar resultados a partir desses arquivos.
4. Receber uma resposta final com análise, código executado, resultados e até arquivos gerados, como gráficos.

Esse fluxo é especialmente útil para tarefas como análise de CSVs, geração de gráficos, processamento de documentos, manipulação de imagens e cálculos matemáticos.

---

## 2. Principais conceitos

### Files API

A **Files API** permite fazer upload de arquivos separadamente antes da chamada principal para Claude.

Em vez de enviar o conteúdo bruto do arquivo dentro da mensagem, como base64, você pode:

1. Fazer upload do arquivo.
2. Receber um objeto de metadados.
3. Usar o `file_id` nas próximas mensagens.

Exemplo conceitual:

```python
file_metadata = upload("streaming.csv")
```

Depois, você referencia o arquivo:

```python
{
    "type": "container_upload",
    "file_id": file_metadata.id
}
```

Isso é útil quando o mesmo arquivo será usado várias vezes ou quando o arquivo é grande demais para ser incluído diretamente em todas as mensagens.

---

### Envio direto de imagens

O primeiro diagrama relembra o fluxo tradicional de envio de imagem:

```text
User Message
├── Image Block: dados brutos da imagem
└── Text Block: pergunta sobre a imagem
```

Claude recebe os dados da imagem e responde com um bloco de texto.

Exemplo:

```text
Usuário envia imagem + “What do you see in this image?”
Claude responde: “I see a tall tree...”
```

Esse fluxo funciona, mas pode ser menos conveniente quando o mesmo arquivo precisa ser reutilizado.

---

### Upload de arquivo com `file_id`

O segundo diagrama mostra uma alternativa mais eficiente:

1. O servidor envia uma **File Upload Request**.
2. Claude retorna um objeto `FileMetadata`, contendo um identificador único.
3. A próxima mensagem usa esse `file_id` em vez dos dados brutos.

Exemplo de estrutura:

```json
{
  "type": "image",
  "source": {
    "type": "file",
    "file_id": "file_011C..."
  }
}
```

Isso evita reenviar o arquivo inteiro em todas as requests.

---

### Code Execution Tool

A **Code Execution Tool** é uma ferramenta server-side fornecida pela Anthropic.

Diferente de ferramentas customizadas, você não precisa implementar a função por conta própria. Basta incluir o schema da ferramenta na request.

Exemplo:

```python
tools=[
    {
        "type": "code_execution_20250522",
        "name": "code_execution"
    }
]
```

Claude pode então gerar e executar código Python em um ambiente isolado.

---

### Ambiente de execução

O código é executado em um **container Docker isolado**.

Características principais:

* sem acesso à internet;
* ambiente separado e controlado;
* pode executar Python;
* pode analisar arquivos enviados;
* pode gerar resultados, como tabelas, gráficos e arquivos;
* Claude pode executar código várias vezes na mesma conversa.

A ausência de internet é importante: Claude não consegue baixar dados externos durante a execução. Por isso, os arquivos precisam ser fornecidos previamente pela Files API.

---

### Code execution com cálculo

Um dos diagramas mostra um exemplo simples:

```text
Usuário: Calculate PI to the 50th decimal place.
Use your code execution tool.
```

Claude então:

1. Cria um bloco de uso da ferramenta.
2. Escreve código para calcular π.
3. Executa o código no container.
4. Recebe o resultado.
5. Responde em texto com o valor calculado.

Fluxo:

```text
Server Tool Use Block → código gerado
Code Execution Tool Result Block → resultado da execução
Text Block → resposta final
```

---

### Code execution com arquivos

Outro exemplo mostra um arquivo CSV chamado `my_data.csv`.

Fluxo:

1. O servidor faz upload do CSV.
2. Recebe um `file_id`.
3. Envia uma mensagem com um `container_upload`.
4. Pede para Claude analisar os dados.
5. Claude executa código no container.
6. Claude retorna análise e resultados.

Exemplo conceitual:

```python
add_user_message(
    messages,
    [
        {
            "type": "text",
            "text": "Analyze the data in the file"
        },
        {
            "type": "container_upload",
            "file_id": file_metadata.id
        }
    ]
)
```

---

### Blocos da resposta

Quando Claude usa Code Execution, a resposta pode conter vários tipos de blocos:

| Bloco                          | Função                             |
| ------------------------------ | ---------------------------------- |
| `TextBlock`                    | Explicações e resposta final       |
| `ServerToolUseBlock`           | Código que Claude decidiu executar |
| `CodeExecutionToolResultBlock` | Resultado da execução do código    |
| `code_execution_output`        | Arquivos gerados, como gráficos    |

Isso permite inspecionar o que Claude fez, qual código executou e quais resultados obteve.

---

### Geração de arquivos

Claude pode criar arquivos dentro do container, como:

* gráficos `.png`;
* relatórios;
* arquivos processados;
* tabelas exportadas;
* saídas intermediárias.

No exemplo do conteúdo, Claude analisa dados de churn de uma plataforma de streaming e gera um gráfico chamado algo como:

```text
churn_analysis_comprehensive.png
```

Esse arquivo pode ser baixado posteriormente usando a Files API.

---

### Exemplo de análise de churn

O dataset mostrado contém dados de usuários de streaming, como:

* `UserID`;
* `SubscriptionTier`;
* `TotalViewingHoursLastMonth`;
* `TopGenre`;
* sessões de binge-watching;
* custo mensal;
* indicador de churn.

Claude usa code execution para analisar os principais fatores de cancelamento.

O resultado inclui:

* análise por plano de assinatura;
* análise por gênero;
* importância das variáveis em modelo preditivo;
* coeficientes de regressão logística;
* histogramas de comportamento;
* gráficos de churn por interações com suporte;
* resumo estatístico final.

Esse exemplo demonstra como Claude pode atuar como um analista de dados que escreve código, executa, interpreta resultados e gera visualizações.

---

## 3. Pontos importantes para certificação

* A **Files API** permite fazer upload de arquivos e reutilizá-los por meio de um `file_id`.
* Enviar arquivos por `file_id` pode ser mais prático do que enviar base64 em todas as mensagens.
* A **Code Execution Tool** permite que Claude execute Python em um container Docker isolado.
* O ambiente de execução **não possui acesso à internet**.
* Como não há internet no container, arquivos devem ser fornecidos via Files API ou blocos de upload.
* Code execution não exige implementação customizada do lado do desenvolvedor.
* Claude pode executar código múltiplas vezes durante uma única resposta.
* A resposta pode conter blocos de texto, blocos de uso de ferramenta e blocos de resultado da execução.
* Claude pode gerar arquivos, como gráficos, que ficam disponíveis para download.
* Files API + Code Execution é uma combinação poderosa para análise de dados.
* É possível enviar CSVs, pedir análise estatística e receber gráficos como saída.
* O desenvolvedor mantém controle sobre os arquivos enviados e recuperados.
* Code execution é útil para cálculos, análise de dados, transformação de documentos, geração de relatórios e processamento de imagens.
* Como o código roda em ambiente isolado, há ganhos de segurança, mas também limitações operacionais.
* A ferramenta é indicada quando a tarefa exige computação real, não apenas geração textual.

---

## 4. Termos-chave

**Files API**
API usada para fazer upload de arquivos e obter um identificador reutilizável.

**File ID**
Identificador único retornado após o upload de um arquivo.

**FileMetadata**
Objeto de metadados que contém informações sobre o arquivo enviado, incluindo o `file_id`.

**Base64**
Formato de codificação usado para incluir arquivos diretamente em mensagens, como imagens ou PDFs.

**Image Block**
Bloco de mensagem usado para enviar uma imagem a Claude.

**Document Block**
Bloco usado para enviar documentos, como PDFs.

**Container Upload Block**
Bloco usado para disponibilizar um arquivo dentro do ambiente de execução de código.

**Code Execution Tool**
Ferramenta que permite a Claude executar código Python em um ambiente isolado.

**Docker Container**
Ambiente isolado onde o código é executado com segurança.

**ServerToolUseBlock**
Bloco que mostra que Claude decidiu usar uma ferramenta server-side.

**CodeExecutionToolResultBlock**
Bloco que contém o resultado da execução do código.

**code_execution_output**
Saída gerada pelo código, podendo incluir arquivos como gráficos.

**Churn**
Cancelamento ou abandono de um serviço por parte do usuário.

**Regressão logística**
Modelo estatístico usado frequentemente para prever eventos binários, como churn ou não churn.

---

## 5. Boas práticas

Use a **Files API** quando o arquivo for grande, reutilizado ou inconveniente de enviar diretamente em base64.

Use **Code Execution** quando a tarefa exigir cálculo real, manipulação de dados ou geração de arquivos.

Boas situações para usar Code Execution:

```text
Analisar CSVs
Gerar gráficos
Calcular estatísticas
Executar transformações em dados
Criar relatórios
Processar imagens
Validar resultados matemáticos
Explorar padrões em datasets
```

Ao trabalhar com arquivos, prefira este fluxo:

```text
Upload do arquivo → recebe file_id → envia file_id para Claude → Claude analisa com code execution
```

Também é importante pedir explicitamente o que você espera da análise.

Exemplo melhor:

```text
Run a detailed analysis to determine major drivers of churn.
Your final output should include at least one detailed plot summarizing your findings.
```

Esse prompt deixa claro:

* a tarefa principal;
* o objetivo analítico;
* o tipo de saída esperada;
* a necessidade de visualização.

---

## 6. Limitações, riscos e cuidados

A principal limitação da Code Execution Tool é que o container **não tem acesso à internet**.

Isso significa que Claude não pode:

* baixar datasets externos;
* chamar APIs externas;
* instalar pacotes da internet, se eles não estiverem disponíveis;
* buscar informações online durante a execução.

Outro cuidado importante é que Claude pode executar código múltiplas vezes. Isso é útil para iteração, mas pode aumentar tempo e custo.

Também é importante revisar os resultados. Mesmo quando Claude executa código, ele ainda pode:

* interpretar dados incorretamente;
* escolher métricas inadequadas;
* gerar gráficos confusos;
* tirar conclusões exageradas;
* não considerar limitações estatísticas;
* usar modelos simples demais para problemas complexos.

Para análise de dados, sempre verifique:

```text
A qualidade do dataset
As colunas usadas
A presença de dados ausentes
A metodologia estatística
As premissas do modelo
Se as conclusões são suportadas pelos dados
```

No caso de arquivos sensíveis, também é necessário cuidado com privacidade e governança:

* não enviar dados pessoais sem necessidade;
* anonimizar datasets quando possível;
* controlar quem pode acessar arquivos enviados;
* verificar políticas internas de retenção e segurança;
* não colocar segredos, senhas ou credenciais nos arquivos.

---

## 7. Resumo para revisão rápida

* Files API permite fazer upload de arquivos e reutilizá-los por `file_id`.
* Isso evita reenviar o arquivo bruto em toda mensagem.
* Code Execution permite que Claude execute Python em um container Docker isolado.
* O container não tem acesso à internet.
* Files API é o principal caminho para levar dados ao ambiente de execução.
* Claude pode analisar arquivos, gerar código, executar e interpretar resultados.
* A resposta pode conter blocos de texto, blocos de ferramenta e blocos de resultado.
* Claude pode gerar arquivos como gráficos e relatórios.
* CSV + Code Execution é um caso de uso forte para análise de dados.
* É possível pedir análises completas, como detecção de fatores de churn.
* Code execution não exige que o desenvolvedor implemente a ferramenta.
* O desenvolvedor deve revisar metodologia, resultados e segurança dos dados.

---

## 8. Perguntas simuladas

### 1. Qual é a principal função da Files API?

A) Executar código Python diretamente
B) Fazer upload de arquivos e permitir referência posterior por `file_id`
C) Criar embeddings vetoriais automaticamente
D) Substituir system prompts

**Resposta correta: B.**

**Explicação:** A Files API permite enviar arquivos previamente e referenciá-los depois usando um identificador único.

---

### 2. Qual é uma vantagem de usar `file_id` em vez de base64?

A) O arquivo pode ser referenciado novamente sem reenviar os dados brutos
B) O arquivo fica público na internet
C) Claude ignora o conteúdo do arquivo
D) O arquivo deixa de contar como entrada

**Resposta correta: A.**

**Explicação:** O `file_id` facilita reutilização e evita incluir o conteúdo completo em cada mensagem.

---

### 3. Onde o código da Code Execution Tool é executado?

A) No navegador do usuário
B) Em um banco de dados vetorial
C) Em um container Docker isolado
D) Diretamente no sistema operacional do usuário

**Resposta correta: C.**

**Explicação:** Claude executa código em um ambiente isolado baseado em container Docker.

---

### 4. Qual é uma limitação importante do ambiente de Code Execution?

A) Ele não executa Python
B) Ele não tem acesso à internet
C) Ele não aceita arquivos CSV
D) Ele só funciona com imagens

**Resposta correta: B.**

**Explicação:** O container não possui acesso à internet, então os dados precisam ser enviados previamente.

---

### 5. Qual combinação é especialmente poderosa para análise de dados?

A) Files API + Code Execution
B) Temperature + Stop Sequences
C) XML tags + Streaming
D) System prompt + Markdown

**Resposta correta: A.**

**Explicação:** A Files API fornece os dados, e Code Execution permite processá-los com código.

---

### 6. O que é um `container_upload`?

A) Um bloco que disponibiliza um arquivo para o ambiente de execução
B) Uma forma de apagar arquivos do servidor
C) Uma ferramenta de busca semântica
D) Um tipo de prompt caching

**Resposta correta: A.**

**Explicação:** O `container_upload` permite que o arquivo enviado seja acessado pelo código executado no container.

---

### 7. O que o `ServerToolUseBlock` representa?

A) O código ou ação que Claude decidiu executar usando uma ferramenta
B) Apenas a resposta final ao usuário
C) Um erro de autenticação
D) Um arquivo PDF enviado pelo usuário

**Resposta correta: A.**

**Explicação:** Esse bloco mostra o uso da ferramenta server-side, como código gerado para execução.

---

### 8. O que pode aparecer em um `CodeExecutionToolResultBlock`?

A) O resultado da execução do código
B) Apenas o prompt original
C) A chave da API
D) A configuração de temperatura

**Resposta correta: A.**

**Explicação:** Esse bloco contém os resultados produzidos pela execução do código.

---

### 9. Qual é um exemplo de arquivo que Claude pode gerar com Code Execution?

A) Um gráfico `.png` com análise de dados
B) Uma chave secreta de API
C) Um novo modelo Claude
D) Uma conexão de internet para o container

**Resposta correta: A.**

**Explicação:** Claude pode gerar visualizações, relatórios e outros arquivos a partir do código executado.

---

### 10. No exemplo de churn, qual era o objetivo da análise?

A) Identificar os principais fatores associados ao cancelamento de usuários
B) Converter imagens em base64
C) Criar um novo schema JSON
D) Fazer web search sobre streaming

**Resposta correta: A.**

**Explicação:** O dataset de streaming foi usado para analisar os principais drivers de churn.

---

### 11. Por que a Files API é importante para Code Execution?

A) Porque o container não tem internet e precisa receber arquivos previamente
B) Porque o container só aceita arquivos públicos
C) Porque Claude não consegue ler CSVs
D) Porque Code Execution não gera saída

**Resposta correta: A.**

**Explicação:** Como o ambiente não acessa a internet, a Files API permite fornecer os dados necessários.

---

### 12. Qual cuidado é importante ao usar Code Execution para análise de dados?

A) Revisar se a metodologia e as conclusões fazem sentido
B) Nunca olhar o código executado
C) Sempre confiar cegamente nos gráficos
D) Evitar usar datasets reais em qualquer situação

**Resposta correta: A.**

**Explicação:** Apesar de executar código, Claude ainda pode interpretar dados de forma inadequada ou tirar conclusões exageradas.

---

### 13. Qual das opções melhor descreve o fluxo Files API + Code Execution?

A) Upload do arquivo, referência por `file_id`, análise com código e resposta final com resultados
B) Criação de embeddings, busca lexical e RRF
C) Envio de system prompt, cache e stop sequence
D) Streaming de tokens sem execução de código

**Resposta correta: A.**

**Explicação:** Esse é o fluxo central apresentado no conteúdo.

---

### 14. Qual é uma boa prática ao pedir uma análise de dados para Claude?

A) Especificar o objetivo e o tipo de saída esperada
B) Enviar apenas “analise isso” sem contexto
C) Não informar qual arquivo usar
D) Impedir Claude de gerar gráficos

**Resposta correta: A.**

**Explicação:** Instruções claras ajudam Claude a produzir uma análise mais útil e alinhada ao objetivo.

---

### 15. Qual frase resume melhor o conteúdo?

A) A Files API permite fornecer arquivos a Claude, e Code Execution permite que Claude escreva e execute código para analisá-los.
B) A Files API substitui completamente o uso de mensagens.
C) Code Execution permite que Claude navegue livremente na internet.
D) Code Execution é usado apenas para responder perguntas simples de texto.

**Resposta correta: A.**

**Explicação:** O foco da aula é mostrar como combinar upload de arquivos e execução de código para tarefas complexas.
