# Prompt 2 — erro, limite e dados de teste

Tipo: casos de erro, fronteira e massa da integra??o.

O consumidor e o provedor s?o os mesmos do prompt de contrato. Este prompt n?o repete o caminho feliz. Ele pede onde a fronteira quebra ou fica no limite.

## Vers?o ainda gen?rica

```text
Gere casos de erro e dados de teste para a API de produtos do fornecedor.
```

Resultado observado: mensagens soltas ("dados inv?lidos", "n?o autorizado") sem c?digo HTTP, sem a mensagem do envelope e sem separar 400, 401 e 403. A descri??o com 2 caracteres e o intervalo de 366 dias n?o apareceram. O caso n?o dava para conferir.

## Vers?o ajustada

```text
Gere casos de erro, de limite e a massa de teste da integra??o entre o sistema de compras e GET /protheus/com/produtosfornecedor.

Use somente o contrato e a massa fict?cia da Aurora j? publicados no prompt de contrato. Data corrente: 20261008. N?o invente regra. N?o mostre valor de token diferente do valor de teste aurora-token-teste, e n?o trate esse valor como segredo de produ??o.

A valida??o para na primeira falha, nesta ordem: token, pagina??o, JSON, filial, loja, fornecedor, produto, descri??o e per?odo.

Para cada caso informe a situa??o, o que ? enviado, o Code e a Message exata:
- Par?metro de token vazio: 400, "Parametro de token nao configurado".
- Header sem token: 401, "Header api-token ausente ou vazio".
- Token divergente: 403, "Token diferente do parametro".
- Body JSON inv?lido: 400, "Body JSON invalido".
- Filial curta, loja curta ou fornecedor ausente: 400, com a mensagem do campo.
- Data 2026-09-01: 400, "Periodo invalido".
- lineperpage 100 e intervalo 20251008–20261008: aceitos, Code 200.
- lineperpage 101 e intervalo 20251007–20261008: 400.
- Descri??o com 3 caracteres: aceita. Com 2 caracteres: 400, "descricao invalida".
- P?gina depois do ?ltimo registro: 404, "Nao ha dados para o filtro informado".
- Pedido do produto inativo AUR-INAT-01 na loja 01: 404.
- Pedido de AUR-LOJA-02 na loja 01: 404.

N?o descreva o caso sem Code, Message e, no sucesso, a lista de codigoProduto.
```

## Casos obtidos com a vers?o ajustada

| Situa??o | Code | Message |
|---|---|---|
| lineperpage 100 e intervalo de 365 dias | 200 | Solicitacao concluida - 3 registros |
| Descri??o Ole (3 caracteres) | 200 | Solicitacao concluida - 1 registros |
| P?gina com startline 4 | 404 | Nao ha dados para o filtro informado |
| lineperpage 101 | 400 | Paginacao invalida |
| Intervalo de 366 dias | 400 | Periodo invalido |
| Descri??o Ol (2 caracteres) | 400 | descricao invalida |
| Par?metro de token vazio | 400 | Parametro de token nao configurado |
| Header sem token | 401 | Header api-token ausente ou vazio |
| Token divergente | 403 | Token diferente do parametro |
| Body JSON inv?lido | 400 | Body JSON invalido |
| Filial 01 | 400 | filial invalida |
| Loja 1 | 400 | loja invalida |
| Fornecedor ausente | 400 | codigoFornecedor invalido |
| Data 2026-09-01 | 400 | Periodo invalido |
| Produto inativo AUR-INAT-01 | 404 | Nao ha dados para o filtro informado |
| AUR-LOJA-02 pedido na loja 01 | 404 | Nao ha dados para o filtro informado |

Um token divergente responde 403 mesmo que a filial tamb?m esteja errada, porque a valida??o para na primeira falha.
