# Tipos de remessas no Simple

## Realizando 'Envio de Remessa' pelo Simple
​
O envio de Remessa trata-se do envio de moeda nacional para estrangeira em conta no exterior.
Atualmente, o Simple realiza o envio de remessa com duas naturezas de operação: para a conta do próprio cliente no exterior ou para um beneficiário.
​
## Realizando 'Recebimento de Remessa' pelo Simple

O recebimento de remessas trata-se do recebimento de moeda nacional, vinda de moeda estrangeira em conta no exterior.
​
# Integrando um simulador externo com a funcionalidade de "Envio de remessas" do SIMPLE
​
Primeiro, é necessário ter em mãos o seu **correspondent_identifier** e **correspondent_id**. Essas informações são cruciais para realizar requisições na API da SIMPLE.
​
- **correspondent_identifier** (_String_) é um nome, um identificador único para cada correspondente associado ao SIMPLE.
​
> Exemplo: matriz
​
- **correspondent_id** (_Integer_) é o ID associado ao correspondente
​
> Exemplo: 1
​
## Acessando informações sobre moedas, taxas, cotações e valores
​
**Não é necessário fazer nenhum cálculo no frontend**, já que o endpoint de _EXCHANGES_ é o responsável por trazer todas as informações relacionadas à moedas, taxas, cotações, valores em real e valores em moeda estrangeira. O conjunto dessas informações retornadas pela API pode ser chamado de **cotação de uma remessa**.
​

Como mencionado as remessas se difereciam entre **envio** e **recebimento**, no contexto de rota da API serão lidadas respectivamente como `/outbound` e `/inbound`.

O prefixo base para simular qualquer cotação de uma remessa é o seguinte: 
```
https://api.frentecorretora.com.br/v1/exchanges/remittance
```

Logo após o prefixo é informado o tipo da remessa(outbound ou inbound):
```
https://api.frentecorretora.com.br/v1/exchanges/remittance/outbound
ou
https://api.frentecorretora.com.br/v1/exchanges/remittance/inbound
```

Após informar o tipo da remessa, será necessário informar a forma com a qual será calculada a cotação. A API da SIMPLE suporta 2 tipos de cálculos para remessas:
​
- de real (BRL) para moeda estrangeira
- de moeda estrangeira para real (BRL)

**IMPORTANTE**
- **Outbound**, suporta Euro (EUR) e Dólar Americano (USD) como moeda estrangeira.
- **Inbound**, suporta Euro (EUR), Dólar Americano (USD) e Libra Esterlina (GBP) como moeda estrangeira.
​

Quando o cálculo é feito de real para moeda estrangeira, adicionamos o sufixo `/reverse`.
​
```
// Para outbound
https://api.frentecorretora.com.br/v1/exchanges/remittance/outbound/reverse

// Para inbound
https://api.frentecorretora.com.br/v1/exchanges/remittance/inbound/reverse
```
​
Quando o cálculo é feito de moeda estrangeira para real, adicionamos o sufixo `/`.
​
```
// Para outbound
https://api.frentecorretora.com.br/v1/exchanges/remittance/outbound/

// Para inbound
https://api.frentecorretora.com.br/v1/exchanges/remittance/inbound/
```

Essa distinção entre rotas (`/reverse` e `/`) acontece pois as taxas de spread podem variar, dependendo do tipo de cálculo usado.
​

Há alguns parâmetros obrigatórios a serem passados no endpoint, para que as informações desejadas sejam retornadas na requisição:
​
- **purposeCode**	(_String_): Indica a natureza da operação de remessa. São 2 valores possíveis:
  - IR001: remessa será realizada para a própria pessoa
  - IR002: remessa será enviada para outro beneficiário
​
- **currency** (_String_): Código da moeda que o usuário deseja comprar
  - Alguns exemplos são USD, EUR, GBP.
​
- **correspondentId**	(_Integer_): ID do correspondente em questão
​
- **value**	(_Integer_): valor em reais (BRL) que o usuário deseja comprar. Esse valor deve ser multiplicado por 10⁴ antes de ser enviado.
  - R$ 4.000 deve ser enviado como 40000000
  - R$ 1.500 deve ser enviado como 15000000
  - R$ 560 deve ser enviado como 5600000

​
Para conversões de BRL (real) para moeda estrangeira:

