# Prompt 1 — contrato de integra??o

Tipo: cen?rios de integra??o e contrato entre componentes.

Cen?rio: o sistema externo de compras chama a API fict?cia do ERP.

`GET /protheus/com/produtosfornecedor`

Empresa fict?cia: Distribuidora Aurora Ltda. N?o h? token de produ??o, CNPJ, rota real nem dado de cliente.

## Vers?o vaga

```text
Gere testes de integra??o de uma API de produtos.
```

Resultado observado: casos gen?ricos de timeout, login e CRUD. N?o apareceram filial, loja, envelope `Code` / `Message` / `itens`, produto sem pre?o nem a fronteira entre o compras e o ERP. O texto descrevia a ideia do teste e n?o fixava o que a resposta devia conter.

## Vers?o ajustada

```text
Voc? apoia o teste de integra??o entre dois componentes. O consumidor ? o sistema externo de compras. O provedor ? a API fict?cia GET /protheus/com/produtosfornecedor, da Distribuidora Aurora Ltda.

N?o invente rota, segredo, CNPJ nem cliente real. Se a regra de neg?cio n?o estiver neste contrato, pare e pergunte. N?o escreva o teste.

Contrato j? aprovado:
- Header api-token obrigat?rio, comparado ao par?metro MV_APITOK depois de tirar espa?os. O valor de teste ? aurora-token-teste. N?o grave segredo.
- Query startline: inteiro m?nimo 1; aus?ncia vira 1.
- Query lineperpage: inteiro de 1 a 100; aus?ncia vira 100.
- Body: filial com 4 caracteres, loja com 2, codigoFornecedor com 6. codigoProduto ? opcional e exato. descricao ? opcional, parcial, sem diferenciar mai?sculas, de 3 a 40 caracteres. dataInicial e dataFinal v?m juntas ou n?o v?m; formato YYYYMMDD; final maior ou igual ? inicial; final at? 20261008; diferen?a m?xima de 365 dias.
- Produto inativo fica de fora. Produto sem pre?o permanece.
- Cada item devolve codigoProduto, descricao, unidade, precoVenda, custoStandard e codigoFornecedor.
- Sucesso: Code 200 e Message "Solicitacao concluida - N registros".

Massa, toda na filial 0101 e no fornecedor AUR001:
- AUR-ARZ-001, ativo, loja 01, inclus?o 20261001, pre?os 8,50 e 5,00.
- AUR-FEI-002, ativo, loja 01, inclus?o 20260915, pre?os 7,00 e 4,00.
- AUR-OLEO-900, ativo, loja 01, inclus?o 20260801, pre?os 0 e 0.
- AUR-INAT-01, inativo, loja 01.
- AUR-LOJA-02, ativo, loja 02.

Gere somente o caminho esperado da integra??o. Para cada caso informe: quem chama, o que envia (header, query e body), o Code, a Message e a lista de codigoProduto. N?o descreva o cen?rio sem esses campos.
```

## Casos obtidos com a vers?o ajustada

| Caso | Quem chama | Envio | Code | Itens |
|---|---|---|---|---|
| P?gina inicial | Compras | token v?lido, startline 1, lineperpage 10, filial 0101, loja 01, AUR001 | 200 | AUR-ARZ-001, AUR-FEI-002, AUR-OLEO-900 |
| Segunda p?gina | Compras | startline 3, lineperpage 2, mesmos filtros | 200 | AUR-OLEO-900 |
| Descri??o e per?odo | Compras | descricao fei, 20260901 a 20261008 | 200 | AUR-FEI-002 |
| C?digo exato | Compras | codigoProduto AUR-OLEO-900 | 200 | AUR-OLEO-900, com precoVenda 0 e custoStandard 0 |

O ?leo sem pre?o permanece. O inativo e o item da loja 02 n?o entram nessa p?gina. A mensagem do primeiro caso ? `Solicitacao concluida - 3 registros`.
