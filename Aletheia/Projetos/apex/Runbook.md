# Runbook - Apex

## Desenvolvimento local

```powershell
cd C:\Estudos\apex
cp .env.example .env
# Ajuste JWT_SECRET (32+ chars), MSSQL_* e NTFY_URL
docker compose up -d
npm install
npx prisma migrate dev
npm run prisma:seed
npm run start:dev
```

- API: `http://localhost:3000`
- Postgres no host: porta **5433** (`DATABASE_URL` em `.env.example`)
- Health: `GET /health`, readiness: `GET /health/ready`

### Token para scripts

```powershell
curl -X POST http://localhost:3000/api/v1/auth/token `
  -H "Content-Type: application/json" `
  -d '{"clientId":"apex-ingest-client","clientSecret":"<secret-do-seed>"}'
```

Credenciais admin: variáveis `BOOTSTRAP_ADMIN_*` no seed.

## Testes

```powershell
npm test
npm run test:e2e
```

## Produção (10.2.30.102)

```bash
cp .env.production.example .env.production
# DATABASE_URL com IP 10.2.30.102:5432 (não localhost dentro do container)
npm run docker:prod:build
npm run docker:prod:up
npm run deploy:health
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

## Google Chat

Incoming webhooks nos espaços desejados → `GOOGLE_CHAT_WEBHOOK_TASY`, `GOOGLE_CHAT_WEBHOOK_TASY_TABELA_PRECO`, `GOOGLE_CHAT_WEBHOOK_MV`.

Docs: https://developers.google.com/workspace/chat/quickstart/webhooks