```
// Para outbound
https://api.frentecorretora.com.br/v1/exchanges/remittance/outbound/reverse?purposeCode=IR001&currency=USD&correspondentId=1&value=15000000

// Para inbound
https://api.frentecorretora.com.br/v1/exchanges/remittance/inbound/reverse?purposeCode=IR001&currency=USD&correspondentId=1&value=15000000
```

Para conversões de moeda estrangeira para BRL (real):

```
// Para outbound
https://api.frentecorretora.com.br/v1/exchanges/remittance/outbound?purposeCode=IR001&currency=USD&correspondentId=1&value=15000000

// Para inbound
https://api.frentecorretora.com.br/v1/exchanges/remittance/inbound?purposeCode=IR001&currency=USD&correspondentId=1&value=15000000
```

​
Um exemplo de payload de resposta seria:
​
```js
{
  "id": "e37be94632986caf21bf8ce8924dd98402578e12",
  "correspondentId": 3,
  "purposeCode": "IR001",
  "currency": {
    "code": "USD",
    "name": "Dólar Americano",
    "commercialExchangeRate": {
      "divisor": 10000,
      "updatedAt": "2020-04-23T19:28:19.000Z",
      "value": 55030
    },
    "countryFlagUrl": "https://s3.amazonaws.com/frente-exchanges/flags/united-states.svg",
    "levelingRate": {
      "divisor": 10000,
      "value": 55030
    },
    "maxValue": 3000,
    "minValue": 100,
    "offer": {
      "divisor": 100,
      "value": 1000000
    },
    "price": {
      "withoutTax": {
        "divisor": 10000,
        "value": 56131
      },
      "withTax": {
        "divisor": 10000,
        "value": 56748
      }
    },
    "spreadPercentage": 2
  },
  "tax": {
    "bankFee": {
      "valueUSD": "20",
      "total": {
        "divisor": 100,
        "value": 0
      }
    },
    "iof": {
      "percentage": 1.1,
      "total": {
        "divisor": 100,
        "value": 61700
      }
    }
  },
  "total": {
    "withoutTax": {
      "divisor": 100,
      "value": 5613100
    },
    "withTax": {
      "divisor": 100,
      "value": 5674800
    }
  },
  "type": "OUTBOUND",
  "clamp": "MINIMUM",
  "timeToLive": 600,
  "expiresAt": "2020-04-21T22:41:05.000Z",
  "createdAt": "2020-04-21T22:31:05.186Z"
}
```

#### Sobre a relação entre value e divisor

Note que no payload de exemplo acima, alguns campos possuem uma propriedade `value` e `divisor`. O `divisor` é responsável por formatar o valor de `value` para a exibição no front-end da aplicação.

Então por exemplo, se quisermos exibir o valor total de uma transação:

```js
total: {
  withTax: {
    value: 14836951,
    divisor: 10000,
  }
}
```

Faríamos o seguinte cálculo:

```
const formattedTotal = `R$ (total.withTax.value / total.withTax.divisor)`

// R$ 1483.6951
```

### Explicando o payload de EXCHANGES

- **id**: ID da remessa

- **correspondentId**: ID do correspondente que foi passado como parâmetro

- **reverse**: Indica como foi calculada a cotação, se foi de moeda nacional para enstrangeira e vice-versa

- **purposeCode**: natureza da operação
  - **IR001**: remessa para o próprio cliente
  - **IR002**: remessa para um beneficiário terceiro

- **currency**: `Object` contendo informações sobre a moeda
  - **code**: sigla da moeda escolhida pelo usuário
  - **name**: nome da moeda
  - **commercialExchangeRate** `Object` cotação comercial para a moeda atual
    - **divisor**: divisor usado para formatar o `value`
    - **updatedAt**: data de atualização da cotação comercial atual em UTC
    - **value**: valor total da cotação comercial
  - **countryFlagUrl**: url para buscar a bandeira da moeda escolhida, no formato svg
  - **levelingRate**: `Object` taxa de nivelamento da cotação (que pode ser igual ao `commercialExchangeRate`)
    - **divisor**: divisor usado para formatar o `value`
    - **value**: valor da taxa de nivelamento
  - **maxValue**: valor máximo a ser transacionado.
  - **minValue**: valor mínimo a ser transacionado.
  - **offer**: `Òbject` moeda nacional oferecido na cotação
    - **divisor**: divisor usado para formatar o `value`
    - **value**: valor final em moeda nacional
  - **price**: `Object` preço de venda da cotação
    - **withoutTax**: `Object` preço de venda da moeda pedida na cotação em BRL sem impostos
      - **divisor**: divisor usado para formatar o `value`
      - **value**: valor final do preço
    - **withTax**: `Object` preço de venda da moeda pedida na cotação em BRL com impostos
      - **divisor**: divisor usado para formatar o `value`
      - **value**: valor final do preço
  - **spreadPercentage**: valor em % do spread total incluso no valor da moeda. Spread total contempla spread base + spread do correspondent (caso exista)

