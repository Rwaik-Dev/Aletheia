# Historico de Desenvolvimento - Apex

## 2026-09-16 (tarde)

- Documentação e tooling alinhados a **pnpm**: `README.md`, `Dockerfile`, `docker/entrypoint.sh`, `.env.production.example`, `packageManager`/`engines` no `package.json`.
- Runbook Obsidian atualizado (`pnpm install`, scripts `pnpm *`).
- PR [#7](https://github.com/Rwaik-Dev/apex/pull/7) **mergeado** em `main`; branch `chore/pnpm-docs-and-docker` removida.

## 2026-09-16

- **HP_CONN_001:** notificação Google Chat enriquecida para erros HP com divergência de layout.
- Consulta RP via `exm_pardini` (`consultaExamesHpardiniPorMnemonico`); suporte a múltiplos exames `smk_cod` / `smk_nome` na mesma mensagem.
- Metadata HP no ingest: `pedido`, `chave_local`, `exm_pardini`, `arquivo_origem`, `cod_formato_certo`, `cod_formato_errado`.
- Fallback: notificação enviada mesmo se RP falhar ou não retornar linhas (aviso na mensagem).
- Testes unitários: `mssql.service.spec.ts`, `notification.service.spec.ts`, `google-chat.service.spec.ts`.
- Commit `661423a`; PR [#6](https://github.com/Rwaik-Dev/apex/pull/6) **mergeado** em `main` (2026-09-16).
- Limpeza de branches: removidas localmente `feat/jwt-auth-and-admin-api` e `master`; no remoto `feat/production-docker-deploy` e `feat/jwt-auth-and-admin-api`. Repositório fica só com `main`.
- Obsidian: regras HP_CONN_001 em [[Projetos/apex/Regras de Negocio|Regras de Negocio]]; sync Git em [[Projetos/apex/Runbook#Git e branches|Runbook]].

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
