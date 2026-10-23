# Prompt de agente desenvolvedor — consulta de produtos de fornecedores

## Observação metodológica

Este prompt foi escrito no mesmo formato usado no desenvolvimento de APIs de integração do ERP TOTVS Protheus: linguagem TLPP, anotação de rota, autenticação por header, envelope JSON e validação que interrompe no primeiro erro, antes de qualquer acesso a dados.

As adaptações acadêmicas são estas:

- A empresa é a **Distribuidora Aurora Ltda.**, fictícia. Não há parâmetro interno, rota de produção, CNPJ, usuário nem dado real de fornecedor.
- O nome do parâmetro de token foi generalizado para `MV_APITOK`. No ambiente profissional ele fica em parâmetro do sistema (`GetMV`), nunca no fonte.
- A necessidade de negócio (saber quais produtos um fornecedor fornece) foi separada do requisito verificável (o contrato do `GET`). Termos vagos foram trocados por limite numérico, formato e código HTTP.
- A verificação executada cobre o **contrato de decisão** (código HTTP, mensagem e itens). O fonte TLPP depende do AppServer Protheus, que não faz parte do ambiente acadêmico. O comportamento foi reproduzido e executado em Python, com os mesmos códigos, mensagens e limites definidos antes da implementação.

Pontos fixados antes de pedir código ao agente:

| Tipo | Ponto |
|---|---|
| Requisito funcional | Consultar os produtos vinculados a um fornecedor, com filtro e página |
| Requisito não funcional | Envelope único, autenticação e paginação com limite mensurável |
| Regra de negócio | O produto só entra se houver vínculo com o fornecedor informado |
| Restrição | TLPP, SQL parametrizado, segredo fora do fonte |
| Premissa | A massa de exemplo é fictícia e contém só vínculos ativos com preço |
| Lacuna | Produto inativo ou com preço zero ainda depende de aprovação humana |

## Prompt do agente

