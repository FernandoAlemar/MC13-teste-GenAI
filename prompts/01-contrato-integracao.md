# Prompt 1 — contrato de integração

Tipo: cenários de integração e contrato entre componentes.

Cenário: o sistema externo de compras chama a API fictícia do ERP.

`GET /protheus/com/produtosfornecedor`

Empresa fictícia: Distribuidora Aurora Ltda. Não há token de produção, CNPJ, rota real nem dado de cliente.

## Versão vaga

```text
Gere testes de integração de uma API de produtos.
```

Resultado observado: casos genéricos de timeout, login e CRUD. Não apareceram filial, loja, envelope `Code` / `Message` / `itens`, produto sem preço nem a fronteira entre o compras e o ERP. O texto descrevia a ideia do teste e não fixava o que a resposta devia conter.

## Versão ajustada

```text
Você apoia o teste de integração entre dois componentes. O consumidor é o sistema externo de compras. O provedor é a API fictícia GET /protheus/com/produtosfornecedor, da Distribuidora Aurora Ltda.

Não invente rota, segredo, CNPJ nem cliente real. Se a regra de negócio não estiver neste contrato, pare e pergunte. Não escreva o teste.

Contrato já aprovado:
- Header api-token obrigatório, comparado ao parâmetro MV_APITOK depois de tirar espaços. O valor de teste é aurora-token-teste. Não grave segredo.
- Query startline: inteiro mínimo 1; ausência vira 1.
- Query lineperpage: inteiro de 1 a 100; ausência vira 100.
- Body: filial com 4 caracteres, loja com 2, codigoFornecedor com 6. codigoProduto é opcional e exato. descricao é opcional, parcial, sem diferenciar maiúsculas, de 3 a 40 caracteres. dataInicial e dataFinal vêm juntas ou não vêm; formato YYYYMMDD; final maior ou igual à inicial; final até 20261008; diferença máxima de 365 dias.
- Produto inativo fica de fora. Produto sem preço permanece.
- Cada item devolve codigoProduto, descricao, unidade, precoVenda, custoStandard e codigoFornecedor.
- Sucesso: Code 200 e Message "Solicitacao concluida - N registros".

Massa, toda na filial 0101 e no fornecedor AUR001:
- AUR-ARZ-001, ativo, loja 01, inclusão 20261001, preços 8,50 e 5,00.
- AUR-FEI-002, ativo, loja 01, inclusão 20260915, preços 7,00 e 4,00.
- AUR-OLEO-900, ativo, loja 01, inclusão 20260801, preços 0 e 0.
- AUR-INAT-01, inativo, loja 01.
- AUR-LOJA-02, ativo, loja 02.

Gere somente o caminho esperado da integração. Para cada caso informe: quem chama, o que envia (header, query e body), o Code, a Message e a lista de codigoProduto. Não descreva o cenário sem esses campos.
```

## Casos obtidos com a versão ajustada

| Caso | Quem chama | Envio | Code | Itens |
|---|---|---|---|---|
| Página inicial | Compras | token válido, startline 1, lineperpage 10, filial 0101, loja 01, AUR001 | 200 | AUR-ARZ-001, AUR-FEI-002, AUR-OLEO-900 |
| Segunda página | Compras | startline 3, lineperpage 2, mesmos filtros | 200 | AUR-OLEO-900 |
| Descrição e período | Compras | descricao fei, 20260901 a 20261008 | 200 | AUR-FEI-002 |
| Código exato | Compras | codigoProduto AUR-OLEO-900 | 200 | AUR-OLEO-900, com precoVenda 0 e custoStandard 0 |

O óleo sem preço permanece. O inativo e o item da loja 02 não entram nessa página. A mensagem do primeiro caso é `Solicitacao concluida - 3 registros`.