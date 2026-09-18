# Monitor Integração Tasy

Cliente Python para Windows que monitora HTMLs de erro do **Tasy**, mantém estado local, gera planilhas Excel e envia snapshot de erros ativos ao **Apex**.

Repositório: `C:\Estudos\monitor-integracao`

## Interface (UI)

Alinhado ao padrão compartilhado: [[Projetos/monitores/UI Desktop - Padrão Apex Clients|UI Desktop — padrão Apex Clients]].

- Pacote `ui/` (Sun Valley / `sv-ttk`, shell header + abas Config / Execução / Logs)
- `app.py` apenas como controller
- Ícones: `assets/` + `python scripts/generate_icon.py`
- Executáveis: `MonitorConfig.exe` + `MonitorIntegracaoService.exe`

## Estrutura

- `app.py`: orquestração da UI e callbacks
- `ui/`: layout e tema
- `config.py` / `config_repository.py`: SQLite + DPAPI
- `monitor_core.py`: parser HTML Tasy, estado, Apex
- `processador.py`: loop manual
- `monitor_service.py` / `service_control.py`: serviço Windows (`MonitorIntegracao`)
- `tests/`: core, config, serviço, `test_ui_theme.py`

## Defaults Tasy (config)

- Pastas: erros / avaliados / Excel sob `F:\Tasy\Saida\…` (ajustável na UI)
- Integration Apex: `TASY`
- Serviço SCM: `MonitorIntegracao`
- Dados: `%PROGRAMDATA%\MonitorIntegracao\` (prod) ou `%LOCALAPPDATA%\MonitorIntegracaoDev\` (dev)
