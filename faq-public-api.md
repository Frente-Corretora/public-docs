# FAQ API Pública

## Credenciais do ambiente de produção

Antes da criação de credenciais para o ambiente de produção, é necessário cumprir um checklist na aplicação criada.

Todas as interações com a API devem ser feitas usando a UI da aplicação criada, simulando a interação de usuário.

Colete evidências das interações do checklist (prints, gifs ou vídeos).

Envie um email para dev@frentecorretora.com.br com o assunto **ACESSO API DE PRODUÇÃO - {NOME_PARCEIRO}**, contendo todas as evidências e outras informações necessárias. Informe o envio do email no channel de integração do slack.

### 1. Cadastrar cliente

Deve ser cadastrado um cliente, juntamente com:

- 1 endereço
- 1 conta
- 1 beneficiário
- 1 documento válido.

O status desse cliente deve estar como *accepted* (aprovado) no painel de adminstração do Simple para passar nas validações.

Enviar o **CPF** do cliente para validações.

### 2. Criar operação de papel moeda

> Somente se a aplicação interagir com essa API

Criar uma operação de papel moeda, com loja e valor a sua escolha.

A moeda da operação deve USD ou EUR.

O status da operação deve ser *waiting_payment* (aguardando pagamento) no painel de adminstração do Simple para passar nas validações.

Enviar o **ID** da operação para validações.

### 3. Criar operação de remessa

> Somente se a aplicação interagir com essa API

Criar duas operações de remessa, uma para disponibilidade no exterior e outra para manutenção de residente com moeda, valor e país de destino a sua escolha.

O status da remessa deve ser *processing* (processando) no painel de adminstração do Simple para passar nas validações.

Enviar o **ID** das operações para validações.

### 4. Termos de uso

Uma evidência de onde os termos de uso estão sendo apresentados para o usuário e que ele precisa confirmar a existência deles antes de interagir com a API, portanto esse aceite deve ocorrer antes de conseguir criar uma operação.
