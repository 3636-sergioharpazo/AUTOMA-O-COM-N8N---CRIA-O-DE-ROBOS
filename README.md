# AUTOMA-O-COM-N8N---CRIA-O-DE-ROBOS

> API para criação de robôs com n8n — automação com Docker, VPS e API completa

---

## Sumário

* [Sobre](#sobre)
* [Funcionalidades](#funcionalidades)
* [Requisitos](#requisitos)
* [Arquitetura](#arquitetura)
* [Instalação rápida (Docker)](#instalacao-rapida-docker)
* [Instalação em VPS](#instalacao-em-vps)
* [Configuração da API](#configuracao-da-api)
* [Endpoints principais](#endpoints-principais)
* [Exemplos de uso](#exemplos-de-uso)
* [Segurança](#seguranca)
* [Deploy / Produção](#deploy--producao)
* [Contribuição](#contribuicao)
* [Licença](#licenca)
* [Contato](#contato)

---

## Sobre

Este repositório fornece uma **API** e infraestrutura para criar, gerenciar e executar *robôs* (workflows) usando o n8n — com foco em execução via **Docker** e deploy em **VPS**. A ideia é oferecer endpoints REST para controlar instâncias do n8n, criar workflows via API, ativar/desativar, armazenar credenciais seguras e executar automações em ambientes escaláveis.

## Funcionalidades

* Gerenciamento de workflows do n8n via API (criar/atualizar/excluir/listar)
* Execução remota de workflows (trigger manual ou por webhook)
* Gerenciamento de credenciais seguras (armazenadas criptografadas)
* Integração com Docker para execução isolada de n8n
* Instruções de deploy em VPS (systemd + Docker Compose)
* Logs e monitoramento básico
* Autenticação por token (JWT / API key)

## Requisitos

* Docker >= 20.x
* Docker Compose >= 1.29.x (ou compose v2)
* Node.js (apenas para desenvolvimento da API, se aplicável)
* Um VPS com acesso root ou usuário com sudo (para deploy em produção)
* Certificado SSL (Let's Encrypt recomendado) para produção

## Arquitetura

Fluxo simplificado:

1. Cliente -> Faz requisição à **API** do projeto.
2. API valida / autentica -> realiza operações (criar workflow, armazenar credenciais, acionar webhook).
3. API converte/empacota e envia dados para a instância n8n (via API interna do n8n ou via execução Docker Compose).
4. n8n executa o workflow e retorna status/logs para a API.

## Instalação rápida (Docker)

> Exemplo para desenvolvimento local usando Docker Compose.

```bash
# clone o repo
git clone https://github.com/seu-usuario/seu-repo.git
cd seu-repo

# copie o exemplo de .env
cp .env.example .env
# edite .env conforme necessário (JWT_SECRET, N8N_BASIC_AUTH_ACTIVE, etc.)

# sobe os containers
docker compose up -d --build

# verifica logs
docker compose logs -f
```

O `docker-compose.yml` deve conter serviços: `api`, `n8n`, `db` (opcional: postgres/mongodb), e `reverse-proxy` (opcional: traefik/nginx).

## Instalação em VPS

Passos básicos (exemplo com Ubuntu):

1. Atualize o servidor:

```bash
sudo apt update && sudo apt upgrade -y
```

2. Instale Docker e Docker Compose (ou Compose v2 via apt/release oficial).

3. Crie o diretório do projeto, clone e copie `.env`.

4. Configure um proxy reverso (nginx ou traefik) para expor o n8n com HTTPS.

5. Configure systemd para garantir que o `docker compose up -d` seja executado no boot (ou use crontab @reboot).

6. Obtenha certificados com Let's Encrypt (certbot) e aponte o proxy reverso.

## Configuração da API

1. Variáveis de ambiente principais (exemplo `.env`):

```
PORT=3000
JWT_SECRET=uma_chave_secreta_aqui
API_KEY=uma_chave_api
N8N_BASE_URL=http://n8n:5678
DB_URL=postgres://user:pass@db:5432/n8n_api
ENCRYPTION_KEY=chave_para_criptografia
```

2. Inicialize o banco de dados (migrations/seeds) se a API usar um banco relacional.

3. Gere chaves/segredos com `openssl rand -hex 32` para `JWT_SECRET` e `ENCRYPTION_KEY`.

## Endpoints principais (exemplos)

> Nota: adapte caminhos e parâmetros conforme implementação.

* `POST /auth/login` — Autenticação (recebe usuário/senha, retorna JWT)
* `POST /workflows` — Cria um workflow no n8n (envia JSON do workflow)
* `GET /workflows` — Lista workflows
* `GET /workflows/:id` — Detalhes do workflow
* `PUT /workflows/:id` — Atualiza workflow
* `DELETE /workflows/:id` — Remove workflow
* `POST /workflows/:id/execute` — Executa (trigger) o workflow manualmente
* `POST /webhooks/:hookId` — Endpoint público para triggers do workflow
* `POST /credentials` — Salva credenciais criptografadas
* `GET /logs/:workflowId` — Retorna logs/execuções

### Exemplo de payload (criar workflow)

```json
POST /workflows
Content-Type: application/json
Authorization: Bearer <token>

{
  "name": "meu-workflow",
  "nodes": [...],
  "connections": {...},
  "active": false
}
```

## Exemplos de uso (cURL)

Criar workflow:

```bash
curl -X POST https://seu-dominio.com/api/workflows \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d @workflow.json
```

Executar workflow manualmente:

```bash
curl -X POST https://seu-dominio.com/api/workflows/123/execute \
  -H "Authorization: Bearer $TOKEN"
```

## Segurança

* Nunca exponha as credenciais do n8n publicamente.
* Use HTTPS em produção (Let's Encrypt).
* Armazene credenciais sensíveis criptografadas (ex.: libs de criptografia com `ENCRYPTION_KEY`).
* Utilize políticas de CORS restritivas e limite IPs quando possível.
* Rotacione chaves e tokens periodicamente.

## Deploy / Produção

* Utilize um reverse-proxy (nginx/traefik) para gerenciar certificados e redirecionar tráfego para n8n e API.
* Faça backups periódicos do banco de dados e dos workflows exportados.
* Configure monitoramento (Prometheus/Grafana ou Sentry para erros da API).
* Considere orquestração com Docker Swarm / Kubernetes para alta disponibilidade.

## Contribuição

1. Fork o repositório
2. Crie uma branch feature/bugfix
3. Faça commits claros e envie PR
4. Escreva testes para novas funcionalidades

## Licença

Este projeto está sob a licença MIT — veja o arquivo `LICENSE` para detalhes.



> Se quiser, eu adapto este README com instruções específicas do seu projeto (arquivos `docker-compose.yml`, fluxos n8n de exemplo, modelos de `.env`, ou endpoints reais). Basta me enviar mais detalhes.
