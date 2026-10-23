# Casos de integração — produtos do fornecedor

Fronteira testada: o sistema externo de compras chama `GET /protheus/com/produtosfornecedor`.

Estes casos foram gerados pelos dois prompts ajustados. Eles conferem o contrato HTTP. Não compilam TLPP e não executam `BeginSQL` no ERP.

Massa fictícia, filial 0101, fornecedor AUR001, data corrente 20261008:

| Item | Situação |
|---|---|
| AUR-ARZ-001 | Ativo, loja 01, inclusão em 01/10/2026 |
| AUR-FEI-002 | Ativo, loja 01, inclusão em 15/09/2026 |
| AUR-OLEO-900 | Ativo, loja 01, sem preço, inclusão em 01/08/2026 |
| AUR-INAT-01 | Inativo, loja 01 |
| AUR-LOJA-02 | Ativo, loja 02 |

## Gerados pelo prompt de contrato

Diversidade do fluxo entre os dois componentes.

| ID | Caso | Resultado esperado |
|---|---|---|
| C1 | Token válido, startline 1, lineperpage 10, loja 01 | 200 e os três ativos, com o óleo em preço 0 e custo 0 |
| C2 | startline 3, lineperpage 2 | 200 e somente AUR-OLEO-900 |
| C3 | descricao fei e período 20260901–20261008 | 200 e somente AUR-FEI-002 |
| C4 | codigoProduto AUR-OLEO-900 | 200 e somente o óleo |

## Gerados pelo prompt de erro, limite e dados

Diversidade da fronteira: o valor aceito e o primeiro valor recusado.

| ID | Caso | Code |
|---|---|---|
| L1 | lineperpage 100 e período de 365 dias | 200 |
| L2 | descrição com 3 caracteres | 200 |
| L3 | startline 4 | 404 |
| L4 | lineperpage 101 | 400 |
| L5 | período de 366 dias | 400 |
| L6 | descrição com 2 caracteres | 400 |
| E1 | parâmetro de token vazio | 400 |
| E2 | header sem token | 401 |
| E3 | token divergente | 403 |
| E4 | body JSON inválido | 400 |
| E5 | filial com 2 caracteres | 400 |
| E6 | loja com 1 caractere | 400 |
| E7 | fornecedor ausente | 400 |
| E8 | data com hífen | 400 |
| E9 | produto inativo | 404 |
| E10 | item da loja 02 pedido na loja 01 | 404 |

O sucesso não se contenta com Code 200. C1 falha se o inativo ou o item da outra loja aparecerem, ou se o óleo não voltar com os dois preços zerados.