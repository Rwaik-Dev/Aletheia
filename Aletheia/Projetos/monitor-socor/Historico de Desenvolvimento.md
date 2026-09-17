# Historico de Desenvolvimento - Monitor SOCOR

## 2026-09-17

- Adicionada camada operacional alinhada ao **monitor-integracao** / **monitor-hp**.
- Configuração segura: SQLite + DPAPI (`MonitorSocor` / `MonitorSocorDev`), secret `APEX_API_KEY`.
- UI Tkinter (`app.py`): Configuração, Execução (serviço Windows com auto-start), Logs.
- Serviço **MonitorSocor** via `monitor_service.py` + pywin32.
- Núcleo em `monitor_core.py`; pacote `monitor/` mantém parser/classifier MV.
- Auth Apex: service client JWT + Bearer no ingest (alinhado ao catálogo Apex).
- PyInstaller: `MonitorSocorConfig.exe`, `MonitorSocorService.exe`.
- Migração `.env` → SQLite (inclui `INTERVALO_SEGUNDOS`).

## Verificacao

- `pytest` com `MONITOR_TEST_CRYPTO=1` e `MONITOR_SKIP_ACL=1`.
