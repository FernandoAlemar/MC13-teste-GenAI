# Testes de integração — API fictícia de produtos do fornecedor

Material acadêmico da Unidade III. A empresa é a Distribuidora Aurora Ltda., fictícia.

O cenário é a integração entre dois componentes:

- consumidor: sistema externo de compras;
- provedor: `GET /protheus/com/produtosfornecedor`.

Não há token de produção, CNPJ, rota real nem fonte do ERP.

## Tipos de prompt

| Arquivo | Tipo | O que pediu |
|---|---|---|
| [prompts/01-contrato-integracao.md](prompts/01-contrato-integracao.md) | Contrato de integração | Caminho esperado entre os dois componentes |
| [prompts/02-erro-limite-dados.md](prompts/02-erro-limite-dados.md) | Erro, limite e dados | Fronteira aceita, primeira fronteira recusada e falhas de contrato |

Cada arquivo guarda a versão vaga, o que ela produziu e a versão ajustada.

## Casos

[casos/integracao-produtos-fornecedor.md](casos/integracao-produtos-fornecedor.md)

São casos da fronteira HTTP: código, mensagem e itens. Esta suíte não compila TLPP e não executa a consulta SQL no banco.