# Histórico de Desenvolvimento — Monitor SOCOR

## 2026-09-17

- **Renovação de UI:** pacote `ui/`, tema Sun Valley (`sv-ttk`), ícone em `assets/`, shell com header/footer, abas repaginadas (config scrollável, card de serviço, logs monospace + auto-refresh).
- Documentação Obsidian revisada: [[Projetos/monitor-socor/Visao Geral|Visão geral]], [[Projetos/monitor-socor/Runbook|Runbook]], [[Projetos/monitor-socor/Regras de Negocio|Regras de negócio]]; setup atualizado.
- Correção doc: autenticação Apex via **service client** (`CLIENT_ID` / `CLIENT_SECRET`), não `APEX_API_KEY`.

## 2026-09-17 (implementação)

- Camada operacional alinhada ao **monitor-integracao** / **monitor-hp**.
- Configuração segura: SQLite + DPAPI (`MonitorSocor` / `MonitorSocorDev`); secrets `client_id` e `client_secret`.
- UI Tkinter (`app.py`): Configuração, Execução (serviço Windows com auto-start), Logs.
- Serviço **MonitorSocor** via `monitor_service.py` + pywin32.
- Núcleo em `monitor_core.py`; pacote `monitor/` mantém parser/classifier MV.
- Auth Apex: JWT (`POST /api/v1/auth/token`) + Bearer no ingest; `resolveAbsent: true`.
- PyInstaller: `MonitorSocorConfig.exe`, `MonitorSocorService.exe`.
- Migração `.env` → SQLite (`env_migration.py`; inclui `INTERVALO_SEGUNDOS`).

## Verificação

```powershell
$env:MONITOR_TEST_CRYPTO = "1"
$env:MONITOR_SKIP_ACL = "1"
python -m pytest -q
```

26 testes passando (checkpoint 2026-09-17, incl. `tests/test_ui_theme.py`).
