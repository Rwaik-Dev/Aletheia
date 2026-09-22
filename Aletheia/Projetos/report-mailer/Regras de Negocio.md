# Regras de Negócio — Report Mailer

## Agendamento

- Apenas jobs com `isActive = true` no PostgreSQL são registradas no `SchedulerService` na inicialização da API.
- Expressão cron validada antes de persistir; timezone por job (padrão `America/Sao_Paulo`).
- Disparos usam fila com `noOverlap` por tarefa cron.
- Alteração de cron/toggle exige **reload** (`POST /jobs/:key/reload`) ou reinício da API para refletir no scheduler (toggle/cron atualizam banco e acionam reload via controller).

## Concorrência e lock

- Uma execução `running` por job por vez.
- Execução com mais de **30 minutos** sem conclusão é encerrada como `failed` (stale) e uma nova execução pode iniciar.
- Se o scheduler tentar rodar enquanto a job já executa, registra warning (`JobAlreadyRunningError`) sem derrubar o processo.

## Configuração de e-mail

- `configJson.mail.to` e `configJson.mail.cc`: arrays de e-mails válidos (validação Zod no PATCH config).
- BCC global opcional via env `MAIL_BCC` (CSV).
- Destinatários vazios: cada executor pode aplicar fallback definido no código.
- Relatórios sem dados podem enviar **texto informativo sem anexo** (ex.: diligenciamento).

## Pipeline do relatório

1. Consulta SQL Server (queries por módulo em `reports/common`).
2. Montagem de contexto Handlebars → HTML.
3. Puppeteer/Chromium → PDF em `PDF_OUTPUT_DIR`.
4. Nodemailer → Gmail com anexo quando houver arquivo.
5. PDF arquivado como `ReportArtifact` (retenção: últimos **5** por job).

## Observabilidade

- Toda execução gera `JobExecution` + eventos (`job.started`, `job.succeeded` / `job.failed`, `pdf.retained`).
- Falhas abrem/atualizam `JobIncident`; sucesso resolve incidentes abertos.
- Metadados de e-mail (messageId, accepted/rejected) podem constar em `JobExecution.meta`.

## Auditoria e RBAC

- **viewer:** leitura; `errorStack` omitido nas execuções.
- **operator:** run, reload, logs de auditoria.
- **admin:** alteração de cron, toggle, `configJson`, gestão de usuários.
- Ações sensíveis registradas em `AuditLog` com usuário autenticado.

## Autenticação

- Login: e-mail + senha; JWT no cookie `httpOnly` no frontend.
- Rate limit por IP em `/auth/login`.
- Usuário inativo não autentica.

## Seed (dados iniciais)

- Upsert de todas as jobs conhecidas por `key`.
- Cron escalonado (8h) para várias jobs reduzir pico simultâneo.
- Admin criado somente se `SEED_ADMIN_PASSWORD` ≥ 12 caracteres.

Referência completa: `C:\Estudos\local-docs\report-mailer\DOCUMENTATION.MD`
