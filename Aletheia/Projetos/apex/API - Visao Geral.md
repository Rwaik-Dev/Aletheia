# API Apex — Visão geral

Referência para integração de **novos clients** (scripts, daemons, frontends, ferramentas internas).

Repositório: `C:\Estudos\apex`

## Documentos desta série

- [[Projetos/apex/API - Autenticacao|Autenticação e autorização]]
- [[Projetos/apex/API - Catalogo de Endpoints|Catálogo de endpoints]]
- [[Projetos/apex/API - Modelos e Enums|Modelos, enums e fingerprint]]
- [[Projetos/apex/API - Erros HTTP e limites|Erros HTTP, rate limit e headers]]
- [[Projetos/apex/API - Guia Novos Clients|Guia para novos clients]]

Ver também: [[Projetos/apex/Regras de Negocio|Regras de negócio]], [[Projetos/apex/Runbook|Runbook]], `docs/frontend-integration.md` no repo (frontend).

## Base URL

| Ambiente | URL base |
|----------|----------|
| Local (Nest) | `http://localhost:3333` |
| Produção | `http://10.2.30.102:3333` (ou host configurado) |

> O **frontend** Next.js usa `http://localhost:3000` — não é a URL da API.

## Versão e prefixo

- Prefixo REST: **`/api/v1`** (exceto health e `GET /`).
- Formato: **JSON** (`Content-Type: application/json`).
- Charset: UTF-8.

## Autenticação (resumo)

| Tipo | Como obter token | Uso |
|------|------------------|-----|
| **Service client** | `POST /api/v1/auth/token` | Scripts, ingest, consultas signum |
| **Usuário humano** | `POST /api/v1/auth/login` | UI, admin (refresh/logout) |

Header nas rotas protegidas:

```http
Authorization: Bearer <accessToken>
```

Access token JWT: TTL padrão **900 s** (15 min), configurável via `JWT_ACCESS_TTL_SECONDS`.

## Rotas públicas (sem Bearer)

- `GET /`
- `GET /health`
- `GET /health/ready`
- `POST /api/v1/auth/token`
- `POST /api/v1/auth/login`
- `POST /api/v1/auth/refresh`
- `POST /api/v1/auth/logout`

Todas as demais rotas documentadas exigem JWT válido.

## Quem pode acessar o quê

| Escopo | Service JWT (`typ: service`) | User `USER` | User `ADMIN` |
|--------|------------------------------|-------------|--------------|
| Erros (ingest/list/detail) | Sim | Sim | Sim |
| Signum (filas/exame) | Sim | Sim | Sim |
| `/api/v1/admin/*` | **Não** (403) | **Não** (403) | Sim |

## Dois formatos de erro

1. **REST padrão** (erros, auth, admin): corpo Nest usual (`message`, status HTTP).
2. **Signum** (filas/exame): `{ "error": true, "message": "..." }` — ver [[Projetos/apex/API - Erros HTTP e limites|Erros HTTP]].

## OpenAPI / Swagger

Não há Swagger publicado no projeto; esta série Obsidian espelha o código-fonte (`src/**/*.controller.ts`, DTOs).
