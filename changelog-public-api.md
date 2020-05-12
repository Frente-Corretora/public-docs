# Changelog

Este documento tem o objetivo de manter um histórico de todas as alterações feitas na API pública do Simple, como novas rotas, novas entidades e rotas depreciadas.

## [Unreleased]

## 2020-05-07 - Gerenciamento de endereços

O controle de endereços foi alterado para ter mutations isoladas para cada ação possível em um endereço, dessa forma temos um gerenciamento mais simples dos endereços de um cliente.

Todos os endereços possuem um `id` único dentro do array de endereços de um cliente e o mesmo deve ser utilizado para gerenciamento dos endereços.

A partir de agora, todas as alterações de endereço devem ser feitas através de mutations exclusivas para esta finalidade. Alterações de endereço via mutations de `physicalPerson` não devem ser feitas, pois esta funcionalidade deixará de ser suportada em breve.

Utilize as novas mutations de `createNewAddress`, `updateAddress` e `deleteAddress`.

Também foi adicionada a query para obter os dados de um endereço específico, para isso utilize `getAddress`.

Saiba mais acessando a documentação da API em (https://frentecorretora.com.br#)
