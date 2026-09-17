# Regras de negócio — Monitor SOCOR (MV)

## Arquivos monitorados

- Padrão: `*_alerta.txt` na pasta configurada (`pasta_erros`)
- Ignorados: subpastas (apenas arquivos no nível raiz da pasta)
- Após processamento: movidos para `{pasta_erros}/analisados/` (sufixo timestamp se nome colidir)

## Parse do arquivo

1. Lê blocos que começam com `ER -` até encontrar `**** Fim` (primeira ocorrência encerra leitura de novos blocos no restante do arquivo conforme implementação em `monitor/parser.py`).
2. Data/hora: extraída da linha `**...:` com formato `dd/mm/yyyy HH:MM:SS` no cabeçalho do log; aplicada a todos os erros do arquivo.
3. Linha de negócio esperada:

```text
ER - Paciente {codigo}, OS {os}, Item {item} - {descricao}
```

4. Blocos **SQL** (`SQLSTATE`, `SQL Server`, `Native Client`) ou **XML** (`Estrutura do arquivo`, `.xml`): paciente/OS/item = `N/A`, descrição = corpo do bloco.

## Classificação (`errorCode` enviado ao Apex)

| `tipo_erro` | Critério (descrição normalizada) |
|-------------|----------------------------------|
| `amostra_invalida` | contém "amostra" e "invalida" |
| `servico_invalido` | contém "servico" e "invalido" |
| `item_inativo` | "nao esta ativo" (ou variante acentuada) |
| `item_preco` | não encontrado na tabela de preços |
| `outros` | marcadores SQL/XML ou estrutura de arquivo |
| `desconhecido` | demais casos |

SQL/XML são classificados como `outros` quando parseados como erro técnico.

## Estado local (`estado_erros.json`)

- Chave MD5 de: `tipo_erro | paciente | os | item | descricao` (normalizados)
- Cada ciclo: registra/atualiza erros lidos dos arquivos; marca `ultima_atividade`
- **Expiração:** erros sem atividade por `intervalo_inatividade_minutos` são removidos do estado antes do envio
- Snapshot enviado = apenas erros **ativos** após expiração

Diferente do monitor HP: não há janela de recorrência 24h; o modelo é snapshot + `resolveAbsent: true` no Apex.

## Payload Apex

Endpoint: `POST {APEX_URL}/api/v1/errors/ingest`

Header: `Authorization: Bearer <accessToken>` (service client)

Raiz:

```json
{
  "integration": "MV",
  "source": "<hostname ou configurado>",
  "resolveAbsent": true,
  "errors": []
}
```

Cada erro:

| Campo Apex | Origem |
|------------|--------|
| `message` | `Paciente X, OS Y, Item Z - descricao` ou só descrição se N/A |
| `errorCode` | `tipo_erro` (classificador) |
| `filePath` | fingerprint estável: paciente\|os\|item\|descricao\|tipo (normalizado) |
| `occurredAt` | data do cabeçalho do log (ISO UTC), se parseável |
| `metadata` | `paciente`, `os`, `exame` (item), `arquivo_origem`, `chave_local` |

`resolveAbsent: true` faz o Apex tratar erros que sumiram do snapshot como resolvidos/ausentes (comportamento de snapshot completo).

## Contrato completo

Ver [[Projetos/apex/API - Guia Novos Clients|Guia Novos Clients]] e [[Projetos/apex/Regras de Negocio|Regras de Negócio — Apex]].
