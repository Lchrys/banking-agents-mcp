# Banking Agents Ecosystem: Orquestração Multiagente com MCP & A2A

### Projeto desenvolvido como estudo na ALURA
---
Sistema de exemplo de **atendimento bancário multiagente** usando:
- **A2A** para comunicação com agentes especialistas
- **MCP (Model Context Protocol)** para exposição de ferramentas/recursos
- **Supervisor** para roteamento de intenção do usuário
- **Frontends** para chat (Streamlit e React AG-UI)

## Visão geral da arquitetura

O projeto simula um banco digital com fluxo de atendimento por agentes:

- `supervisor`: recebe a pergunta do usuário e decide quais agentes chamar
- `agents/abrir_conta`: especialista em abertura de conta
- `agents/cartao_credito`: especialista em cartão de crédito
- `recursos`: servidor MCP com ferramentas e recursos (conta/cartão)
- `bfa`: camada de discovery e busca semântica (BM25) para skills (agentes + tools)
- `frontend`: interface Streamlit para conversa com o supervisor
- `frontend2`: interface React com protocolo AG-UI

## Estrutura do projeto

```text
.
├─ supervisor/              # API FastAPI + orquestração LangGraph
├─ agents/
│  ├─ abrir_conta/          # Agente A2A de abertura de conta
│  └─ cartao_credito/       # Agente A2A de cartão de crédito
├─ recursos/                # Servidor MCP (tools/resources)
├─ bfa/                     # Discovery de agentes/tools e resolver BM25
├─ frontend/                # Frontend Streamlit
├─ frontend2/               # Frontend React (AG-UI)
└─ docker-compose.yml       # Stack completa local
```

## Requisitos

- Docker
- Docker Compose (v2+)
- Chave da OpenAI (`OPENAI_API_KEY`)

## Configuração de ambiente

Os componentes com LLM usam `load_dotenv()`. Crie um arquivo `.env` com sua chave:

- `supervisor/.env`
- `agents/abrir_conta/.env`
- `agents/cartao_credito/.env`

Conteúdo:

```env
OPENAI_API_KEY=sua_chave_aqui
```

## Subir o projeto

Na raiz do repositório:

```bash
docker compose up --build
```

Serviços e portas:

- Supervisor API: `http://localhost:8080`
- Frontend Streamlit: `http://localhost:9090`
- Agente abrir_conta: `http://localhost:8081`
- Agente cartao_credito: `http://localhost:8082`
- BFA: `http://localhost:8083`
- Recursos MCP: `http://localhost:8084`
- Frontend React (AG-UI): `http://localhost:3000`

## Endpoints principais

### Supervisor (`:8080`)
- `POST /chat`  
  Recebe `{ "message": "..." }` e retorna resposta consolidada do supervisor.
- `POST /`  
  Endpoint streaming para AG-UI.

### BFA (`:8083`)
- `GET /skills`
- `GET /skills/agents`
- `GET /skills/tools`
- `GET /resolve?query=...`
- `GET /resolve/agents?query=...`
- `GET /resolve/tools?query=...`

### Recursos MCP (`:8084`)
- `GET /tools` (lista ferramentas MCP)

## Fluxo funcional (resumo)

1. Usuário envia pergunta no frontend.
2. Supervisor classifica intenção e escolhe agente(s).
3. Supervisor chama agentes via A2A.
4. Agentes usam tools/resources MCP no serviço `recursos` quando necessário.
5. Respostas são agregadas e retornam ao usuário.

## Desenvolvimento rápido

- Reinício ao salvar está habilitado nos containers (`--reload` / polling).
- Para derrubar tudo:

```bash
docker compose down
```

## Observações

- Este projeto é voltado para estudo/prototipação de arquitetura multiagente.
- Não usar em produção sem hardening de segurança, autenticação e persistência robusta.
