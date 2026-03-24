# 🤖 WhatsApp AI Bot

> Chatbot inteligente integrado ao WhatsApp com RAG (Retrieval-Augmented Generation), memória de conversa persistida no Redis e stack completa em Docker.

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![LangChain](https://img.shields.io/badge/LangChain-1C3C3C?style=for-the-badge&logo=langchain&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI-412991?style=for-the-badge&logo=openai&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![ChromaDB](https://img.shields.io/badge/ChromaDB-FF6B35?style=for-the-badge)

---

## 📋 Sobre o Projeto

O **WhatsApp AI Bot** é um chatbot de produção que responde mensagens no WhatsApp com inteligência artificial, usando seus próprios documentos como base de conhecimento.

O sistema integra quatro tecnologias de forma orquestrada via Docker Compose:

- **Evolution API** — gateway que conecta o WhatsApp ao sistema via webhook
- **FastAPI** — servidor que recebe os webhooks e processa as mensagens
- **LangChain + OpenAI** — orquestração do RAG e geração de respostas
- **Redis** — armazenamento do histórico de conversa de cada usuário (por número de telefone)
- **ChromaDB** — banco vetorial para busca semântica nos documentos

O resultado é um bot que **lembra do contexto de cada conversa individualmente** e responde com base exclusivamente no conteúdo dos documentos fornecidos — pronto para uso em cenários reais como suporte ao cliente, FAQ automatizado ou assistente de documentação.

---

## ✨ Funcionalidades

- 💬 **Integração nativa com WhatsApp** via Evolution API e webhook
- 🧠 **RAG com memória persistente por usuário** — cada número de WhatsApp tem seu próprio histórico armazenado no Redis
- 📄 **Suporte a múltiplos formatos** — indexa arquivos `.pdf` e `.txt` automaticamente
- 📦 **Processamento automático** — arquivos novos são detectados, indexados e movidos para `processed/` sem intervenção manual
- 🚫 **Filtro de grupos** — ignora mensagens de grupos, respondendo apenas a conversas individuais
- 🐳 **Stack 100% dockerizada** — sobe o ambiente inteiro com um único comando
- 💾 **Persistência de dados** — volumes Docker garantem que o vectorstore e os arquivos sobrevivam a reinicializações

---

## 🛠️ Tecnologias Utilizadas

| Tecnologia | Função |
|---|---|
| **FastAPI** | Servidor webhook que recebe e processa as mensagens do WhatsApp |
| **LangChain** | Orquestração das chains de RAG com histórico de conversa |
| **OpenAI API** | LLM para geração de respostas e embeddings |
| **ChromaDB** | Banco de dados vetorial para busca semântica nos documentos |
| **Redis** | Armazenamento persistente do histórico de conversa por sessão |
| **Evolution API** | Gateway WhatsApp — conecta o número ao sistema via webhook |
| **PostgreSQL** | Banco de dados interno da Evolution API |
| **Docker + Compose** | Containerização e orquestração de todos os serviços |

---

## 🏗️ Arquitetura do Sistema

```
WhatsApp (usuário)
       │
       ▼
 Evolution API (porta 8080)
 (gateway WhatsApp → webhook)
       │
       │  POST /webhook  {chat_id, message}
       ▼
   FastAPI (porta 8000)
   └── app.py
         │
         ├── Ignora mensagens de grupos (@g.us)
         │
         ▼
   LangChain RAG Chain
   ├── history_aware_retriever
   │     ├── Redis → busca histórico do chat_id
   │     └── Reformula a query com contexto do histórico
   │
   ├── ChromaDB → busca vetorial nos documentos (top-k chunks)
   │
   ├── OpenAI GPT → gera resposta com base nos chunks
   │
   └── Redis → salva a nova mensagem no histórico
         │
         ▼
   Evolution API → envia resposta ao WhatsApp
```

### Responsabilidade de cada arquivo

| Arquivo | Responsabilidade |
|---|---|
| `app.py` | Servidor FastAPI com o endpoint `/webhook` |
| `chains.py` | Monta as LangChain chains (RAG + histórico conversacional) |
| `vectorstore.py` | Carrega documentos, gera chunks e cria/carrega o ChromaDB |
| `memory.py` | Conecta ao Redis para recuperar/salvar histórico por `session_id` |
| `evolution_api.py` | Envia mensagens de volta ao WhatsApp via Evolution API |
| `prompts.py` | Prompts de contextualização e resposta |
| `config.py` | Configurações centralizadas (chaves, URLs, caminhos) |
| `docker-compose.yml` | Orquestração de todos os 4 serviços |

---

## 🐳 Stack Docker

O `docker-compose.yml` sobe 4 serviços com dependências corretamente configuradas:

```
┌─────────────────────────────────────────────┐
│              Docker Compose                 │
│                                             │
│  ┌──────────────┐    ┌──────────────────┐   │
│  │ evolution-api│    │       bot        │   │
│  │  (porta 8080)│    │   (porta 8000)   │   │
│  └──────┬───────┘    └────────┬─────────┘   │
│         │depends_on           │depends_on   │
│         ▼                     ▼             │
│  ┌──────────────┐    ┌──────────────────┐   │
│  │   postgres   │    │      redis       │   │
│  │  (porta 5432)│    │   (porta 6379)   │   │
│  └──────────────┘    └──────────────────┘   │
│                                             │
│  Volumes persistentes:                      │
│  • evolution_instances  • postgres_data     │
│  • redis                • vectorstore_data  │
│  • rag_files                                │
└─────────────────────────────────────────────┘
```

---

## 🚀 Como Rodar

### Pré-requisitos

- [Docker](https://docs.docker.com/get-docker/) e [Docker Compose](https://docs.docker.com/compose/) instalados
- Chave de API da [OpenAI](https://platform.openai.com/)
- Conta na [Evolution API](https://doc.evolution-api.com/) (self-hosted via Docker)

### 1. Clone o repositório

```bash
git clone https://github.com/guilherme-donato-dev/whatsapp_ai_bot.git
cd whatsapp_ai_bot
```

### 2. Configure as variáveis de ambiente

Crie o arquivo `.env` na raiz do projeto:

```env
# OpenAI
OPENAI_API_KEY=sk-...

# Evolution API
EVOLUTION_API_URL=http://evolution-api:8080
EVOLUTION_API_KEY=sua_chave_aqui
EVOLUTION_INSTANCE_NAME=nome_da_instancia

# Redis
REDIS_URL=redis://redis:6379/0

# Caminhos dos arquivos
RAG_FILES_DIR=/app/rag_files
VECTOR_STORE_PATH=/app/vectorstore
```

### 3. Adicione seus documentos

Coloque arquivos `.pdf` ou `.txt` na pasta `rag_files/`:

```
rag_files/
├── manual_produto.pdf
├── faq.txt
└── politica_de_troca.pdf
```

> O bot indexará esses arquivos automaticamente na inicialização e os moverá para `rag_files/processed/` após processar.

### 4. Suba todos os serviços

```bash
docker compose up -d
```

Aguarde todos os containers inicializarem. Você pode acompanhar os logs com:

```bash
docker compose logs -f bot
```

### 5. Configure o Webhook na Evolution API

Acesse o painel da Evolution API em `http://localhost:8080` e configure o webhook apontando para:

```
http://bot:8000/webhook
```

### 6. Conecte o WhatsApp

Via painel da Evolution API, escaneie o QR Code com o número que será o bot.

A partir desse ponto, qualquer mensagem enviada para esse número receberá uma resposta automática do bot.

---

## 📁 Estrutura do Projeto

```
whatsapp_ai_bot/
├── app.py              # Servidor FastAPI + endpoint /webhook
├── chains.py           # Chains LangChain (RAG + conversacional)
├── vectorstore.py      # Carregamento de documentos e ChromaDB
├── memory.py           # Histórico de conversa via Redis
├── evolution_api.py    # Envio de mensagens ao WhatsApp
├── prompts.py          # Prompts do sistema
├── config.py           # Configurações centralizadas
├── Dockerfile          # Build da imagem do bot
├── docker-compose.yml  # Orquestração dos 4 serviços
├── requirements.txt    # Dependências Python
├── .dockerignore
├── .gitignore
├── .env                # Variáveis de ambiente (NÃO versionar)
└── rag_files/
    └── processed/      # Arquivos já indexados são movidos para cá
```

---

## 🔧 Decisões Técnicas

**Por que Redis para o histórico de conversa?**
O Redis é um banco em memória extremamente rápido, ideal para armazenar histórico de sessão. O `RedisChatMessageHistory` do LangChain usa o `chat_id` (número de telefone do WhatsApp) como `session_id`, garantindo que cada usuário tenha seu próprio histórico isolado — algo essencial em um chatbot multi-usuário.

**Por que o bot filtra mensagens de grupos?**
A Evolution API entrega webhooks tanto de mensagens individuais quanto de grupos. O `chat_id` de grupos sempre termina em `@g.us`. O filtro `if not '@g.us' in chat_id` garante que o bot só responda a conversas individuais, evitando spam e comportamento indesejado em grupos.

**Por que os arquivos RAG são movidos para `processed/`?**
Para evitar reindexação desnecessária a cada restart do container. Uma vez que um arquivo é processado e seus chunks estão no ChromaDB, ele é movido para `processed/`. Arquivos novos colocados em `rag_files/` são automaticamente detectados na próxima inicialização.

**Por que volumes Docker para o vectorstore?**
O ChromaDB persiste seus dados em disco. Sem o volume `vectorstore_data`, os dados seriam perdidos ao derrubar o container. O volume garante que os embeddings gerados sobrevivam a atualizações e reinicializações do bot.

---

## ⚠️ Observações Importantes

- Este projeto requer um **servidor com IP público** para que o webhook da Evolution API seja acessível. Para testes locais, utilize um serviço como [ngrok](https://ngrok.com/) ou [localtunnel](https://github.com/localtunnel/localtunnel).
- O uso da **OpenAI API** gera custos baseados em tokens. Para uso intenso, monitore seu consumo no painel da OpenAI.
- **Nunca versione o arquivo `.env`** — ele contém suas chaves de API.

---

## 👨‍💻 Autor

**Guilherme Donato**

[![GitHub](https://img.shields.io/badge/GitHub-guilherme--donato--dev-181717?style=flat-square&logo=github)](https://github.com/guilherme-donato-dev)
