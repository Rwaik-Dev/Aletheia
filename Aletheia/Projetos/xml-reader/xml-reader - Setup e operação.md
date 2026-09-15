---
tags:
  - xml-reader
  - setup
  - docker
  - devops
projeto: xml-reader
ultima-revisao: 2026-09-15
---

# xml-reader — Setup e operação

Relacionado: [[xml-reader]] · [[xml-reader - Documentação técnica]]

## Pré-requisitos

| Requisito | Detalhe |
| --- | --- |
| Node.js | Alinhar com Docker: **22.x** |
| pnpm | Lockfile no repo; Docker usa `pnpm@11.17.0` |
| Rede | Acesso ao SQL Server corporativo + HTTPS ao host HP |

## Variáveis de ambiente

Template no repo: **`.env.exemple`** (typo intencional no nome do arquivo). `.env` está no `.gitignore` e **não** entra na imagem Docker.

### MSSQL (Smart)

| Variável | Obrigatória | Default |
| --- | --- | --- |
| `MSSQL_HOST` | Sim | — |
| `MSSQL_USER` | Sim | — |
| `MSSQL_PASSWORD` | Sim | — |
| `MSSQL_NAME` | Sim | — |
| `MSSQL_PORT` | Não | `1433` |
| `MSSQL_ENCRYPT` | Não | `false` |

`lib/db.ts`: `trustServerCertificate: true`, timeouts 15s / 30s. `readEnv()` (`lib/env.ts`) faz trim e remove aspas (útil com `docker run --env-file`).

### Hermes Pardini XML

| Variável | Default / notas |
| --- | --- |
| `HP_XML_URL` | `https://www.hermespardini.com.br/cal/exames/modelos.xml` |
| `HP_XML_USER` / `HP_XML_PASSWORD` | **Definir sempre**; código tem fallback hardcoded se faltarem (risco — ver doc de segurança) |
| `HP_TLS_INSECURE` | `"true"` desliga verificação TLS (`rejectUnauthorized: false`) |

## Comandos

```bash
pnpm install
cp .env.exemple .env   # preencher credenciais

pnpm dev      # http://localhost:7000
pnpm build && pnpm start
pnpm lint
pnpm test
pnpm docs:pdf # gera DOCUMENTATION.pdf
```

## Docker

```bash
docker build -t xml-reader .
docker run --env-file .env -p 7000:7000 xml-reader
```

Notas:

- Segredos só em **runtime** (`--env-file`).
- Hostname SQL deve **resolver dentro do container** (usar IP/FQDN se alias interno falhar).
- Dockerfile: `strict-ssl false` no npm/pnpm para ambientes com inspeção SSL corporativa.

## PDF da documentação no repo

| Arquivo | Função |
| --- | --- |
| `DOCUMENTATION.md` | Fonte |
| `md-to-pdf.config.json` | Margens, browser (Edge no Windows do projeto) |
| `pdf-theme.css` | Estilo do PDF |

Regenerar após mudanças relevantes em `DOCUMENTATION.md`.
