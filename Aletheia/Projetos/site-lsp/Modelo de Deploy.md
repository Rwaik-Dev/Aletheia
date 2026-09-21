---
tags:
  - site-lsp
  - deploy
  - docker
  - arquitetura
projeto: site-lsp
ultima-revisao: '2026-09-21'
status: ativo
---

# Modelo de repositório e deploy — site-lsp

[[Visão Geral]] · [[Arquitetura]] · [[Runbook]] · [[Painel Admin]]

Paridade com `DOCUMENTATION.md` §21 e `README.md`.

## Repositório e fronteiras lógicas

| Aspecto | Como está implementado |
| --- | --- |
| Repositório | **Apenas `site-lsp`** — um Git, um `package.json`, um pipeline de build |
| Aplicação | **Monólito Next.js** — site (`/`), painel (`/admin/*`) e APIs (`/api/*`) no mesmo deploy |
| Banco | PostgreSQL único (`DATABASE_URL`); schema e migrações em `prisma/` neste repo |
| Uploads | `public/banners`, `units`, `vacinas`, `convenios` (volumes Docker em produção) |
| Auth admin | Server Actions + sessões em banco; Auth.js em `/api/auth/*` |
| Cache da home | Invalidação **in-process** (`revalidatePublicHomeContent()`, `revalidateTag`) — **sem** `POST /api/revalidate` |

Separação **lógica** (pastas, guards, CSP em `/admin`), não física (dois repos ou dois containers de app). **Não existe** projeto `site-lsp-admin` para este produto.

```mermaid
flowchart TB
  subgraph repo [Repositório site-lsp]
    Next[Next.js monólito]
  end
  subgraph runtime [Runtime]
    Public[/ + /privacidade]
    Admin[/admin/*]
    API[/api/*]
  end
  DB[(PostgreSQL)]
  Vol[Volumes uploads]

  Next --> Public
  Next --> Admin
  Next --> API
  Public --> DB
  Admin --> DB
  API --> DB
  Public --> Vol
  Admin --> Vol
```

## Runtime local e Docker

**Dev:** `npm run dev` → `http://localhost:3000` (home e `/admin/login` no mesmo host).

**Docker Compose** (`docker-compose.yml`):

| Serviço | Função |
| --- | --- |
| `postgres` | PostgreSQL 16 |
| `nextjs` | Monólito; porta **3500→3000**; migrate/seed opcional no entrypoint |

Um único container `nextjs-app` atende tráfego público e administrativo. Detalhes operacionais: [[Runbook#Docker Compose]].

## Variáveis de deploy

Ver [[Runbook#Variáveis de ambiente]] e `.env.example`.

| Variável | Papel |
| --- | --- |
| `SITE_URL` | URL canônica HTTPS; e-mails e metadata; admin no **mesmo** host (`/admin/...`) |
| `DATABASE_URL` | Prisma (no Compose, host `postgres`) |
| `AUTH_SECRET`, `MFA_ENCRYPTION_KEY`, `ANALYTICS_HASH_SALT` | Segredos da aplicação única |
| `HEALTH_CHECK_TOKEN`, `TRUSTED_IP_HEADER` | Operação atrás de proxy |

**Não usadas** (ausentes do código e do `.env.example`): `ADMIN_URL`, `SITE_INTERNAL_URL`, `REVALIDATE_SECRET`.

## Endurecimento na borda (recomendação)

Mesmo em monólito, em produção convém restringir `/admin` no reverse proxy (allowlist, VPN ou auth na borda), além de MFA, rate limit e sessões já existentes. Isso **não** implica segundo repositório.

## ADR-001 (histórico)

A ideia de extrair o CMS para outro repositório está documentada em [[ADR-001 Separacao do painel CMS em site-lsp-admin]] como **decisão não adotada**. Manter runbooks e docs alinhados ao monólito descrito aqui.
