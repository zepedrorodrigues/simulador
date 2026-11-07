# Requisitos — simulador v1

Estado: análise de requisitos fechada a 2026-11-07, sem decisões de tecnologia.

## O que é

Um formulário único → um pedido a cada simulador público de banco → uma tabela
comparativa, com um PDF por banco. Uma interacção, sem estado, sem conta. O
simulador não pede crédito: só simula, e reencaminha para o simulador do banco.

## Fora de âmbito na v1

Contas e consentimentos · alertas de mudanças · histórico · dados agregados ·
módulo de rendimento/investimento · modelo de sustentação · enquadramento legal ·
pedidos de crédito reais · plano de pagamentos completo · intermediários como
utilizadores · ligação ao banco com dados pré-preenchidos.

## Factos de partida

| # | facto |
|---|---|
| F1 | Bancos portugueses; idealmente todos, arranque aceitável com 10. |
| F2 | Sem anúncios, de imóveis ou de bancos. |
| F3 | Os valores do banco são os que o simulador **do banco** devolve hoje; o que não se obteve não se mostra como obtido. |
| F4 | Nada se guarda: o utilizador digita, o pedido segue para o banco, a resposta volta ao utilizador. |
| F5 | O resultado é indicativo; não fingimos ser o banco. |
| F6 | Portugal, pt-PT, particulares; residentes e não residentes. |

## Requisitos funcionais

| id | requisito | critério de aceitação |
|---|---|---|
| RF1 | Formulário único: valor do imóvel, localização (Continente, Açores, Madeira), montante, prazo, titulares (idade, rendimento **bruto mensal**, encargos de cada), residência, finalidade, modalidade de taxa. | Só o mínimo que um simulador de banco pede; sem documentos nem contratos. Quando um banco pede o rendimento noutra forma (líquido, anual), a conversão é nossa e fica indicada no bloco desse banco. |
| RF2 | Campos condicionais só quando uma regra precisa deles (ex.: 1.ª habitação própria permanente, para as isenções de jovens). | Um campo condicional não aparece a quem não é afectado por ele. |
| RF3 | Todas as finalidades: HPP, secundária, arrendamento, construção, obras, terreno, transferência, multiopções. | Cada banco responde, ou aparece como «não oferece». |
| RF4 | Titulares até ao máximo que algum banco aceite. | Um banco com limite menor aparece como «não oferece». |
| RF5 | Garantia pública para jovens até 35 anos. | Pedida quando a idade a torna elegível, nos bancos que a simulam. |
| RF6 | O utilizador escolhe a modalidade: fixa, variável, mista. | Um banco sem essa modalidade para essa finalidade aparece como «não oferece». |
| RF7 | Duas versões por banco: com e sem bonificações (vendas associadas). | A que o banco não devolver fica «indisponível». |
| RF8 | Bloco por banco: MTIC, TAEG, TAN, Euribor considerada, spread, prestação com e sem seguros, valor dos seguros, e cada comissão. | Todos os valores do bloco vêm da resposta do banco. |
| RF9 | Custos da compra, numa linha à parte que soma ao total: IMT, imposto do selo sobre a aquisição, escritura, registos — com as isenções para jovens quando se aplicam. | Rotulados «calculado pelo simulador», distintos do bloco do banco. |
| RF10 | Taxa variável e mista: Euribor de hoje e três cenários (ver *Cenários*). | Rotulados «calculado pelo simulador», com o valor da Euribor de cada cenário à vista. |
| RF11 | Tabela comparativa; por banco, uma ligação para o detalhe, o PDF e o simulador do banco. | Do resumo ao detalhe e ao banco em um clique cada. Ordenada por prestação; em empate, por MTIC e depois por TAEG. |
| RF12 | PDF por banco: relatório nosso com a resposta do banco. | Diz que não é documento do banco; só existe enquanto o resultado está aberto. |
| RF13 | Os resultados aparecem à medida que cada banco responde. | Um banco lento não atrasa os outros. |
| RF14 | Aviso: indicativo, sem pedido de crédito, os dados seguem para o simulador do banco. | Visível antes de submeter e no resultado. |

### Custos a discriminar (RF8 + RF9)

Comissão de avaliação · comissão de dossier/abertura · comissão de formalização ·
imposto do selo sobre o crédito e sobre os juros · comissão de processamento da
prestação · seguro de vida · seguro multirriscos · IMT · imposto do selo sobre a
aquisição · escritura · registos. A comissão por amortização antecipada é
informativa: não é custo certo, não entra no total.

## Requisitos não funcionais

| id | requisito |
|---|---|
| RNF1 | **Fidelidade**: o bloco do banco só leva valores que o banco devolveu nesse pedido. O que o simulador calcula (RF9, RF10) vai fora do bloco e rotulado. |
| RNF2 | Três estados distintos por célula: **obtido**, **o banco não oferece**, **indisponível** (falhou obter). |
| RNF3 | Nada persiste depois da interacção — nem dados pessoais, nem respostas. |
| RNF4 | Cobertura inicial: os 10 bancos cujo simulador seja mais fácil de reproduzir; os restantes a seguir. |
| RNF5 | Corre no browser. Está planeada uma aplicação: nada no desenho a deve impedir. |

