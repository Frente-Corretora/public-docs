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
**Não é necessário fazer nenhum cálculo no frontend**, já que o endpoint de _EXCHANGES_ é o responsável por trazer todas as informações relacionadas à moedas, taxas, cotações, valores em real e valores em moeda estrangeira.
​
O endpoint base é:
​
`https://api.frentecorretora.com.br/v1/exchanges/remittance`
​
A API da SIMPLE suporta 2 tipos de cálculos para remessas:
​
- de real (BRL) para moeda estrangeira
- de moeda estrangeira para real (BRL)
​
Quando o cálculo é feito de real para moeda estrangeira, usamos a rota `/reverse`.
​
 BRL -> USD/EUR
> `https://api.frentecorretora.com.br/v1/exchanges/remittance/reverse`
​
Quando o cálculo é feito de moeda estrangeira para real, a rota é `/`.
​
USD/EUR -> BRL
> `https://api.frentecorretora.com.br/v1/exchanges/remittance/`
​
Essa distinção entre rotas acontece pois as taxas de spread podem variar, dependendo do tipo de cálculo usado.
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
Um exemplo de endpoint completo seria:
​
Para conversões de BRL (real) para moeda estrangeira:
​
`https://api.frentecorretora.com.br/v1/exchanges/remittance/reverse?purposeCode=IR001&currency=USD&correspondentId=1&value=15000000`
​
Para conversões de moeda estrangeira para BRL (real):
​
`https://api.frentecorretora.com.br/v1/exchanges/remittance?purposeCode=IR001&currency=USD&correspondentId=1&value=15000000`
​
Um exemplo de payload de resposta seria:
​
```js
{
  id: "98e69bc5e28f0a29f24e33970e511daca6e75031",
  createdAt: "2020-01-24T18:38:52.647Z",
  expiresAt: "2020-01-24T18:48:52.000Z",
  purposeCode: "IR001",
  maxExpireTime: 600,
  currency: {
    currencyCode: "USD",
    currencyName: "Dólar Americano",
    countryFlagUrl: "https://s3.amazonaws.com/frente-exchanges/flags/united-states.svg",
    totalSpreadValue: 2.5,
    minValue: 50,
  },
  offer: {
    value: 3439894,
    divisor: 10000,
  },
  tax: {
    iof: {
      percentage: 1.1,
      total: {
        value: 163049,
        divisor: 10000,
      },
    },
  },
  price: {
    withTax: {
      value: 43606,
      divisor: 10000,
    },
    withoutTax: {
      value: 43132,
      divisor: 10000,
    },
  },
  total: {
    withTax: {
      value: 15000000,
      divisor: 10000,
    },
    withoutTax: {
      value: 14836951,
      divisor: 10000,
    },
  },
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

- id: ID da remessa

- createdAt: data de criação da remessa

- expiresAt: data de expiração da remessa

- purposeCode: natureza da operação
  - IR001: remessa para o próprio cliente
  - IR002: remessa para um beneficiário terceiro

- maxExpireTime: tempo máximo de expiração da remessa

- currency: `Object` contendo informações sobre a moeda
  - currencyCode: sigla da moeda escolhida pelo usuário
  - currencyName: nome da moeda
  - countryFlagUrl: url para buscar a bandeira da moeda escolhida, no formato svg
  - totalSpreadValue: valor em % do spread total incluso no valor da moeda. Spread total contempla spread base + spread do correspondente
  - minValue: valor mínimo a ser transacionado.

- offer: `Object` contendo o valor total em moeda estrangeira que o usuário deseja transacionar.
  - value: valor total em moeda estrangeira, não formatado
  - divisor: divisor usado para formatar o `value`

- tax: `Object` contendo informações sobre os impostos inclusos na moeda
  - iof.percentage: % do IOF sobre o valor total da transação
  - iof.total.value: valor total do IOF sobre o valor final da transação
  - iof.total.divisor: divisor usado para formatar o `value`

- price: `Object` contendo informações sobre o valor da moeda no momento da simulação
  - withTax: é o VET, ou seja, valor base da cotação da moeda, acrescido de spread e impostos
  - withoutTax: valor base da cotação acrescido do spread, **sem** os impostos (IOF)

- total: `Object` contendo o valor total da transação em moeda nacional
  - withtax: valor total da transação em moeda nacional, acrescido de spread e impostos
  - withoutTax: valor total da transação em moeda nacional, acrescido de spread, **sem** os impostos (IOF)

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
​
​
Um exemplo de URL completa seria:
​
`https://iamsimple.com.br/frente/app/checkout/remittance?purposeCode=IR001&remittanceAmountBRL=1500&currencyCode=USD&remittanceAmount=344.15`
​
Dessa forma o usuário irá entrar na SIMPLE do correspondente, com os valores já definidos no simulador.
