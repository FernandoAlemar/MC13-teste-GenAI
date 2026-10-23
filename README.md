# Testes de integra??o — API fict?cia de produtos do fornecedor

Material acad?mico da Unidade III. A empresa ? a Distribuidora Aurora Ltda., fict?cia.

O cen?rio ? a integra??o entre dois componentes:

- consumidor: sistema externo de compras;
- provedor: `GET /protheus/com/produtosfornecedor`.

N?o h? token de produ??o, CNPJ, rota real nem fonte do ERP.

## Tipos de prompt

| Arquivo | Tipo | O que pediu |
|---|---|---|
| [prompts/01-contrato-integracao.md](prompts/01-contrato-integracao.md) | Contrato de integra??o | Caminho esperado entre os dois componentes |
| [prompts/02-erro-limite-dados.md](prompts/02-erro-limite-dados.md) | Erro, limite e dados | Fronteira aceita, primeira fronteira recusada e falhas de contrato |

Cada arquivo guarda a vers?o vaga, o que ela produziu e a vers?o ajustada.

## Casos

[casos/integracao-produtos-fornecedor.md](casos/integracao-produtos-fornecedor.md)

S?o casos da fronteira HTTP: c?digo, mensagem e itens. Esta su?te n?o compila TLPP e n?o executa a consulta SQL no banco.