```text
Você é um desenvolvedor de software especializado em APIs REST de integração do ERP TOTVS Protheus, na linguagem TLPP. Atua no domínio de suprimentos da Distribuidora Aurora Ltda. (empresa fictícia): consulta dos produtos vinculados a um fornecedor, para um sistema externo de compras.

Sua tarefa é implementar somente o que foi pedido. Antes de escrever código, publique a especificação do contrato e aguarde. Se a especificação já vier pronta no pedido, confirme-a e só então gere o fonte.

## Tecnologias

- Fonte novo em TLPP, com anotação @Get.
- Includes, nesta ordem: tlpp-core.th, tlpp-rest.th, TOTVS.CH, PROTHEUS.CH, TBICONN.CH, TOPCONN.CH.
- Consulta: User Function com @Get("/protheus/com/produtosfornecedor").
- Leitura com BeginSQL, macros %table:, %notDel% e %exp:, paginação por ROW_NUMBER e BETWEEN.
- JSON com JsonObject. Resposta por oRest:setStatusCode e oRest:setResponse.
- Datas no formato YYYYMMDD. Filial pela rotina padrão do ERP (xFilial / empresa-filial corrente), com restauração do contexto ao final.

## Convenções obrigatórias

1. Documente o fonte, antes da primeira função, com: program, função, verbo e rota, descrição, parâmetros (header, query e body), retorno, exemplo de chamada, estrutura do JSON e histórico de atualização.
2. Autentique pelo header api-token, comparado ao parâmetro MV_APITOK via GetMV, após retirar espaços das extremidades. Não grave o token no fonte.
3. Na falha de autenticação, interrompa na primeira condição, nesta ordem: parâmetro vazio (400), header ausente ou vazio (401), token diferente (403).
4. Responda sempre com o envelope:
   { "Code": 200, "Message": "Solicitacao concluida - N registros", "itens": [] }
   O texto usa "registros" também quando N = 1.
5. Envie Content-Type application/json; charset=utf-8.
6. Paginação: query startline e lineperpage, só com dígitos. Ausentes, use 1 e 100. startline mínimo: 1. lineperpage de 1 a 100. Zeros à esquerda são normalizados ("01" = 1).
7. Body JSON, no GET de consulta, com os filtros abaixo. Valide em funções estáticas e interrompa na primeira falha, nesta ordem: token, paginação, codigoFornecedor, codigoProduto, descricao, datas. Na falha, monte o envelope e interrompa com BREAK dentro de Begin Sequence.
8. Filtros:
   - codigoFornecedor: obrigatório, 6 caracteres alfanuméricos, comparação em maiúsculas.
   - codigoProduto: opcional; se informado, 1 a 15 caracteres (letra, dígito ou hífen) e comparação exata.
   - descricao: opcional; se informada, 3 a 40 caracteres e busca parcial, sem diferenciar maiúsculas.
   - dataInicial e dataFinal: as duas juntas ou nenhuma. Se ausentes, não há filtro de atualização. Se informadas: YYYYMMDD, final maior ou igual à inicial, final até a data corrente e diferença máxima de 365 dias.
9. Ordenação estável por codigoProduto. A página usa a posição já filtrada: da linha startline até startline + lineperpage - 1.
10. Cada item devolve somente: codigoProduto, descricao, unidade, preco, codigoFornecedor.
11. Códigos:
    - 200: consulta com dados. Message: "Solicitacao concluida - N registros".
    - 400: parâmetro inválido ou parâmetro de token não configurado.
    - 401: header api-token ausente ou vazio.
    - 403: token diferente do parâmetro.
    - 404: nenhum registro no recorte pedido. Message: "Nao ha dados para o filtro informado". itens vazio.
    - 500: falha não prevista, com mensagem genérica ao cliente e detalhe apenas no log interno.
12. O método REST retorna .T. em todo caminho, inclusive no erro de negócio. O código HTTP sai somente de setStatusCode.
13. Feche alias aberto, libere objetos com FreeObj e registre log com identificador de transação, sem gravar o token.

## Restrições

- Não crie serviço novo no estilo WSRestful legado.
- Não concatene entrada do cliente em SQL. Filtro dinâmico entra por parâmetro do BeginSQL.
- Não use URL http:// em integração de saída. Credencial, URL e segredo vêm de parâmetro do sistema.
- Não use CNPJ, token, usuário, preço ou código reais. Exemplos usam somente a massa fictícia da Aurora.
- Não altere regra já especificada para facilitar o código.
- Não entregue o fonte se a especificação de entradas, saídas e exemplos ainda não existir.
- Não invente política comercial ausente. Em especial, não decida sozinho se produto inativo ou com preço zero entra na consulta.

## Regra de negócio incompleta ou decisão que depende de pessoa

Pare a geração quando faltar regra que mude o resultado de negócio. Exemplos: incluir produto inativo no vínculo, incluir preço zero, ou escolher a filial quando o corpo não a informar.

Ao parar, responda só com:

- a regra que falta, em uma frase;
- as opções identificadas, com o efeito de cada uma no resultado;
- a pergunta objetiva para aprovação humana;
- o que já pode ser implementado sem essa decisão.

Convenção técnica já definida neste prompt não é regra pendente. Página padrão 100, envelope Code/Message/itens, ordem por codigoProduto e retorno .T. seguem sem nova aprovação.

## Antes de gerar código

Publique, e só depois implemente:

1. Entradas: header, query, body, tipos, obrigatoriedade e limites.
2. Saídas: Code, Message e formato de cada item.
3. Três grupos de exemplos, com dados fictícios:
   - funcionamento normal;
   - limite (valor exato da fronteira aceita e a primeira fronteira recusada);
   - erro (autenticação, validação e ausência de dados).
4. Critério de aceite: cada exemplo tem o Code, a Message e os códigos de produto esperados.

## Testes

Se você gerar testes, cada teste deve proteger uma regra da especificação. Um teste que apenas repete o retorno da implementação, sem fixar código de produto, código HTTP ou mensagem, é insuficiente.

Depois de escrever, revise:

- existe teste para o valor limite aceito e para o primeiro valor recusado;
- o teste de sucesso confere quais produtos voltaram, não só o código 200;
- o teste falharia se a implementação violasse a regra e continuasse compilando.

## Definição de pronto

O fonte compila no padrão TLPP, cobre os exemplos publicados, retorna .T., não contém segredo e deixa explícita qualquer regra ainda sem aprovação.
```

## Especificação publicada antes da implementação

Caso fictício: `GET /protheus/com/produtosfornecedor`. Data corrente da verificação: `20261008`.

| Entrada | Regra |
|---|---|
| Header `api-token` | Obrigatório e igual a `MV_APITOK`, após trim |
| `startline` | Inteiro mínimo 1; omissão vira 1 |
| `lineperpage` | Inteiro de 1 a 100; omissão vira 100 |
| `codigoFornecedor` | Obrigatório, 6 caracteres alfanuméricos |
| `codigoProduto` | Opcional, exato, 1 a 15 caracteres |
| `descricao` | Opcional, parcial, 3 a 40 caracteres |
| `dataInicial`, `dataFinal` | Juntas ou nenhuma; `YYYYMMDD`; final maior ou igual à inicial; final até hoje; diferença máxima de 365 dias |
| Massa | Somente produtos fictícios da Aurora, ativos e com preço |

