# Levantamento dos simuladores dos bancos

Medido a 2026-11-07, no browser e com `curl`, sobre os simuladores públicos. Serve o
RNF4: escolher os bancos pela facilidade de reproduzir a simulação.

**Legenda da facilidade**
- **fácil**: um pedido HTTP, sem sessão nem desafio anti-robô;
- **médio**: há uma API JSON, mas exige token, sessão ou vários passos;
- **difícil**: há estado no servidor ou um desafio anti-robô;
- **sem simulador**: não há simulação pública.

| banco | simulador | como calcula | anti-robô no cálculo | facilidade | medido |
|---|---|---|---|---|---|
| CGD | `simuladorch.cgd.pt` | `POST /calculate`, formulário → JSON | nenhum | **fácil** | cálculo ponta a ponta, também por `curl` |
| Banco CTT | `simuladorch.bancoctt.pt` | `POST /api/simulation/simulate`, JSON | nenhum (reCAPTCHA só no contacto e na proposta) | **fácil** | código do site; cálculo não executado |
| Santander | `santander.pt/credito-habitacao/simulador-credito-habitacao` | token OAuth com `client_id` público → `POST …/home_simulations/get_by_rates` | nenhum (reCAPTCHA só ao recuperar simulação guardada) | médio | código do site; cálculo não executado |
| Millennium bcp | `millenniumbcp.pt/credito/credito-habitacao/simulador` | `POST …/api/rpc/GenerateSimulation` (Azure API Management, chave pública no código) | nenhum (captcha só no envio de documentos) | médio | código do site; cálculo não executado |
| Novo Banco | `novobanco.pt/particulares/credito-habitacao/simulador-credito-habitacao` | REST `…/simhb/site/simulacao…`, vários passos (criar → actualizar → calcular) | nenhum encontrado | médio | código do site; cálculo não executado |
| Crédito Agrícola | `creditoagricola.pt/particulares/credito/simulador-credito-habitacao` | `POST /api/credit/calculator`, JSON | nenhum no cálculo | médio | por `curl` sem browser, a API responde 302 → 404 |
| Montepio | `simuladores.bancomontepio.pt/ITSCredit.External/…/MTSIMULATOR` | gateway ITSCredit `Calculator/api/Calculator/Calculate` | nenhum | médio | por `curl` sem browser, responde 410 (exige sessão) |
| UCI | `uci.pt/pt/simulador-credito-habitacao/` | `POST /api/sitecore/MortgagePaymentCalculatorPT/Calculate` | reCAPTCHA presente na página; por medir se protege o cálculo | médio | só os pedidos à rede |
| BPI | `bancobpi.pt/…/simulador-credito-habitacao` | OutSystems Web: *postback* com `__OSVSTATE` (~9 KB de estado) | nenhum | **difícil** | formulário e estado medidos |
| Bankinter | `banco.bankinter.pt/such/credito-habitacao/…` | API JSON same-origin | **Cloudflare** (`cf-mitigated: challenge`, 403 por `curl`) | **difícil** | resposta medida; pede nome e telemóvel no 1.º passo |
| ABANCA | — | só exemplos representativos fixos e telefone | — | **sem simulador** | página medida |
| Banco Best | — | «Simule na App»; no site só aparece o login | — | **sem simulador** no site | página medida; por confirmar se a app simula sem conta |

## CGD — o que a resposta traz (medido)

Pedido: aquisição, habitação própria permanente, imóvel de 250 000 €, empréstimo de
200 000 €, 30 anos, taxa variável. O simulador expresso **não pede idades nem
rendimentos**.

| campo | sem bonificações (`BaseResult`) | com bonificações (`DiscountedResult`) |
|---|---|---|
| prestação | 992,68 € | 912,10 € |
| Euribor 6M | 2,975% | 2,975% |
| spread | 1,350% | 0,650% |
| TAN | 4,325% | 3,625% |
| TAEG | 4,9% | 4,3% |
| MTIC | 378 167,66 € | 348 702,94 € |

O bloco `Fees` traz: comissão de análise 226,20 €, de avaliação 239,20 €, de
formalização 202,80 €, seguro de vida 16,72 €/mês e multirriscos 14,14 €/mês. Há
ainda a comissão mensal de conta, 6,55 €.

Cada cálculo devolve um código, e `/{código}/FINE` entrega a **FINE do banco em
PDF** (1,37 MB, `application/pdf`).

A Euribor 6M da resposta (2,975%) coincide com a média mensal de outubro de 2026
na série do BCE (2,975%).

`/limits` devolve os limites do produto: rácio financiamento/valor até 90%, idade de
18 a 75 anos (75 no fim do prazo), empréstimo de 5 000 € a 1 000 000 €, prazo de 1 a
40 anos, e as combinações de taxa e indexante aceites.

## O que isto muda nos requisitos

- **RF7 (com e sem bonificações)**: a CGD devolve as duas versões numa só resposta.
  No Santander, o código do site já separa `bonifiedPlanDetails` de
  `nonBonifiedPlanDetails`.
- **RF12 (PDF)**: a CGD dá a FINE oficial. O nosso relatório pode remeter para ela,
  sem a apresentar como sua.
- **RF1 (formulário único)**: os bancos não pedem os mesmos dados. A CGD não pede
  idade nem rendimento; o BCP pede profissão e rendimento; o Bankinter pede nome e
  telemóvel. Os campos que um banco não usa não seguem para ele.
- **RNF3 (nada se guarda do nosso lado)**: a CGD, e provavelmente outros, guardam a
  simulação **do lado deles**, porque cada cálculo cria um código recuperável. Não
  é guardado por nós, mas o aviso do RF14 deve dizê-lo.

## Proposta para os 3 bancos do MVP

1. **CGD**: fácil, medida ponta a ponta, e é o maior banco do crédito à habitação.
2. **Banco CTT**: fácil pelo código do site; falta executar um cálculo real.
3. **Santander**: médio, mas está entre os quatro maiores e tem as duas versões
   (com e sem bonificações).

O Millennium bcp é a alternativa ao Santander, com o mesmo esforço.

## Por medir

- Executar um cálculo real no CTT, no Santander e no BCP.
- Saber o que o Crédito Agrícola e o Montepio exigem para a API responder fora do
  browser (cabeçalhos, sessão).
- Saber se o reCAPTCHA da UCI protege o cálculo ou só a FINE.
- Saber se a app do Banco Best simula sem conta.
- Os campos `Origination…` aparecem na CGD e no CTT, e o Montepio corre em
  ITSCredit. Se for o mesmo fornecedor, o trabalho reaproveita-se — é uma
  hipótese, não está verificada.
