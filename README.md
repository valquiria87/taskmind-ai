# AI Task Manager Agent — Google ADK + Gemini + Trello

Agente inteligente de produtividade desenvolvido em Python com **Google Agent Development Kit (ADK)**, **Gemini 3.5 Flash** e integração com a API do **Trello**, utilizando a biblioteca `py-trello`.

O projeto foi desenvolvido como solução para o desafio da Digital Innovation One (DIO). Seu objetivo é organizar as tarefas diárias por meio de uma conversa em linguagem natural, registrando e atualizando cartões automaticamente em um quadro Kanban do Trello.

## Funcionalidades

O agente interpreta as solicitações do usuário e utiliza ferramentas Python (*Function Calling*) para executar ações diretamente no Trello.

| Ferramenta | Funcionalidade |
|---|---|
| `get_today_date` | Consulta a data atual e o dia da semana em português. |
| `create_task` | Cria um cartão na lista **A fazer**, com título, descrição e prazo de entrega. |
| `list_tasks` | Lista todas as tarefas ou filtra por status: A fazer, Em andamento ou Concluído. |
| `move_task` | Move um cartão entre as listas do quadro conforme o progresso informado pelo usuário. |
| `delete_task` | Exclui permanentemente um cartão mediante solicitação do usuário. |

### Como o agente funciona

1. Inicia a conversa apresentando a data atual e perguntando quais tarefas o usuário deseja organizar.
2. Recebe as tarefas em linguagem natural.
3. Cria um cartão para cada tarefa na lista **A fazer**.
4. Consulta e apresenta as tarefas existentes, de forma geral ou por status.
5. Atualiza a posição dos cartões conforme o usuário informa que iniciou ou concluiu uma atividade.
6. Remove cartões quando solicitado.

## Tecnologias utilizadas

- Python 3.10 ou superior
- Google Agent Development Kit (ADK)
- Gemini 3.5 Flash
- Trello API
- Biblioteca `py-trello`
- `python-dotenv` para gerenciamento das variáveis de ambiente

## Configuração do Trello

### 1. Criar o quadro Kanban

No Trello, crie um quadro chamado **DIO**.

Crie as três listas com estes nomes:

- A fazer
- Em andamento
- Concluído

Caso utilize outro nome para o quadro, será possível configurá-lo posteriormente pela variável `TRELLO_BOARD_NAME`.

### 2. Criar um aplicativo no Trello

