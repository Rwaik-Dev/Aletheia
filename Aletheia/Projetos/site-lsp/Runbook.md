---
tags:
  - site-lsp
  - runbook
  - deploy
  - docker
projeto: site-lsp
ultima-revisao: '2026-09-18'
---

# Runbook — site-lsp

[[Visão Geral]] · [[Testes]] · [[Autenticação e Segurança]]

## Pré-requisitos

- Node.js LTS + npm
- PostgreSQL 16 (local ou via Compose)
- Docker opcional (recomendado para paridade prod)

## Setup local (sem Docker)

```bash
cp .env.example .env   # PowerShell: Copy-Item .env.example .env
npm install
npm run prisma:generate
npm run prisma:migrate
# Definir ADMIN_EMAIL + ADMIN_PASSWORD fortes no .env
npm run db:seed
npm run dev
```

- Site: http://localhost:3000
- Admin: http://localhost:3000/admin/login

`DATABASE_URL` exemplo local:
`postgresql://postgres:postgres@localhost:5432/site_lsp?schema=public`

## Variáveis de ambiente

Referência: `.env.example` + `app/lib/env.ts`.

| Variável | Obrig. | Notas |
| --- | --- | --- |
| `DATABASE_URL` | Sempre | Prisma |
| `POSTGRES_PASSWORD` | Compose | Senha DB no docker-compose |
| `SITE_URL` | Prod | HTTPS, sem barra final |
| `AUTH_SECRET` | Prod | Auth.js |
| `ANALYTICS_HASH_SALT` | Prod | Hashes analytics |
| `MFA_ENCRYPTION_KEY` | Prod | AES secrets TOTP |
| `MAIL_*` + `GOOGLE_APP_PASSWORD` | E-mail real | Contato + transacionais |
| `DISABLE_EMAIL_DELIVERY` | Opcional | `true` dev/E2E |
| `HEALTH_CHECK_TOKEN` | Recomendado prod | Bearer health |
| `TRUSTED_IP_HEADER` | **Prod** | Rate limit / IP real |
| `AUTH_TRUST_HOST` | Proxy/Docker | `true` comum |
| `ADMIN_NAME/EMAIL/PASSWORD` | Seed | Senha ≥12, idempotente |
| `RUN_MIGRATE_ON_START` | Docker | default true no entrypoint |
| `RUN_SEED_ON_START` | Docker | compose usa `"false"` após 1º deploy |

Em **dev**, `AUTH_SECRET`, `ANALYTICS_HASH_SALT` e MFA key têm fallbacks inseguros — nunca usar em produção.

## Docker Compose

Arquivos: `docker-compose.yml`, `Dockerfile`, `docker/entrypoint.sh`.

```bash
docker compose up --build
```

| Serviço | Detalhe |
| --- | --- |
| `postgres` | `site_lsp` / user `site_lsp_app`; porta `127.0.0.1:5432` exposta para dev local |
| `nextjs` | App na porta **3500**; `DATABASE_URL` aponta host `postgres` |

Volumes de upload: `banners`, `units`, `vacinas`, `convenios`.

**Seed no container:** `RUN_SEED_ON_START` default `"false"` no compose — ligar só no **primeiro** deploy com `ADMIN_*` no `.env`, depois manter false.

Proxy corporativo (build npm):

```powershell
$env:NPM_CONFIG_STRICT_SSL="false"; docker compose up --build
```

Não usar em CI/prod; preferir `NODE_EXTRA_CA_CERTS` com CA corporativa.

## Banco de dados

```bash
npm run prisma:studio    # UI
npm run db:seed          # re-seed idempotente no host
```

No host com Postgres do compose, `DATABASE_URL` usa `localhost:5432`; dentro do container, hostname `postgres`.

## Deploy (checklist)

1. Secrets reais: DB, `SITE_URL` HTTPS, mail, salts, auth.
2. `TRUSTED_IP_HEADER` + `HEALTH_CHECK_TOKEN`.
3. `docker compose up --build` (ou build Node + `migrate deploy`).
4. Seed inicial uma vez se necessário.
5. Smoke tests (abaixo).

## Smoke tests pós-deploy

- [ ] `GET /api/health` (com Bearer se token configurado)
- [ ] Login admin + MFA
- [ ] Home carrega conteúdo do DB
- [ ] Formulário contato (ou `DISABLE_EMAIL_DELIVERY` em staging)
- [ ] Upload banner/unidade → restart container → arquivo persiste nos volumes

## Troubleshooting

| Sintoma | Ação |
| --- | --- |
| `DATABASE_URL is not configured` | Preencher `.env` |
| Seed recusa senha admin | ≥12 chars, letras+números; evitar placeholders |
| E-mail não envia | Credenciais Gmail; `DISABLE_EMAIL_DELIVERY` |
| Login admin falha | Seed rodou? Lockout? MFA pendente? |
| Health 401 | Configurar token + header Bearer |
| IP `unknown` em analytics | `TRUSTED_IP_HEADER` no proxy |
| E2E falha | DB up, migrações, Playwright browsers |

## Documentação PDF

```bash
npm run docs:pdf
```

Gera HTML/PDF em `.codex/docs/` (gitignored) a partir de `DOCUMENTATION.md`.

## Backup

- **PostgreSQL:** dump regular do volume `postgres_data`.
- **Uploads:** volumes Docker ou cópia de `public/banners|units|vacinas|convenios`.
