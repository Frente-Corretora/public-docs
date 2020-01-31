# Introdução
​
## Sobre essa documentação
​
O objetivo desta documentação é tornar a integração entre os correspondentes e a plataforma SIMPLE o mais acessível e transparente possível, sanando quaisquer dúvidas **de envio de remessa**(EDIT) que a equipe técnica venha a ter à nível de código.
​
## O que é o SIMPLE
​
SIMPLE é uma plataforma _white label_ que proporciona soluções completas de câmbio para os correspondentes. As principais funcionalidades da SIMPLE são:
​
- Compra de papel moeda com delivery
- Remessas internacionais usando blockchain
​
### Realizando 'Envio de Remessa' pelo Simple
​
Atualmente, o Simple realiza o envio de remessa, podendo ser com duas naturezas de operação : para o próprio cliente e para o beneficiário.
O envio de Remessa tratasse do envio de moeda nacional para estrangeira em conta no exterior.
​
### Realizando 'Recebimento de Remessa' pelo Simple
O recebimento de remessas trata-se da entrega de moeda nacional, vinda de moeda estrangeira em conta no exterior.
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
```
{
  id	98e69bc5e28f0a29f24e33970e511daca6e75031
  createdAt	2020-01-24T18:38:52.647Z
  expiresAt	2020-01-24T18:48:52.000Z
  purposeCode	IR001
  maxExpireTime	600
  currency: {
    currencyCode	USD
    currencyName	Dólar Americano
    countryFlagUrl	https://s3.amazonaws.com/frente-exchanges/flags/united-states.svg
    totalSpreadValue	2.5
    minValue	50
  }
  currencyCode	USD
  currencyName	Dólar Americano
  countryFlagUrl	https://s3.amazonaws.com/frente-exchanges/flags/united-states.svg
  totalSpreadValue	2.5
  minValue	50
  offer	{
    value	3439894
    divisor	10000
  }
  tax	{
    iof {
      percentage	1.1
      total	{
        value	163049
        divisor	10000
      }
    }
  }
  price	{
    withTax	{
      value	43606
      divisor	10000
    }
    withoutTax {
      value	43132
      divisor	10000
    }
  }
  total	{
    withTax	{
      value	15000000
      divisor	10000
    }
    withoutTax {
      value	14836951
      divisor	10000
    }
  }
}
```
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
