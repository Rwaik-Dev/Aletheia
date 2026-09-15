# Monitor HP

Cliente Python para Windows que monitora HTMLs do Hermes Pardini, extrai erros iniciados por `Pedido:` e envia ocorrencias para o Apex.

## Estrutura

- `app.py`: interface Tkinter para configuracao, servico e logs.
- `config.py`: modelo tipado com defaults HP.
- `config_repository.py`: SQLite + DPAPI para configuracao e `access_token`.
- `monitor_core.py`: parser HP, estado local, payload e envio Apex.
- `processador.py`: loop manual.
- `monitor_service.py` e `service_control.py`: servico Windows.
- `tests/`: testes de core, configuracao, servico e controle do servico.

## Defaults HP

- Pasta de erros: `C:\Estudos\monitor-hp\erros`
- Pasta de analisados: `C:\Estudos\monitor-hp\analisados`
- Relatorios: `C:\Estudos\monitor-hp\relatorios`
- Apex URL (API): `http://localhost:3333`
- Integration: `HP`
- Source: `10.2.30.53`
- Janela local: 1440 minutos
