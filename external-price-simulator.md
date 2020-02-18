# Introdução - V2

<aside class="warning">
  Se você está procurando a documentação antiga da integração do simulador [clique aqui](https://github.com/Frente-Corretora/public-docs/blob/d0c941d1923bd23723f0df932756cacb106a241b/external-price-simulator.md). Essa documentação foi atualizada.
</aside>

SIMPLE é uma plataforma _white label_ que proporciona soluções completas de câmbio para os correspondentes. As principais funcionalidades da SIMPLE são:
​
- Compra de papel moeda com delivery
- Remessas internacionais usando blockchain

## Sobre essa documentação
​
O objetivo desta documentação é tornar a integração entre os correspondentes e a plataforma SIMPLE o mais acessível e transparente possível, sanando quaisquer dúvidas que a equipe técnica venha a ter à nível de código.

# Acessando as documentações

Para acessar a documentação de integração de um simulador externo com a funcionalidade de **Envio de remessas** do SIMPLE, clique [aqui](https://github.com/Frente-Corretora/public-docs/blob/master/external-remittance-simulator.md) ou acesse esse link:

https://github.com/Frente-Corretora/public-docs/blob/master/external-remittance-simulator.md

A seguir você tem acesso à documentação para integrar um simulador externo com o Simple, na funcionalidade de **Compra de papel moeda**.

# Ferramenta de documentação

Usamos por padrão para documentar nossa API um serviço chamado de [Postman](https://www.postman.com/). Entre as várias funcionalidades e a interface amigável, objetivamos sempre ajudar sua equipe de desenvolvimento, que pode simplesmente mandar executar toda a nossa coleção da API na máquina deles. 

# Documentação de como integrar um Simple com dados vindos de um simulador externo

Temos três entidades em questão:

 - CORRESPONDENT
 - MERCHANTS
 - EXCHANGES

 **CORRESPONDENT** é resposável por retornar os dados do correspondente em questão recebendo um identificador unico na request. Ex: *matriz*

 **MERCHANTS** pertence a um CORRESPONDENT e pode ser visualizado como a cidade de venda moeda, ele contem todos os dados sobre a cidade escolhida pelo cliente e tambèm possui um identicador unico. Ex: *WL-FRENTE-SP*

 **EXCHANGES** é resposável por retornar as taxas das moedas dispíveis da cidade escolhida pelo cliente, baseado na no código AGENT

 ____

## Como obter os MERCHANTS

O endpoint utilizado, passando o `correspondent_id` do corresondente em questão.

`https://api.frentecorretora.com.br/v1/correspondents/{correspondent_id}/merchants`

Toda a documentação tanto de como buscar quanto como qual a resposta está descrita [nesse link do postman](https://docs.api.frentecorretora.com.br/?version=latest#e77c8823-a960-406e-9d9f-8ba9f6eb5770)

Nesse caso, vamos utilizar somente `id` e `label`

Com esses dados conseguimos exibir um select para o cliente selecionar a cidade e a partir disso obtemos as taxas dessa cidade

___

## Como obter os EXCHANGES

O endpoint utilizado, passando o `id` do merchant em questão.

`https://api.frentecorretora.com.br/v1/exchanges/products/{id}`

Toda a documentação tanto de como buscar quanto como qual a resposta está descrita [nesse link do postman](https://docs.api.frentecorretora.com.br/?version=latest#6d6e0109-d519-4258-8bbd-2e04af44f052)

Nesse caso, vamos utilizar somente `countryFlagUrl`, `currencyName`, `currencyCode`

Com esses dados conseguimos exibir um select para o cliente selecionar a moeda e a partir disso calculamos a conversão.
____

## A Cotação

Com as informações de moeda e merchant em mãos você pode requisitar a nossa API um valor (tanto em real, quanto em moeda estrangeira) e ela garantirá para você todas as informações necessárias para exibir para o seu cliente!

Toda a documentação tanto de como buscar quanto como qual a resposta está descrita [nesse link do postman](https://docs.api.frentecorretora.com.br/?version=latest#4ddba80d-73c9-4829-a494-18848bf81ee7).

Com a flag `reverse` você pode indicar o se o valor passado é real ou moeda estrangeira.

Com a flag `rounding` você consegue entender se o seu valor foi arredondado (nossa API arredonda para garantir que tenhamos múltiplos para a transferência para o valor mais próximo), se ele foi arredondado para cima por estar abaixo do mínimo ou arredondado para baixo por estar acima do máximo. **Trabalhe essa resposta para exibir mensagens de atualização para o seu usuário**.

Com as respostas `price`[`withoutTax`] e `price`[`withTax`] você consegue extrair informações de VET e cotação.

Com as respostas `total`[`withTax`] você consegue extrair o que devolver para o seu usuário.

<aside class="warning">
  Lembre-se: Sua cotacão irá expirar em 10 minutos! O mercado de câmbio oscila! Os valores podem ser ligeiramente alterados.
</aside>

## Integração

Quando o seu usuário clicar em comprar no seu simulador você deve montar a URL e redirecionar para ela ou abrir em uma nova guia conforme os valores informados.

`correspondent_identifier` = Identifier do seu Simple, exemplo: *matriz*

`agentId` = id da merchant escolhida pelo usuario

`productId` = id da moeda escolhida pelo usuario

`productAmount` = quantidade da moeda escolhida pelo usuario

`utm_source` = usado para tracking no google analitics

A url ira ficar dessa forma, usando o exemplos acima

`https://iamsimple.com.br/{correspondent_identifier}/app/checkout/paper-money?agentId=WL-FRENTE-SP&productId=USD&productAmount=1000&utm_source=matriz-simulator`