- **tax**: `Object` contendo informações sobre os impostos inclusos na moeda
  - **bankFee**: `Object` tarifa bancária da cotação
    - **valueUSD**: valor total da tarifa em USD
    - **total**: `Object` valor total da tarifa bancária
      - **divisor**: divisor usado para formatar o `value`
      - **value**: valor total
  - **iof**: `Object` IOF sobre o valor total da cotação
    - **percentage**: valor total do IOF sobre o valor final da transação
    - **total**: `Object` valor total do IOF sobre o valor final da transação
      - **divisor**: divisor usado para formatar o `value`
      - **value**: valor total do IOF sobre

- **total**: `Object` contendo o valor total da transação em moeda nacional
  - **withoutTax**: `Object` valor total da transação em moeda nacional, acrescido de spread, **sem** os impostos (IOF)
    - **divisor**: divisor usado para formatar o `value`
    - **value**: valor total
  - **withtax**: `Object` valor total da transação em moeda nacional, acrescido de spread e impostos
    - **divisor**: divisor usado para formatar o `value`
    - **value**: valor total

- **type**: tipo da remessa `INBOUND` ou `OUTBOUND`

- **clamp**: método de arredondamento do valor pedido na cotação, `NORMAL`, `MINIMUM` e `MAXIMUM`

- **timeToLive**: tempo máximo de expiração da cotação

- **expiresAt**: data de expiração da cotação

- **createdAt**: data de criação da cotação

​
### Quando chamar esse endpoint de *EXCHANGES*?
​
No simulador customizado do correspondente, o usuário provavelmente poderá escolher o valor a ser enviado na remessa através de um `<input>`. Você pode optar por chamar o endpoint de EXCHANGES toda vez que o usuário digitar algum valor no input, assim você terá dados atualizados a todo momento. Porém, para não sobrecarregar a API e manter a boa usabilidade de usuário no frontend, recomendamos usar uma função `debounce` para limitar o número de requisições. É possível esperar o usuário digitar todo o valor que ele deseja enviar na remessa, para então fazer apenas 1 requisição à API. Dessa forma, usamos o endpoint de maneira mais inteligente.
​
## Integração
​
Quando o usuário decidir efetuar a transação, a seguinte URL deve ser chamada:
​
`https://iamsimple.com.br/${correspondent_identifier}/app/checkout/remittance`
​
`correspondent_identifier` (_String_) é o nome do seu correspondente.
​
> Exemplo: *frente*

​
Também devem ser enviados os seguintes parâmetros na URL:
​
- **purposeCode**	(_String_): Indica a natureza da operação de remessa. São 2 valores possíveis:
  - IR001: remessa será realizada para a própria pessoa
  - IR002: remessa será enviada para outro beneficiário
​
- **currencyCode** (_String_): Código da moeda que o usuário deseja comprar
  - Alguns exemplos são USD, EUR, GBP.
​
- **remittanceAmountBRL** (_Integer_): Valor em reais a ser enviado na remessa
  - Exemplo: `1500`
​
- **remittanceAmount** (_Integer_): Valor em moeda estrangeira a ser enviada
  - Exemplo: `344.15`

- **remittanceType** (String): Tipo da remessa a ser envianda
  - Exemplo: `inbound` ou `outbound`
​

Um exemplo de URL completa seria:

```
// Para outbound
https://iamsimple.com.br/frente/app/checkout/remittance?purposeCode=IR001&remittanceAmountBRL=1500&currencyCode=USD&remittanceAmount=344.15&remittanceType=outbound
// Para inbound
https://iamsimple.com.br/frente/app/checkout/remittance?purposeCode=IR001&remittanceAmountBRL=1500&currencyCode=USD&remittanceAmount=344.15&remittanceType=inbound
```
​
Dessa forma o usuário irá entrar na SIMPLE do correspondente, com os valores já definidos no simulador.
