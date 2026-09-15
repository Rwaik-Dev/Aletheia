# Historico de Desenvolvimento - Monitor HP

## 2026-09-15

- Usada a estrutura do `monitor-integracao` como base operacional.
- Confirmado que a recorrencia de 24h deve ser local.
- Confirmado que o `access_token` sera salvo manualmente pela UI no banco local.
- Confirmado que a primeira ocorrencia deve ser enviada imediatamente.
- Criados testes TDD para parser HP, estado de recorrencia, payload e envio com bearer token.
- Adaptados defaults, nome do servico, diretórios de dados e specs PyInstaller para Monitor HP.

## Verificacao

- `python -m pytest tests\\test_monitor_core_hp.py -q`: 5 testes passando durante o desenvolvimento.
- `python -m pytest -q`: suite completa passando no checkpoint inicial.
