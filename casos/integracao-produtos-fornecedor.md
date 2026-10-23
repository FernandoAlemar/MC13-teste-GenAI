# Casos de integra??o — produtos do fornecedor

Fronteira testada: o sistema externo de compras chama `GET /protheus/com/produtosfornecedor`.

Estes casos foram gerados pelos dois prompts ajustados. Eles conferem o contrato HTTP. N?o compilam TLPP e n?o executam `BeginSQL` no ERP.

Massa fict?cia, filial 0101, fornecedor AUR001, data corrente 20261008:

| Item | Situa??o |
|---|---|
| AUR-ARZ-001 | Ativo, loja 01, inclus?o em 01/10/2026 |
| AUR-FEI-002 | Ativo, loja 01, inclus?o em 15/09/2026 |
| AUR-OLEO-900 | Ativo, loja 01, sem pre?o, inclus?o em 01/08/2026 |
| AUR-INAT-01 | Inativo, loja 01 |
| AUR-LOJA-02 | Ativo, loja 02 |

## Gerados pelo prompt de contrato

Diversidade do fluxo entre os dois componentes.

| ID | Caso | Resultado esperado |
|---|---|---|
| C1 | Token v?lido, startline 1, lineperpage 10, loja 01 | 200 e os tr?s ativos, com o ?leo em pre?o 0 e custo 0 |
| C2 | startline 3, lineperpage 2 | 200 e somente AUR-OLEO-900 |
| C3 | descricao fei e per?odo 20260901–20261008 | 200 e somente AUR-FEI-002 |
| C4 | codigoProduto AUR-OLEO-900 | 200 e somente o ?leo |

## Gerados pelo prompt de erro, limite e dados

Diversidade da fronteira: o valor aceito e o primeiro valor recusado.

| ID | Caso | Code |
|---|---|---|
| L1 | lineperpage 100 e per?odo de 365 dias | 200 |
| L2 | descri??o com 3 caracteres | 200 |
| L3 | startline 4 | 404 |
| L4 | lineperpage 101 | 400 |
| L5 | per?odo de 366 dias | 400 |
| L6 | descri??o com 2 caracteres | 400 |
| E1 | par?metro de token vazio | 400 |
| E2 | header sem token | 401 |
| E3 | token divergente | 403 |
| E4 | body JSON inv?lido | 400 |
| E5 | filial com 2 caracteres | 400 |
| E6 | loja com 1 caractere | 400 |
| E7 | fornecedor ausente | 400 |
| E8 | data com h?fen | 400 |
| E9 | produto inativo | 404 |
| E10 | item da loja 02 pedido na loja 01 | 404 |

O sucesso n?o se contenta com Code 200. C1 falha se o inativo ou o item da outra loja aparecerem, ou se o ?leo n?o voltar com os dois pre?os zerados.
