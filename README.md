# Desafios criar uma aplicação de microsserviços para simular um sistema simples de compra usando filas com Rabbitmq e nestjs.

O objetivo desse mini projeto é praticar algumas técnicas para criar aplicações com alta confiabilidade, alta disponibilidade e principalmente tolerância a falhas.

## Front-end

- Desenvolva uma aplicação que liste alguns produtos com imagem, nome, descrição, preço e um botão de adicionar ao carrinho.
- Quando o usuário clicar em adicionar ao carrinho, adicione os produtos no seu carrinho de compras.
- Mostre um ícone de "carrinho de compras" no canto superior direito da tela juntamente com a quantidade de itens já adicionados, quando o usuário clicar nesse ícone redirecione ele para uma tela de listagem dos produtos.

- Crie a página de listagem de produtos onde liste os produtos adicionados e seus respectivos preços, também adicione um botão de finalizar compra.

- Quando o usuário clicar em finalizar compra redirecione ele para uma página onde insira os detalhes do seu endereço e cartão de crédito (Nome no cartão, número, data de validade no formato MM/YY e cvv). Para o endereço busque os dados na api do via cep os dados obrigatórios são cep, bairro, cidade, rua e número, país é Brasil.

- Mostre o ícone da bandeira do cartão de acordo com o número de cartão digitado, as bandeiras aceitas de cartão são: visa, mastercard e elo. Por exemplo, quando o usuário digitar um cartão mastercard:

![Example 1](https://user-images.githubusercontent.com/131708063/234089891-88cc3c53-f53a-47a4-bcd7-2581bc1daea3.png)

- Quando o usuário clicar em finalizar a compra envie os dados como um POST para o endpoint http:://localhost:8080/orders, um exemplo de json é mostrado abaixo.

```
  "items": [
        {
            "amount": 2990,
            "description": "Notebook tal",
            "quantity": 1
        }
    ],
    "customer": {
        "name": "Fulano de tal",
        "email": "fulanodetal@gmail.com"
    },
    "address": {
      "line_1": "Rual tal",
      "zip_code": "97650000",
      "city": "Curitiba",
      "state": "PR",
      "country": "Brasil"
    },
    "payment": {
            "payment_method": "credit_card",
            "credit_card": {
                    "number": "400056745400010",
                    "holder_name": "Fulano de tal",
                    "exp_month": 1,
                    "exp_year": 30,
                    "cvv": "3531",
            }
        }

```

- Após a resposta ok da api (status 200) mostre a seguinte mensagem para o usuário "Seu pedido foi realizado com sucesso. Estamos processando seu pagamento, isso pode demorar um pouco te informaremos assim que a compra for aprovada.".

- Depois do pedido concluído, redirecione o usuário para uma tela onde ele veja uma timeline do seu pedido, semelhante à imagem abaixo.

![2](https://user-images.githubusercontent.com/131708063/234095292-ce68266a-e9ba-4749-af12-671d591cc476.png)

- Simule os três status "waiting", "authorized" e "not_authorized", se o status do pedido estiver aguardando mostre o primeiro item da timeline "verde" e os restantes na cor "cinza" a medida que os status mudam avance a timeline. Por exemplo, se o status mudar para "authorized" coloque o item de pagamento aprovado da timeline na cor verde e assim sucessivamente.

- Para simplificar simule uma resposta da api com um mock, por exemplo:
  http:://localhost:8080/orders/1

```
{
    id: 1,
    status: "waiting"
}
```

## Api

- Crie uma aplicação com nestjs na pasta server.
- Adicione e configure o Rabbitmq para as filas serem duráveis e as confirmações de mensagens manuais.
- Crie duas filas, uma para os pedidos e outra para envio de emails. Para a fila de pedidos pode ser algo do tipo 'payment-orders' e para os emails 'payment-emails'.
- Crie um endpoint (POST) que receba pedidos de compra em http:://localhost:8080/orders
- Quando chegar uma requisição nesse endpoint adicione o pedido na tabela pedidos no banco de dados usando prisma e publique uma mensagem na fila com os dados do json enviado pelo front-end.

- Precisamos receber a resposta do Gateway de pagamento, para isso vamos utilizar um webhook. Crie um endpoint na aplicação (POST) em http:://localhost:8080/webhook/orders.

- Quando uma requisição chegar nesse endpoint atualize os dados do pedido no banco de dados de acordo com a resposta do servidor. Também publique uma mensagem na fila de emails.

  ### Consumidor

  - Crie outra aplicação de microsserviços em nestjs na pasta consumer para ser o consumidor das mensagens adicionadas na fila. Quando um evento de trabalho chegar envie os dados para o endpoint do Gateway de pagamento em http:://localhost:8081/orders, após isso envie uma confirmação da mensagem ao rabbitmq.
  - Quando um evento de enviar email chegar, envie um e-mail para o usuário informando o status da transação. Pode-se utilizar o nodemailer com o email frontend.jobii.101@gmail.com e senha 'whwjodupopiwrwwx'.

## Simular a resposta do gateway de pagamento

- Vamos simular os eventos de pagamento que precisamos, para isso vamos utilizar algo extremamente simplificado usando um servidor nodejs com express e a lib amqplib.

- Crie um servidor usando express adicione configure a lib amqplib para que o envio das mensagens aos consumidores sejam atrasadas por um tempo. Além disso, para as mensagens da fila serem duráveis e confirmações manuais.

- Crie outro script chamado consumer.js que vai agrupar a lógica do consumidor de mensagens da fila.

- A fila pode se chamar 'orders' ou outro nome de sua escolha.

- Quando um evento de compra chegar, publique uma mensagem na fila com um atraso de 10 minutos, ou seja, a mensagem vai demorar 10 minutos para ser processada pelo consumidor.

- Quando o consumidor receber uma ordem de pagamento, faça um post para o endpoint do webhook da aplicação em http:://localhost:8080/webhook/orders. Você pode simular o status da transação sorteando um número aleatório entre 0 e 1 em que 0 significa not_authorized e 1 authorized, ou enviar sempre o status authorized fica a sua escolha. Após o envio desse post confirme a mensagem ao rabbitmq.
