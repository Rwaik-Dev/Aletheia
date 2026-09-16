# Regras de Negocio - Monitor HP

## Identificacao do erro

Um erro HP valido e uma linha HTML cuja mensagem inicia com `Pedido:`.

Campos extraidos:

- `pedido`: texto entre aspas apos `Pedido:`.
- `exm_pardini`: codigo apos `exame`.
- `cod_formato_certo`: primeiro `CodigoFormato` da mensagem.
- `cod_formato_errado`: ultimo `CodigoFormato` da mensagem.
- `arquivo`: caminho XML na coluna do HTML.
- `data_hora`: data/hora da coluna do HTML.

## Recorrencia 24h

A consulta das ultimas 24 horas e local. A chave local e o fingerprint Apex incluem `pedido + exm_pardini + cod_formato_errado`, para nao colapsar pedidos diferentes do mesmo exame.

A primeira ocorrencia e enviada imediatamente e tambem registrada no estado local. Novas ocorrencias da mesma chave dentro da janela atualizam `ultima_ocorrencia`, reabrem exportacao e entram no historico `ocorrencias`.

## Payload Apex

Endpoint: `POST {APEX_URL}/api/v1/errors/ingest`

Header: `Authorization: Bearer <accessToken>` obtido via `POST /api/v1/auth/token` (service client)

Payload raiz:

```json
{
  "integration": "HP",
  "source": "10.2.30.53",
  "resolveAbsent": false,
  "errors": []
}
```

Cada erro usa `message = Resultado de exame não importado` e `errorCode = HP_CONN_001`.

### Metadata enviado ao Apex

Objeto `metadata` por erro (JSON livre no ingest, máx. 4 KB):

```json
{
  "pedido": "B0P6SPR",
  "chave_local": "d0c350c13bd256b5116111567486325f",
  "exm_pardini": "A-HIP",
  "arquivo_origem": "LOG_HPARDINI_17082026002134.HTML",
  "cod_formato_certo": "25117",
  "cod_formato_errado": "8317"
}
```

- `exm_pardini` **não** é o código interno do RP; o Apex resolve os exames reais consultando MSSQL antes do Google Chat.
- Ver tratamento no Apex: [[Projetos/apex/Regras de Negocio#HP_CONN_001 — divergência de layout (Hermes Pardini)|HP_CONN_001 no Apex]].
