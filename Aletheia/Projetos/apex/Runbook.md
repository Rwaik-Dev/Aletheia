# Runbook - Apex

## Desenvolvimento local

Gerenciador de pacotes: **pnpm** (`pnpm-lock.yaml`, `packageManager` no `package.json`). Não usar `npm install`.

```powershell
cd C:\Estudos\apex
cp .env.example .env
# Ajuste JWT_SECRET (32+ chars), MSSQL_* e NTFY_URL
corepack enable
docker compose up -d
pnpm install
pnpm prisma:migrate
pnpm prisma:seed
pnpm start:dev
```

- API (Nest): `http://localhost:3333` (`PORT` no `.env`)
- Frontend (Next): `http://localhost:3000` — incluir em `CORS_ORIGINS`
- Postgres no host: porta **5433** (`DATABASE_URL` em `.env.example`)
- Health: `GET /health`, readiness: `GET /health/ready`

### Token para scripts

```powershell
curl -X POST http://localhost:3333/api/v1/auth/token `
  -H "Content-Type: application/json" `
  -d '{"clientId":"apex-ingest-client","clientSecret":"<secret-do-seed>"}'
```

Credenciais admin: variáveis `BOOTSTRAP_ADMIN_*` no seed.

## Testes

```powershell
pnpm test
pnpm test:e2e
```

## Git e branches

Branch única de trabalho: **`main`**. Feature branches antigas foram apagadas após merge dos PRs #5 e #6.

### Atualizar clone local (dev ou outra máquina)

```powershell
cd C:\Estudos\apex
git fetch --prune
git checkout main
git pull origin main
# Remover branches locais órfãs, se ainda existirem:
git branch -d feat/jwt-auth-and-admin-api 2>$null
git branch -d master 2>$null
git remote prune origin
```

Conferir: `git branch -a` deve listar apenas `main` e `remotes/origin/main`.

### Servidor de produção

Após merge em `main`, no host de deploy:

```bash
cd /caminho/do/apex   # ajustar
git fetch --prune
git checkout main
git pull origin main
pnpm docker:prod:build
pnpm docker:prod:up
pnpm deploy:health
```

Não usar mais branches `feat/*` ou `master` neste repositório.

## Produção (10.2.30.102)

```bash
cp .env.production.example .env.production
# DATABASE_URL com IP 10.2.30.102:5432 (não localhost dentro do container)
pnpm docker:prod:build
pnpm docker:prod:up
pnpm deploy:health
```

- Porta exposta: **3333**
- Container roda `prisma migrate deploy` antes da API

## Variáveis críticas

| Variável | Uso |
|----------|-----|
| `DATABASE_URL` | PostgreSQL |
| `JWT_SECRET` | Assinatura JWT |
| `GOOGLE_CHAT_WEBHOOK_*` | Notificações de erro (opcional na validação, mas necessário para enviar) |
| `MSSQL_*` | Filas e exames (obrigatório na subida) |
| `NTFY_URL` | Alertas de fila (obrigatório na subida) |
| `CORS_ORIGINS` | Frontend browser |

## Troubleshooting

- App não sobe: conferir validação de env (`MSSQL_*`, `NTFY_URL`, `JWT_SECRET` ≥ 32 chars)
- 401 em signum: token ausente ou expirado (access token ~15 min)
- 403 em admin: usuário sem role `ADMIN` ou token de service client
- 429: rate limit 100 req/min
- Filas sempre vazias/erro: conectividade MSSQL a partir do host/container
- Sem Google Chat: webhooks vazios ou `errorCode` Tasy fora de `mapeamento` / `tabela de preço`
- HP sem detalhe de exame RP: conferir `MSSQL_*` e se `metadata.exm_pardini` veio no ingest; mensagem ainda sai com aviso se lookup falhar

## Google Chat

Incoming webhooks nos espaços desejados → `GOOGLE_CHAT_WEBHOOK_TASY`, `GOOGLE_CHAT_WEBHOOK_TASY_TABELA_PRECO`, `GOOGLE_CHAT_WEBHOOK_MV`, `GOOGLE_CHAT_WEBHOOK_HP`.

Erro HP `HP_CONN_001`: mensagem de importação com exames resolvidos no RP — ver [[Projetos/apex/Regras de Negocio#HP_CONN_001 — divergência de layout (Hermes Pardini)|regras HP_CONN_001]].

Docs: https://developers.google.com/workspace/chat/quickstart/webhooks
