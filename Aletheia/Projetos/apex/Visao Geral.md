# Apex

Backend NestJS para receber, persistir e notificar erros das integrações **Tasy** e **MV**, além de consultar filas e exames no SQL Server (portado do signum-backend).

Repositório: `C:\Estudos\apex`

Documentação no repo: `README.md` e `docs/frontend-integration.md` (frontend).

## API (Obsidian)

Referência completa para integradores: [[Projetos/apex/API - Visao Geral|API — Visão geral]].

## Responsabilidades

| Área | Função |
|------|--------|
| Erros Tasy/MV | Ingestão REST, deduplicação por fingerprint, resolução por snapshot, Google Chat |
| Signum | Filas Tasy/Checkup/Interface e consulta de exame (`smk_cod`) via MSSQL |
| Alertas | Cron NTFY quando fila > 40 itens (a cada 5 min) |
| Auth | JWT para scripts (service clients) e usuários humanos (ADMIN/USER) |
| Admin | CRUD de usuários, service clients e revogação de sessões (ADMIN) |

## Stack

- NestJS 11, Prisma 7, PostgreSQL
- MSSQL (`mssql`), `@nestjs/schedule`
- JWT + bcrypt, Helmet, Throttler (100 req/min), CORS

## Estrutura (src)

- `auth/` — token, login, refresh, logout
- `admin/` — usuários, service clients, sessões
- `errors/` — ingest e listagem
- `notifications/` — Google Chat, NTFY, políticas Tasy/MV
- `signum/` — filas e consulta exame
- `scheduler/` — lembretes Tasy e monitor de filas
- `health/` — `/health` e `/health/ready`

## Integrações ingeridas

Enum Prisma: **`TASY`**, **`MV`** e **`HP`**. Clientes externos usam um desses valores em `POST /api/v1/errors/ingest`.

## Relacionados

- [[Projetos/monitor-hp/Visao Geral|Monitor HP]] — cliente que envia erros para o Apex (ver alinhamento de `integration` no payload)
