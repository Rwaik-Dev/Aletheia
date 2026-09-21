---
tags:
  - site-lsp
  - projeto
  - nextjs
  - lab-sao-paulo
projeto: site-lsp
repositorio: 'C:/Estudos/site-lsp'
ultima-revisao: '2026-09-21'
status: ativo
---

# site-lsp — Visão geral

Site institucional do **Lab São Paulo** (laboratório de análises clínicas em Belo Horizonte): landing pública + painel CMS em `/admin`, PostgreSQL, Docker e testes automatizados.

| Metadado | Valor |
| --- | --- |
| Pacote npm | `site-lsp` v0.1.0 |
| Repositório | `C:\Estudos\site-lsp` |
| Stack | Next.js 16, React 19, Prisma 7, PostgreSQL 16, Tailwind 4, Auth.js v5 |
| Doc técnica no repo | `DOCUMENTATION.md` — **fonte versionada no Git** (PDF: `npm run docs:pdf`) |
| Paridade | Ao mudar o código, atualize este vault **e** `DOCUMENTATION.md` (mapa §22) |

## Mapa da wiki

| Nota | Conteúdo |
| --- | --- |
| [[Arquitetura]] | Camadas, fluxos, cache, CSP, padrão admin |
| [[Site Público]] | Seções da home, analytics, contato, privacidade |
| [[Painel Admin]] | Módulos, rotas, uploads, Server Actions |
| [[Autenticação e Segurança]] | Login, MFA, sessões, rate limit, retenção |
| [[APIs e Integrações]] | Contact, analytics, health, Auth.js, e-mail |
| [[Modelo de Dados]] | Schema Prisma, migrações, seed |
| [[Runbook]] | Setup local/Docker, deploy, troubleshooting |
| [[Testes]] | Vitest + Playwright |
| [[Modelo de Deploy]] | Monólito: um repo, um container Next.js, Docker |
| [[ADR-001 Separacao do painel CMS em site-lsp-admin]] | Histórico — split **não adotado** |
| [[2026-09-16 Politica de Privacidade LGPD]] | LGPD, `/privacidade`, analytics sem banner de cookies |

## Uma aplicação, duas superfícies

**Um repositório Git** (`site-lsp`) e **um processo Next.js** servem site público e painel no mesmo host (`/` e `/admin`). Ver [[Modelo de Deploy]].


```mermaid
flowchart TB
  subgraph publico [Site público]
    Home[/home]
    Priv[/privacidade]
    APIpub[APIs: contact, analytics, health]
  end

  subgraph admin [Painel /admin]
    Auth[Login MFA onboarding]
    CRUD[CRUD conteúdo]
    Dash[Analytics dashboard]
  end

  DB[(PostgreSQL)]
  Uploads[public/banners units vacinas convenios]

  Home --> DB
  APIpub --> DB
  admin --> DB
  admin --> Uploads
  Home --> Uploads
```

| Superfície | URL base | Responsabilidade |
| --- | --- | --- |
| **Pública** | `/`, `/privacidade` | Leitura de conteúdo, formulário de contato, ingestão de analytics |
| **Admin** | `/admin/*` | CRUD institucional, usuários, analytics agregado, MFA e auditoria |

## Problemas que o sistema resolve

- Conteúdo institucional (banners, unidades, serviços, vacinas, convênios, etc.) editável sem deploy de código.
- Autenticação forte para operadores (sessão em banco, Argon2id, MFA TOTP, lockout, audit log).
- Site resiliente: cache da home com invalidação; fallback estático se o banco estiver vazio ou indisponível.
- Conformidade operacional: política de privacidade alinhada ao código; retenção de analytics/audit; rate limits persistentes.

## Comportamentos-chave (implementados)

- Cache da home: tag `public-home-content`, revalidate 300s; invalidação após mudanças no admin.
- Rate limit em PostgreSQL (`RateLimitEntry`), não em memória.
- Health check: `GET /api/health` (Bearer em produção se `HEALTH_CHECK_TOKEN` definido).
- E-mail via Gmail (app password); `DISABLE_EMAIL_DELIVERY=true` para dev/E2E.
- Logger estruturado server-side (`app/lib/logger.ts`).

## URLs locais

| Ambiente | Site | Admin | Health |
| --- | --- | --- | --- |
| `npm run dev` | http://localhost:3000 | http://localhost:3000/admin/login | http://localhost:3000/api/health |
| Docker Compose | http://localhost:3500 | http://localhost:3500/admin/login | http://localhost:3500/api/health |

## Scripts npm (referência rápida)

```bash
npm run dev              # desenvolvimento
npm run build && npm run start
npm run test             # Vitest
npm run test:e2e         # Playwright
npm run prisma:migrate   # migrate dev
npm run db:seed          # admin + conteúdo inicial (idempotente)
npm run docs:pdf         # gera PDF a partir de DOCUMENTATION.md
```

## Estrutura de pastas (alto nível)

```text
app/
  admin/           painel CMS + auth flows
  api/             contact, analytics, health, [...nextauth]
  components/      UI pública
  lib/             auth, cache, prisma, mail, analytics, env
auth.ts            NextAuth + Prisma adapter
proxy.ts           CSP por request + gate mínimo /admin
prisma/            schema, migrations, seed.ts
docker/            entrypoint (migrate + optional seed + start)
tests/unit/        Vitest
tests/e2e/         Playwright
```

## Links externos / referências internas

- README operacional: `README.md`
- Variáveis: `.env.example` + [[Runbook#Variáveis de ambiente]]
- Rotas nomeadas: `app/lib/routes.ts` (`ROUTES`, `ADMIN_REVALIDATE_PATHS`)
