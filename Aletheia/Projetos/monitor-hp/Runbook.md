# Runbook - Monitor HP

## Configurar

```powershell
cd C:\Estudos\monitor-hp
pip install -r requirements.txt
python app.py
```

Preencher:

- Pasta erros
- Pasta analisados
- Pasta relatorios
- URL base Apex
- Client ID e Client Secret (service client Apex)
- Source `10.2.30.53`
- Integration `HP`

## Executar manualmente

```powershell
python processador.py
```

## Servico Windows

Executar como Administrador:

```powershell
python monitor_service.py --startup=auto install
python monitor_service.py start
```

## Testar

```powershell
$env:MONITOR_TEST_CRYPTO='1'
$env:MONITOR_SKIP_ACL='1'
python -m pytest -q
```

## Troubleshooting

- Sem envio: verificar se `access_token` esta preenchido na UI.
- Sem leitura: verificar se existem `.htm` ou `.html` em `PASTA_ERROS`.
- Arquivo nao movido: verificar permissao em `PASTA_AVALIADOS`.
- Encoding estranho: parser tenta `utf-8`, `latin-1` e `cp1252`.