## Cenários de Euribor (RF10)

Na taxa fixa não há cenários; na mista, aplicam-se só ao período variável.

Cada cenário mantém o seu valor **durante todo o prazo** (ou todo o período
variável). O utilizador vê, junto de cada cenário, o valor de Euribor usado, a
regra que o deu e que se mantém constante — é um limite, não uma previsão.

A trajectória de cada indexante (3M, 6M, 12M) calcula-se **com dados desse
indexante**, nunca derivada de outro.

### Conservador e positivo: extremos históricos

Fonte primária: BCE, séries mensais `FM.M.U2.EUR.RT.MM.EURIBOR{3MD_,6MD_,1YD_}.HSTA`
(médias mensais, 1999-01 a 2026-10), descarregadas a 2026-11-07 de
`data-api.ecb.europa.eu`.

| indexante | conservador (máximo) | positivo (mínimo) | último (2026-10) |
|---|---|---|---|
| 3M | 5,113% (2008-10) | −0,582% (2021-12) | 2,635% |
| 6M | 5,219% (2008-09) | −0,545% (2021-12) | 2,975% |
| 12M | 5,393% (2008-07) | −0,505% (2021-01) | 3,239% |

**Euribor negativa**: a Lei n.º 32/2018 obriga a reflectir o indexante negativo
mesmo quando a soma com o spread fica negativa; o banco pode constituir um crédito
a favor do cliente, a deduzir nos juros futuros. O cenário positivo usa portanto o
valor negativo tal como está. Por confirmar: se houve alteração posterior à lei, e
como se trata o caso de a TAN ficar negativa.
([Diário da República](https://diariodarepublica.pt/dr/detalhe/lei/32-2018-115712241))

### Realista: expectativas a 3–5 anos

Fontes primárias encontradas, ambas do BCE e **ambas só para a Euribor a 3M**:

- pressupostos técnicos das projecções macroeconómicas (trimestrais, derivados dos
  futuros); os números de setembro de 2026 ainda não foram lidos;
- Survey of Monetary Analysts (mediana): 2,65% em mar./abr. 2027, 2,18% no
  3.º trim. 2029 (inquérito de 24–26 ago. 2026).

Falta uma fonte própria para o 6M e o 12M: derivá-los do 3M foi recusado. As
curvas de futuros de mercado (ICE, Eurex) são licenciadas.
Fontes secundárias (Bluegamma, curva de swaps) dão ~3,45–3,59% para o 12M em
2027–2029 e divergem entre si.

## Em aberto

1. Quais são os 10 bancos — sai de um levantamento, banco a banco, do que cada
   simulador aceita e devolve. É também esse levantamento que diz onde o RNF1
   obriga a mostrar «indisponível» no RF5, RF7 e RF8.
2. Euribor negativa: confirmar se a Lei n.º 32/2018 teve alterações.
3. Fonte para a trajectória realista do 6M e do 12M (as primárias encontradas só
   dão 3M), e com que frequência se actualizam os três cenários.

## Funcionalidades e MVP

Validado a 2026-11-07.

**MVP**: habitação própria permanente, aquisição, 1–2 titulares, 3 bancos — o
caso que todos os bancos simulam, logo o que prova o circuito
formulário → bancos → tabela com menos risco.

| # | funcionalidade | prioridade | justificação |
|---|---|---|---|
| 1 | Formulário único, campos mínimos | MVP | Sem ele não há produto. |
| 2 | Pedidos em paralelo, resultados à medida que chegam | MVP | Um banco lento não bloqueia os outros. |
| 3 | Bloco por banco (MTIC, TAEG, TAN, Euribor, spread, prestação com e sem seguros, comissões) | MVP | É o resultado pedido. |
| 4 | Três estados por célula | MVP | Sem isto não se cumpre o RNF1. |
| 5 | Tabela comparativa, ligação ao simulador do banco | MVP | A forma de comparar escolhida. |
| 6 | Custos da compra calculados, em linha à parte | MVP | Entram no total; dependem só da lei. |
| 7 | Avisos (indicativo, sem pedido, dados enviados ao banco) | MVP | Compromisso com o utilizador. |
| 8 | Até 10 bancos | v1 | Cada banco é um levantamento à parte. |
| 9 | Com e sem bonificações | v1 | Depende do que cada simulador devolve. |
| 10 | Três cenários de Euribor | v1 | Falta a fonte do realista para 6M e 12M. |
| 11 | PDF por banco | v1 | Detalhe; a tabela já entrega o essencial. |
| 12 | Restantes finalidades | v1 | A cobertura varia muito por banco. |
| 13 | Garantia pública e isenções para jovens até 35 anos | v1 | Exige campos condicionais. |
| 14 | Mais de 2 titulares; não residentes | v1 | Poucos bancos os simulam. |
| 15 | Ordenar e filtrar a tabela | depois | Inútil com 3 bancos. |
| — | Contas, alertas, histórico, ligação pré-preenchida, módulo de investimento, aplicação | fora | Decidido deixar para o futuro. |
