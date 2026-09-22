# Runbook — Report Mailer

Documentação detalhada: `C:\Estudos\local-docs\report-mailer\DOCUMENTATION.MD`

## Pré-requisitos

- Docker Desktop (Compose) **ou** Node 22 + pnpm 11
- Credenciais **MSSQL** (relatórios) e **Gmail** (app password)
- `JWT_SECRET` **idêntico** em `backend/.env` e `frontend/.env.local`

## Subir stack com Docker (dev)

```powershell
cd C:\Estudos\report-mailer
copy .env.docker.example .env
# Editar backend\.env e frontend\.env.local
docker compose up -d --build
```

| Serviço | URL |
|---------|-----|
| Painel | http://localhost:7000 |
| API | http://localhost:3334 |
| Health | http://localhost:3334/health |

## Dev nativo (API + Next fora do container)

```powershell
cd C:\Estudos\report-mailer
docker compose up postgres -d

cd backend
pnpm install
pnpm exec prisma migrate deploy
pnpm run prisma:seed
pnpm run dev

# Outro terminal
cd ..\frontend
pnpm install
pnpm run dev
```

`DATABASE_URL` no backend: `postgresql://postgres:postgres@localhost:5431/rmdb?schema=public`

## Seed e admin inicial

- Jobs: `pnpm run prisma:seed` (backend)
- Admin: definir `SEED_ADMIN_EMAIL` e `SEED_ADMIN_PASSWORD` (mín. 12 caracteres) no `backend/.env`
- Container: `RUN_SEED_ON_START=true` no backend (usar com cautela em produção)

## Operação via painel

1. Login em `/login`
2. **Jobs** — listar, detalhe por key, executar manualmente (operator+)
3. **Audit** — trilha de alterações (operator+)
4. **Users** — CRUD de usuários (admin)

## Operação via API

```powershell
# Login
$body = @{ email = "admin@example.com"; password = "***" } | ConvertTo-Json
$r = Invoke-RestMethod -Method POST -Uri http://localhost:3334/auth/login -Body $body -ContentType application/json
$token = $r.data.token

# Executar job
Invoke-RestMethod -Method POST -Uri http://localhost:3334/jobs/producao/run `
  -Headers @{ Authorization = "Bearer $token" }
```

## Deploy produção (Compose)

1. `copy .env.production.example .env` — ajustar `DATABASE_URL`, URLs públicas, `JWT_SECRET`
2. `backend/.env` — MSSQL, Gmail, `CORS_ORIGIN` do painel
3. `docker compose up -d --build`
4. Se Postgres já existir no host, **não** subir serviço `postgres` do compose (ver comentários em `.env.production.example`)

## Testes

```powershell
cd C:\Estudos\report-mailer\backend
pnpm run test

cd ..\frontend
pnpm run test:unit
```

## Troubleshooting rápido

| Problema | Verificar |
|----------|-----------|
| Health 503 | Postgres + MSSQL |
| Loop no login | `JWT_SECRET` backend = frontend |
| Job não roda | `isActive`, cron, logs do scheduler |
| Sem PDF | Chromium no container, `PDF_OUTPUT_DIR` |
| Sem e-mail | `GOOGLE_APP_*`, logs mail |
| Banco errado no Docker | `POSTGRES_DB` vs volume antigo (`docker compose down -v`) |

Ver seção 20 em `DOCUMENTATION.MD`.