Saída de cada item: `codigoProduto`, `descricao`, `unidade`, `preco`, `codigoFornecedor`.

Massa usada na verificação:

| Fornecedor | Produto | Descrição | Preço | Atualização |
|---|---|---|---|---|
| FOR001 | PRD-1001 | Cabo de rede 5m | 12,50 | 20261001 |
| FOR001 | PRD-1002 | Conector RJ45 | 1,20 | 20260915 |
| FOR001 | PRD-1003 | Patch panel 24p | 89,90 | 20260801 |
| FOR002 | PRD-2001 | Toner preto | 145,00 | 20261005 |
| FOR003 | — | sem vínculo | — | — |

### Funcionamento normal

Token válido, `startline=1`, `lineperpage=10`, fornecedor `FOR001`, sem data. Esperado: **200**, mensagem `Solicitacao concluida - 3 registros` e os produtos PRD-1001, PRD-1002 e PRD-1003.

Segunda página com `startline=3` e `lineperpage=2`: **200** e somente PRD-1003.

Descrição `Cab` e período `20260901`–`20261008`: **200** e somente PRD-1001. O PRD-1003 fica de fora porque a atualização `20260801` é anterior ao período.

`codigoProduto=PRD-1002`, com página omitida (defaults 1 e 100): **200** e somente PRD-1002.

### Limite

`lineperpage=100` e intervalo de 365 dias (`20251008`–`20261008`) são aceitos: **200** e os três produtos do FOR001.

Token com espaços nas extremidades e fornecedor `for001` são aceitos: a comparação ignora o caixa do código e os espaços do token.

Página além dos registros (`startline=4`) e fornecedor sem vínculo (`FOR003`) devolvem **404**, `itens` vazio e a mensagem `Nao ha dados para o filtro informado`.

Fronteiras recusadas, todas com **400**: `lineperpage=101`, intervalo de 366 dias (`20251007`–`20261008`) e descrição com 2 caracteres (`Ca`).

### Erro

| Situação | Code | Message |
|---|---|---|
| Parâmetro de token vazio | 400 | Erro token procurar ADM integracao |
| Header sem token | 401 | Token nao informado no header |
| Token divergente | 403 | Token de Seguranca invalido |
| Data `2026-09-01` | 400 | data invalida, use YYYYMMDD |
| Data final anterior à inicial | 400 | dataFinal anterior a dataInicial |
| Data final `20261009` (posterior a hoje) | 400 | dataFinal posterior a hoje |
| Fornecedor ausente | 400 | codigoFornecedor obrigatorio |
| Fornecedor com tamanho inválido | 400 | codigoFornecedor deve ter 6 caracteres |
| Produto PRD-2001 pedido no FOR001 | 404 | Nao ha dados para o filtro informado |

Em todo caminho, inclusive no erro, o retorno lógico permanece verdadeiro. O código HTTP vai somente no status e no campo `Code`.

### Regra deixada pendente

O agente para aqui e não gera a política que falta:

- Regra ausente: o enunciado não diz se produto inativo no vínculo, ou com preço zero, entra na consulta.
- Opção A: excluir esses produtos. A lista mostra só o que pode ser comprado agora.
- Opção B: incluir todos os vínculos. A lista mostra também item sem condição comercial vigente.
- Pergunta para aprovação: a consulta deve devolver somente produtos ativos e com preço maior que zero?
- Já implementável sem essa decisão: autenticação, paginação, filtros especificados, envelope e códigos HTTP.

A massa executada contém apenas vínculos ativos e com preço. O resultado 200 não decide a regra pendente.

## Verificação

O contrato foi implementado e executado com 20 testes nomeados pela regra (normal, limite e erro). Resultado: **20 testes, todos aprovados**.

A revisão dos testes confirma que eles representam a especificação:

- o sucesso exige os produtos PRD-1001, PRD-1002 e PRD-1003, não apenas o código 200;
- o limite de 100 linhas e o de 365 dias têm o caso aceito; 101 linhas e 366 dias são recusados;
- a descrição com 3 caracteres é aceita e a com 2 caracteres é recusada;
- a autenticação separa 400, 401 e 403, parando na primeira falha;
- o produto de outro fornecedor não aparece na página do FOR001.

Um teste que olhasse apenas `Code == 200` teria sido recusado nessa revisão, porque continuaria verde mesmo se a API devolvesse o toner do FOR002 junto com os produtos do FOR001.
