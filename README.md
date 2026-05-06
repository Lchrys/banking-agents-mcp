# banking-agents-mcp

Sistema de exemplo de **atendimento bancario multiagente** usando:
- **A2A** para comunicacao com agentes especialistas
- **MCP (Model Context Protocol)** para exposicao de ferramentas/recursos
- **Supervisor** para roteamento de intencao do usuario
- **Frontends** para chat (Streamlit e React AG-UI)

## Visao geral da arquitetura

O projeto simula um banco digital (MDBank) com fluxo de atendimento por agentes:

- `supervisor`: recebe a pergunta do usuario e decide quais agentes chamar
- `agents/abrir_conta`: especialista em abertura de conta
- `agents/cartao_credito`: especialista em cartao de credito
- `recursos`: servidor MCP com ferramentas e recursos (conta/cartao)
- `bfa`: camada de discovery e busca semantica (BM25) para skills (agentes + tools)
- `frontend`: interface Streamlit para conversa com o supervisor
- `frontend2`: interface React com protocolo AG-UI

## Estrutura do projeto

```text
.
├─ supervisor/              # API FastAPI + orquestracao LangGraph
├─ agents/
│  ├─ abrir_conta/          # Agente A2A de abertura de conta
│  └─ cartao_credito/       # Agente A2A de cartao de credito
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

## Configuracao de ambiente

Os componentes com LLM usam `load_dotenv()`. Crie um arquivo `.env` com sua chave:

- `supervisor/.env`
- `agents/abrir_conta/.env`
- `agents/cartao_credito/.env`

Conteudo:

```env
OPENAI_API_KEY=sua_chave_aqui
```

## Subir o projeto

Na raiz do repositorio:

```bash
docker compose up --build
```

Servicos e portas:

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

1. Usuario envia pergunta no frontend.
2. Supervisor classifica intencao e escolhe agente(s).
3. Supervisor chama agentes via A2A.
4. Agentes usam tools/resources MCP no servico `recursos` quando necessario.
5. Respostas sao agregadas e retornam ao usuario.

## Desenvolvimento rapido

- Reinicio ao salvar esta habilitado nos containers (`--reload` / polling).
- Para derrubar tudo:

```bash
docker compose down
```

## Observacoes

- Este projeto e voltado para estudo/prototipacao de arquitetura multiagente.
- Nao usar em producao sem hardening de seguranca, autenticacao e persistencia robusta.