1. Acesse o [portal de Power-Ups do Trello](https://trello.com/power-ups/admin/).
2. Crie um novo aplicativo (*New*).
3. Informe um nome, por exemplo, `TaskAgentDIO`.
4. Selecione o Workspace e salve.
5. Na página do aplicativo, copie a **API Key** e o **API Secret**.

### 3. Gerar o token de acesso

Acesse o endereço abaixo, substituindo `SUA_API_KEY_AQUI` pela sua chave:

```text
https://trello.com/1/authorize?expiration=never&name=TaskAgentDIO&scope=read,write&response_type=token&key=SUA_API_KEY_AQUI
```

Autorize a aplicação e copie o token gerado.

**Importante:** a API Key, o API Secret e o Token são credenciais privadas. Não compartilhe esses dados nem os publique no GitHub.

## Configuração do ambiente

### 1. Clonar o repositório

```bash
git clone URL_DO_SEU_REPOSITORIO
cd NOME_DO_REPOSITORIO
```

### 2. Criar e ativar o ambiente virtual

```bash
python -m venv .venv
```

No Windows, utilizando PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

No Linux ou macOS:

```bash
source .venv/bin/activate
```

### 3. Instalar as dependências

```bash
pip install -r requirements.txt
```

### 4. Configurar as credenciais

Copie o arquivo `.env.exemplo` e crie um arquivo chamado `.env` na pasta do agente.

No Windows PowerShell:

```powershell
Copy-Item .env.exemplo .env
```

Preencha o arquivo `.env` com suas credenciais:

```ini
GOOGLE_GENAI_USE_VERTEXAI=0
GOOGLE_API_KEY=sua_chave_do_google_ai_studio

TRELLO_API_KEY=sua_api_key_do_trello
TRELLO_API_SECRET=seu_api_secret_do_trello
TRELLO_TOKEN=seu_token_do_trello

TRELLO_BOARD_NAME=DIO
```

A chave do Gemini pode ser gerada gratuitamente no [Google AI Studio](https://aistudio.google.com/apikey).

O código utiliza o arquivo `.env` para carregar as credenciais e localizar o quadro configurado no Trello.

**Segurança:** o arquivo `.env` deve estar incluído no `.gitignore`. Somente o `.env.exemplo`, sem credenciais reais, deve ser enviado ao GitHub.

## Melhoria implementada: correção do fuso horário das datas

Uma das melhorias escolhidas foi corrigir o problema de fuso horário na criação de cartões com data de entrega no Trello.

### Problema identificado

Ao enviar uma data sem um horário adequado para a API do Trello, a conversão entre UTC e o horário de Brasília (UTC-3) podia fazer com que o cartão apresentasse uma data um dia anterior à informada.

Por exemplo, uma tarefa cadastrada para determinada data poderia aparecer no Trello com vencimento no dia anterior.

### Solução implementada

Foi criada a função auxiliar `_parse_due_date`, responsável por:

- Interpretar datas nos formatos `DD/MM/AAAA`, `AAAA-MM-DD`, `DD-MM-AAAA` e `DD/MM/AA`;
- Converter a data para o formato ISO 8601;
- Definir o horário como **15:00 UTC**, equivalente a 12:00 no horário de Brasília (UTC-3).

Dessa forma, a conversão de fuso horário não provoca a mudança para o dia anterior.

### Como validar a melhoria

O teste funcional consiste em:

1. Solicitar ao agente a criação de uma tarefa com uma data de entrega específica.
2. Conferir a mensagem de confirmação apresentada pelo agente.
3. Abrir o cartão criado no Trello.
4. Comparar a data de vencimento exibida no cartão com a data informada na conversa.

**Resultado esperado:** a data exibida no Trello deve corresponder ao dia informado pelo usuário, sem retroceder um dia.

## Melhorias adicionais

Além da correção do fuso horário, foram implementados outros recursos:

- **Normalização de status:** reconhece variações com e sem acentos, como `concluído`, `concluido`, `done`, `terminada`, `todo` e `in progress`.
- **Exclusão de tarefas:** implementação da ferramenta `delete_task`, que permite remover cartões por nome, com busca sem diferenciar maiúsculas e minúsculas.
- **Conexão centralizada:** utilização das funções auxiliares `_get_trello_client()` e `_get_board()` para centralizar a autenticação e a localização do quadro.
- **Tratamento de exceções:** utilização de `try/except` nas ferramentas para tratar falhas de conexão, listas inexistentes e cartões não encontrados.

## Execução do agente

Com o ambiente virtual ativado e as credenciais configuradas, execute:

```bash
adk web
```

O ADK disponibilizará uma interface web para conversar com o agente. Abra o endereço local apresentado no terminal, normalmente:

```text
http://localhost:8000
```

Selecione o agente `task_manager` e comece a interação.

## Evidências do funcionamento

As imagens abaixo demonstram a organização das tarefas no quadro Kanban do Trello, separadas por status.

### 1. Tarefas a fazer

![Tarefas a fazer no Trello](evidencias/tarefas-a-fazer.png)

### 2. Tarefas em andamento

![Tarefas em andamento no Trello](evidencias/tarefas-em-andamento.png)

### 3. Tarefas concluídas

![Tarefas concluídas no Trello](evidencias/tarefas-concluidas.png)

## Estrutura do projeto

```text
NOME_DO_REPOSITORIO/
│
├── task_manager/
│   ├── __init__.py
│   ├── agent.py
│   └── .env.exemplo
│
├── evidencias/
│   ├── tarefas-a-fazer.png
│   ├── tarefas-em-andamento.png
│   └── tarefas-concluidas.png
│
├── requirements.txt
├── .gitignore
└── README.md
```

*Observação: ajuste a estrutura acima caso os nomes ou a organização das pastas no seu repositório sejam diferentes.*

## Objetivo do projeto

Este projeto demonstra a aplicação prática de agentes de IA com Python e Google ADK, transformando funções Python em ferramentas capazes de executar ações em um serviço externo.

A integração com o Trello permite demonstrar o uso de linguagem natural, *Function Calling*, gerenciamento de credenciais, integração com APIs e automação de tarefas em um fluxo Kanban.

## Licença

Desenvolvido para o desafio de projeto da [Digital Innovation One (DIO)](https://www.dio.me/).