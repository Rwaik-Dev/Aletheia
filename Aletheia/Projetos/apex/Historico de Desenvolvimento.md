# Historico de Desenvolvimento - Apex

## 2026-09-15

- Revisão de alinhamento entre código e `README.md` (JWT, admin API, signum, deploy produção).
- README atualizado: porta Postgres 5433, env obrigatório, rate limit, cron Tasy 15 min, stack e scripts npm.
- Criada pasta Obsidian `Projetos/apex/` (Visão Geral, Regras, Runbook).

## 2026-09-15 (tarde)

- Série **API Apex** no Obsidian: autenticação, catálogo de endpoints, modelos, erros HTTP, guia para novos clients.

## Verificação

- Endpoints conferidos nos controllers em `src/**/*.controller.ts`.
- Variáveis conferidas em `.env.example` e `src/config/env.validation.ts`.
- DTOs e guards conferidos para documentação API Obsidian.
