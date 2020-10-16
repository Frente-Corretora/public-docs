# Changelog

Este documento tem o objetivo de manter um histórico de todas as alterações feitas na API pública do Simple, como novas rotas, novas entidades e rotas depreciadas.

## [Unreleased]

## 2020-10-16 - Novas moedas para Remessas de Outbound

Remessas de Outbound ganha suporte a 3 novas moedas:
  - Libra Esterlina (GBP);
  - Dólar Canadense (CAD);
  - Dólar Australiano (AUD).

A criação de beneficiário e conta ganha um novo campo `currency`, que aceita valores no padrão `ISO 4217 alpha-3 format`. Para usar as novas moedas é necessário a inclusão desse campo, pois a validação do código da conta será baseada na currency.

Para quem estava integrando e optar por continuar usando apenas Euro (EUR) e Dólar Americano (USD), **nada muda**. A validação do código da conta continuará baseando-se no campo `bankCountry`, logo, não há breaking changes. Confira as mudanças de forma detalhada em [Simple API](https://docs.api.frentecorretora.com.br/?version=latest).

## 2020-06-17 - Remoção de indicadores no profile

Removido do schema de PhysicalPerson e das mutations `registerAPIClient` e `updateAPIClient`, os atributos referentes aos indicadores, `partnerName` e `partnerEmail`.

## 2020-05-07 - Gerenciamento de endereços

O controle de endereços foi alterado para ter mutations isoladas para cada ação possível em um endereço, dessa forma temos um gerenciamento mais simples dos endereços de um cliente.

Todos os endereços possuem um `id` único dentro do array de endereços de um cliente e o mesmo deve ser utilizado para gerenciamento dos endereços.

A partir de agora, todas as alterações de endereço devem ser feitas através de mutations exclusivas para esta finalidade. Alterações de endereço via mutations de `physicalPerson` não devem ser feitas, pois esta funcionalidade deixará de ser suportada em breve.

Utilize as novas mutations de `createNewAddress`, `updateAddress` e `deleteAddress`.

Também foi adicionada a query para obter os dados de um endereço específico, para isso utilize `getAddress`.

Saiba mais acessando a documentação da API em (https://frentecorretora.com.br#)
