# Report Mailer

Plataforma para **agendar, gerar (PDF) e enviar relatórios operacionais** do Laboratório São Paulo por e-mail, com painel administrativo web.

**Repositório:** `C:\Estudos\report-mailer`  
**Documentação completa:** `C:\Estudos\local-docs\report-mailer\DOCUMENTATION.MD`

## Componentes

| Parte | Stack | Porta (dev) |
|-------|-------|-------------|
| Backend API | Express 5 + TypeScript, Prisma 7, Puppeteer, Nodemailer | 3334 |
| Frontend admin | Next.js 16 + React 19 + Tailwind 4 | 7000 |
| Metadados | PostgreSQL (`rmdb` dev / `report_mailer` prod) | 5431 (host) |
| Dados dos relatórios | Microsoft SQL Server | — |
| E-mail | Gmail (app password) via Nodemailer | — |

## Estrutura

- `backend/src/app/` — bootstrap, rotas HTTP, middlewares
- `backend/src/modules/jobs/` — scheduler, runner, CRUD e observabilidade
- `backend/src/modules/reports/common/` — um pacote por relatório (query → HTML → PDF → mail)
- `frontend/app/(protected)/` — dashboard, jobs, auditoria, usuários
- `docker-compose.yml` — Postgres + backend + frontend

## Jobs registradas (executors)

`diligenciamento`, `producao`, `perdas`, `producao-unidade`, `tat-medio`, `tat-medio-micro`, `taxa-recoleta`, `lib-automatica`, `tat-medio-upa`, `taxa-recoleta-upa`, `taxa-recoleta-socor`, `relatorio-geral-socor`, `controle-recoleta`

Detalhes de cron, seed e config de e-mail: ver [[Projetos/report-mailer/Regras de Negocio|Regras de Negócio]] e `DOCUMENTATION.MD`.

## Papéis (RBAC)

- **viewer** — leitura (sem stack de erro)
- **operator** — run/reload + auditoria
- **admin** — cron/toggle/config + usuários

## Links

- [[Projetos/report-mailer/Runbook|Runbook]]
- [[Projetos/report-mailer/Regras de Negocio|Regras de Negócio]]
