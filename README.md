# Pix crédito

Um esboço do que seria a arquitetura do Pix Crédito.

# Índice

- [Personas](#personas)
- [Uso do Pix Crédito](#uso-do-pix-crédito)
  - [Uso do crédito](#uso-do-crédito)
  - [Pagamento da fatura](#pagamento-da-fatura)
  - [Saque do dinheiro](#saque-do-dinheiro)
- [Aprovação de crédito](#aprovação-de-crédito)
- [Riscos para a _issuer_](#riscos-para-a-issuer)

# Personas

São 4 os personas:

- _issuer_ de crédito
- _customer_ de uma transação
- o _shopper_ de um _customer_
- _seller_ de uma transação

O _customer_ de uma transação eventualmente pode ser
o _seller_ em outra transação.

O _shopper_ é um representante do _customer_
autorizado a fazer transações.

A _issuer_ de crédito é quem vai operar e garantir o
Pix Crédito

# Uso do Pix Crédito

O uso do Pix Crédito se faz através de 3 formas:

- [uso do crédito por parte do _customer_ para
  obter bens ou serviços de um
  _seller_](#uso-do-crédito)
- [pagamento da fatura do
  _customer_](#pagamento-da-fatura)
- saque do dinheiro a ser recebido pelo _seller_

Para isso, pressupõe-se que:

- _seller_ é cliente Pix Crédito
- _customer_ é cliente Pix Crédito
- _customer_ já passou por uma análise de risco e
  tem crédito limitado
- _seller_ receberá automaticamente o dinheiro
  devido no dia de saque via conta Pix
- _customer_ configurou seu _shopper_
- todo evento ocorre online
- para todo evento, será mantido um _timestamp_
  indicando quando ocorreu

## Uso do crédito

O _customer_ deseja adquirir um bem ou serviço do
_seller_. _Shopper_ (representando o _customer_) e
_seller_ chegam a uma conclusão de quais valores e
sobre condição de pagamento desses valores. _Seller_
inicia uma transação, indicando:

- valor
- condição de parcelamento
- _shopper_
- _customer_
- identificação do dispositivo usado para criação da
  transação

_Seller_ recebe de volta o código da transação,
valor, condição de parcelamento, _shopper_ e qual o
dispositivo usado para iniciar a transação.

> O dispositivo é sempre informado a fim de auditoria
> e garantia de segurança por parte dos operadores.

Nesse momento, _shopper_ recebe a transação, e os
atores no sistema podem tomar as seguintes ações:

- _shopper_: validar a transação
- _shopper_: negar a transação
- _customer_: negar a transação
- _shopper_: se abster de tomar ação
- _seller_: invalidar a transação

Ao **_shopper_: validar a transação** temos ainda [2
cenários](#aprovação-de-crédito):

- o crédito ser aprovado pela _issuer_
- o crédito ser reprovado pela _issuer_

Se o crédito for aprovado pela _issuer_, então a
transação atualiza o _status_ para "aprovada",
podendo então gerar movimentos.

Porém o crédito pode ser reprovado pela _issuer_.
Nestas circunstâncias, o _shopper_ e o _customer_
serão notificados disto e encorajados a **negar a
transação**, porém a eles será _facultada_ essa
escolha, pois há a possibilidade de normalizar o
crédito disponível.

O fato de tentar validar a transação (seja ela
efetivada com sucesso ou não) implica no envio das
seguintes informações:

- identificação do dispositivo usado para validar a
  transação
- modo de autenticação e validação da autenticação do
  _shopper_ para aquela transação

Ao **_shopper_: negar a transação**, a transação
atualiza o _status_ para "rejeitada pelo _shopper_".
Para negar a transação, é necessário que o _shopper_
forneça:

- identificação do dispositivo usado para negar a
  transação
- modo de autenticação e validação da autenticação do
  _shopper_ para aquela transação

Ao **_customer_: negar a transação**, a transação
atualiza o _status_ para "rejeitada pelo _customer_".
Para negar a transação, é necessário que o _customer_
forneça:

- identificação do dispositivo usado para negar a
  transação
- modo de autenticação e validação da autenticação do
  _customer_ para aquela transação

O _customer_ tem um curto intervalo de tempo para
invalidar uma transação após validada pelo _shopper_.

Também há a possibilidade de **_shopper_: se abster
de tomar ação** e deixar a transação sem reação.
Após determinado tempo, _shopper_ será notificado
sobre a existência da transação em aberto. Após
determinado tempo, o _customer_ será notificado
sobre a existência de transações em aberto pelo
_shopper_ específico.

Finalmente, após determinado tempo, a transação é
cancelada, mudando o _status_ para "tempo de
autenticação execedido". Isso deve ter mesmos efeitos
práticos de negar a transação.

Por fim, há a possibilidade de **_seller_: invalidar
a transação**. Isso pode ocorrer a qualquer momento
antes da validação da transação. Caso ocorra condição
de corrida e a requisição de invalidar a transação
ocorra após a validação pelo _shopper_ dentro de um
determinado intervalor de tempo, a transação passará
para o _status_ "invalidada pelo _seller_".

Para invalidar uma transação, é necessário fornecer:

- identificação do dispositivo usado para negar a
  transação
- modo de autenticação e validação da autenticação do
  _seller_ para aquela transação


Invalidar uma transação deve ter mesmos efeitos
práticos de negar a transação.

Ao negar a transação, quaisquer pré-cálculos sobre
uso do crédito, para esta transação, em _cache_
precisam ser invalidados. O _seller_ é notificado de
que a transação específica não foi concluída.

Toda mudança de _status_ de uma transação precisa ser
armazenada.

## Pagamento da fatura

Uma fatura é composta por movimentos e juros. O
valor da fatura é o total da soma dos movimentos no
intervalo de uso de crédito e a soma dos juros. Além
disso, a fatura tem data de publicação, intervalo de
datas para receber movimentos, movimentos vindos de
parcelamento e data de vencimento.

Movimentos advém de:

- transações  
  nestes casos, o movimento precisa indicar de qual
  transação ele advém
- taxa de serviço  
  taxa da assinatura do Pix Crédito,
  [taxa de saque]((#saque-do-dinheiro))
- juros  
  juros advindos de montante não pago

Será ofertado duas possibilidades:

- pagamento parcial do montante devedor
- pagamento total do montante devedor

Pagamentos são feitos via Pix endereçados a
_issuer_, identificando a fatura ao qual o pagamento
se refere. Pagamentos são identificados como advindos
de transações externas.

As taxas de serviço entram como movimentações
advindas de transações externas, tendo como
beneficiário a _issuer_. Será cobrado um custo de
anuidade para que o _customer_ possa usar os serviços
do Pix Crédito.

Caso não aconteça pagamento o suficiente para cobrar
o montante devedor e os juros no vencimento da
fatura, juros passarão a correr.

## Saque do dinheiro

> TODO

# Aprovação de crédito

Dado um _customer_ que tem limite de crédito `C`,
sendo que já consumiu `D`, e _shopper_ do _customer_
que foi configurado com limite de crédito `C' <= C`
e que consumiu `D' <= D`, uma transação `T` de valor
`T.vr` pode ser aprovada se:

- `D + T.vr <= C`
- `D' + T.vr <= C'`
- `D' < C'`

Caso todas as condições sejam aprovadas, a transação
é devidamente aprovada, mudando de _status_ para
"aprovada".

Caso haja condições de aprovação de crédito não sejam
atendidas, a transação `T` não é aprovada e as
condições que não foram atendidas para aprovar a
transação são informadas ao _customer_ e ao
_shopper_.

De posso da informação do porquê que determinada
transação não foi aprovada, o _customer_ pode tomar
alguma ação:

- pagamento de fatura (diminuindo `D`)
- pedido de aumento de limite de crédito (aumentado
  `C`)
- configurar o _shopper_ para um limite maior
  (aumentando `C'`)
- aprovação individual desta transação

No caso de aprovação individual desta transação,
teremos que `D' > C'`, a transação vai para o
_status_ de "aprovada" e será colocado como modo de
autenticação:

- identificação do dispositivo usado do _shopper_
  para validar a transação
- modo de autenticação e validação da autenticação do
  _shopper_ para aquela transação
- identificação do dispositivo usado do _customer_
  para validar a transação
- modo de autenticação e validação da autenticação do
  _customer_ para aquela transação

Outras formas de aprovação de crédito podem ser
descritas no futuro, como a possibilidade de limitar
valores altos de transações, limitação diária de
_shopper_ ou outra limitação.

# Riscos para a _issuer_

As datas de fatura dos _customers_ podem não
coincidir com as datas de saques dos _sellers_, isso
implica que há o risco de haver saques para os
_sellers_ antes de haver o pagamento. Isso pode ser
mitigado tendo capital de giro. Eventualmente
empréstimos deverão ser adquiridos para que não haja
perda de _trustness_ por parte dos _sellers_.

Também há o risco de o _customer_ não arcar com o seu
gasto do mês, pagando menos do que o montande devedor
total, ou até mesmo menos do que o "mínimo"
calculado. Para esse tipo de situação, enquanto for
sustentável, cobrar os juros para no mínimo cobrir o
gasto com dinheiro empenhado para pagar os _sellers_.

Caso a situação com o _customer_ se torne
insustentável, pode-se vender a dívida do _customer_
para algum serviço de proteção ao crédito para
mitigar o prejuízo.

# Fontes de arrecadação da _issuer_

1. taxa de assinatura do serviço: _customer_
1. taxa de saque: _seller_
1. juros sobre montante devido: _customer_

A principal fonte deve ser focada na _taxa de saque_.
Essa modalidade de arrecadação de dinheiro não se dá
diretamente através da cobrança do _seller_.

> Explicar que se ganha retendo valor pago pelo
> _customer_.