Buscapé Marketplace

# API Pedidos - V2 

### 1 - Fluxo de informações 

Principais interações que podem ser realizadas entre parceiros (consumidores) e a API Pedidos Buscapé.

![Fluxo de interações](/images/Fluxo_Buscape_Marketplace.png)

Obs.: Os e-mails transacionais que reportam o status do pedido para os clientes devem ser desabilitados. Esse tipo de e-mail já é enviado pelo Buscapé Marketplace.

### 2 - Consulta pedido

A consulta de pedido retorna o pedido no momento atual. O pedido pode está nos seguintes status:

| Status de pedido | Descrição | Quem Envia |
| -----------------| ----------| -----------|
| new | Pedido novo no Buscapé Marketplace | Buscapé |
| accept | Pedido aceito pelo parceiro | Seller |
| not_accept | Pedido recusado pelo parceiro | Seller |
| pending | Aguardando meio de pagamento | Buscapé |
| approved | Pagamento aprovado | Buscapé |
| not_approved | Pagamento não aprovado | Buscapé |
| cancelled | Pedido cancelado | Buscapé |
| invoiced | Pedido Faturado | Seller |
| in_hosting | Item na transportadora | Seller |
| in_route | Item em rota de entrega | Buscapé |
| retrying | Retentativas | Buscapé |
| reversal | Devolução | Buscapé |
| delivered | Pedido totalmente entregue | Buscapé |

#### 2.1 - Consulta pedido por Status

O Buscapé Marketplace disponibiliza o serviço para recuperar uma lista de pedidos por status que estão relacionados com o token do parceiro informado. Este serviço irá retornar um ou mais pedidos conforme o status.

- HTTP Request Method: GET

- HTTP Request URL: [http://api.buscape.com.br/orders/v2/status/{STATUS}?limit={NUMBER}&offset={NUMBER}&lastUpdate={DATE}](http://api.buscape.com.br/orders/v2/status)

- Body Response: Array [Orders](#IntegraçãoPedidos-V2-Orders)

#### 2.2 - Consulta pedido por ID Buscapé

O Buscapé Marketplace disponibiliza o serviço para recuperar detalhes do pedido informado

- HTTP Request Method: GET

- HTTP Request URL Pattern: [http://api.buscape.com.br/orders/v2](http://api.buscape.com.br/orders/v2)/{ID_Buscape}

- Body Response: [Orders](#IntegraçãoPedidos-V2-Orders)

### 3 - Consulta de estoque

O Buscapé Marketplace realiza a consulta de estoque, que consiste em perguntar ao parceiro (sistematicamente), se existe estoque para os itens do pedido.

Caso o parceiro confirme a disponibilidade de estoque, o Buscapé irá reduzir internamente o estoque dos itens associados ao pedido, conforme a quantidade de cada item, e o pedido continuará o fluxo normal; caso contrário o pedido será cancelado. Para isto o parceiro deverá implementar um serviço de resposta para a consulta de estoque.

Para isto o parceiro deve implementar um Webservice REST que use formato JSON. Este serviço será responsável por receber as requisições de consulta de estoque vindos do Buscapé, e o parceiro deverá responder se tem estoque ou não para os itens do pedido. Caso o parceiro confirme a disponibilidade de estoque, o Buscapé irá reduzir internamente o estoque s itens associados ao pedido, conforme a quantidade de cada item.

*A quantidade a ser retornada será o total em estoque menos o total consultado, mesmo que a subtração seja menor que 0. Ex: 10(Estoque) – 12(Consulta) = -2


#### 3.1 - Requisição REST:

A requisição que o parceiro irá receber por método POST terá o seguinte formato:

- HTTP Method: POST

- JSON Request: [Stock](#621---stock-request)

#### 3.2 - Resposta REST:

A requisição que o parceiro irá enviar por método POST terá o seguinte formato:

- HTTP Method: POST

- JSON Request: [Stock](#621---stock-response)

### 4 - Notificação de pedido

O Buscapé Marketplace disponibiliza a funcionalidade de Notificação de Eventos: quando um determinado Evento ocorrer (por exemplo um pedido com pagamento aprovado), iremos notificar a sua Aplicação

Para integrar seu sistema e aproveitar ao máximo os recursos de notificações, é necessário que sua aplicação esteja preparada para receber um POST de um JSON no formato:

- HTTP Method: POST

- JSON Request: [Notification](#63---notification)
```json
{
   "eventDate":"YYYY-MM-DD",
   "sellerId":"xxxxxxx",
   "orderUri":"http:\/\/api.buscape.com.br\/orders\/152xxxxxxxx",
   "order":{
    "orderStatus": "approved",
    "orderedItems": [
        {
            "sku": null,
            "skuSellerId": "12345678",
            "quantity": 1,
            "price": 99.99,
            "discount": 0
        }
    ],
    "paymentMethods": [
        {
            "method": "CARTAO",
            "amount": 99.99,
            "installments": 0,
            "paymentDueAt": "YYYY-MM-DDThh:mm:ss.000Z",
            "approvedAt": null
        }
    ],
    "sellerOrder": "152xxxxxxxx",
    "totalFreight": 0,
    "purchaseAt": "YYYY-MM-DDThh:mm:ss.000Z",
    "lastUpdateAt": "YYYY-MM-DDThh:mm:ss.000Z",
    "clientProfileData": {
        "email": "teste@teste.com",
        "firstName": "Primeiro Nome",
        "lastName": "Ultimo Nome",
        "documentType": "CPF",
        "document": "12345678900",
        "phone": "(11)22223344",
        "corporate": {
            "corporateName": null,
            "tradeName": "",
            "stateInscription": "",
            "corporatePhone": null
        }
    },
    "shippingInfo": [
        {
            "address": {
                "addressType": "RESIDENCIAL OU COMERCIAL",
                "receiverName": "Receptor da encomenda",
                "postalCode": "cep",
                "city": "cidade",
                "state": "estatdo",
                "country": "pais",
                "street": "nome da rua",
                "number": "numero",
                "neighborhood": "bairro",
                "complement": "complemento",
                "reference": "referencia"
            },
            "deliveries": [
                {
                    "item": {
                        "sku": "sku",
                        "skuSellerId": "12345678",
                        "quantity": null
                    },
                    "selectedSla": "",
                    "otd": {
                        "shippingEstimate": 5,
                        "transitTime": 0,
                        "crossDockingTime": 0,
                        "scheduledAt": null,
                        "scheduledPeriod": null
                    },
                    "trackingNumber": null,
                    "sellerDeliveryId": "123456789",
                    "freightPrice": 0,
                    "freightCost": 0,
                    "additionalInfo": "",
                    "carrier": {
                        "name": "Tipo de envio",
                        "cnpj": ""
                    },
                    "invoice": {
                        "number": 0,
                        "value": 0,
                        "url": "",
                        "issuanceDate": null,
                        "invoiceKey": null
                    },
                    "tracking": {
                        "description": null,
                        "occurredAt": null,
                        "controlPoint": "accept"
                    },
                    "giftWrap": {
                        "available": false,
                        "value": 0,
                        "giftCard": {
                            "from": "",
                            "to": "",
                            "message": ""
                        }
                    },
                    "lockTTL": "2d"
                }
            ]
        }
    ],
    "orderID": "152xxxxxxxx"
   }
}
```


#### 4.1 - URL de Callback

É o endereço que será chamado pelo Buscapé Marketplace para notificar o parceiro que ocorreu algo novo no sistema. Se resume ao endereço de um serviço rest que a aplicação desenvolvida pelo parceiro saberá interpretar. Todas as chamadas deverão ter suporte ao método POST e deverão retornar o status 200 ou 201 em caso de sucesso na requisição.

#### 4.2 - Aceite

Ao receber um novo pedido, o parceiro deverá, antes de tudo, validá-lo. Estando ele de acordo ele envia uma notificação de Aceite para o Buscapé Marketplace.

**A partir do momento que o parceiro recebe o pedido, ele deve reservar o estoque até o pedido ser de fato cancelado, mesmo que envie a recusa.**

É necessário que sua aplicação esteja preparada para enviar um POST de um JSON no formato:

- HTTP Method: POST

- HTTP Request URL Pattern: [http://api.buscape.com.br/orders/v2/{ID_Buscape}/acceptance](http://api.buscape.com.br/orders/v2)

- JSON Request: [Acceptance](#64---acceptance)

Caso o Pedido contenha informações divergentes ao esperado, o seller deve enviar uma notificação de “Não Aceito” e especificar o motivo no campo “message”.

Obs.: Quando o pedido não for aceito isto não significa que ele será cancelado. Nossa central de atendimento entrará em contato ou com o seller ou com o cliente para tratar dos novos termos para prosseguir ou não com a venda. 

#### 4.3 - Situações de Exceção

Ocorrerão até 5 tentativas de notificação, em intervalos regulares. Se em nenhuma delas o sistema do Buscapé receber código esperado, a notificação será armazenada como não entregue e será cancelada. Todos os históricos das chamadas são armazenados por até 60 dias e poderão ser disponibilizados caso solicitado.

É recomendado que o sistema do parceiro consulte os serviços em intervalos regulares, não inferiores á 30 minutos, para checar se existe algum novo recurso disponível, pedido realizado, pagamento aprovado, etc. Por convenção, as notificações não possuem todos os detalhes de uma transação, necessitando assim a chamada aos respectivos serviços relacionados.

### 5 - Atualiza o tracking do pedido

O Buscapé Marketplace disponibiliza o serviço para registrar uma nova operação de tracking de Entrega para os itens do pedido

- HTTP Request Method: POST

- HTTP Request URL Pattern: [http://api.buscape.com.br/orders/v2/{ID_Buscape}/tracking](http://api.buscape.com.br/orders/v2)

- JSON Request: [Tracking](#65---tracking)

#### 5.1 - Status do tracking

O status do tracking é definido no atributo **tracking.controlPoint** através dos seguintes status:

| Status de Tracking | Descrição |
| ------------------ | --------- |
| invoiced | Pedido Faturado |
| in_hosting | Item na transportadora |

#### 5.2 - Campos obrigatórios na atualização de Tracking

Dependendo do status do Tracking alguns itens são obrigatórios:

*   Status **invoiced**:

| **Atributo** | **Descrição** |
| ------------ | --------------|
| item | Item alterado pela operação de tracking |
| item.skuSellerId | SKU do parceiro |
| item.quantity | Quantidade do item |
| tracking | Informações de sobre o Tracking |
| tracking.controlPoint | Status do pedido |
| tracking.description | Descrição adicional sobre tracking |
| tracking.occurredAt | Data da ocorrência |
| invoice | Informações sobre a Nota Fiscal da compra |
| invoice.number | Número da Nota Fiscal |
| invoice.value | Valor da Nota Fiscal da compra |
| invoice.url | Url para consulta da DANFE |
| invoice.issuanceDate | Data de emissão da Nota Fiscal |
| invoice.invoiceKey | Número da chave de acesso à nota fiscal.  |
```json
[
  {
    "item": {
      "skuSellerId": "string",
      "quantity": "number"
    },
    "tracking": {
      "controlPoint": "string",
      "description": "string",
      "occurredAt": "datetime"
    },
    "invoice": {
      "number": "number",
      "value": "number",
      "url": "number",
      "issuanceDate": "datetime",
      "invoiceKey": "number"
    }
  }
]
```

*   Status **in_hosting**: 
    Nesse status pode enviar <span>**trackingNumber** e/ou **carrier**, além dos demais atributos.</span>

| **Atributo** | **Descrição** |
| ------------ | ------------- |
| item | Item alterado pela operação de tracking |
| item.skuSellerId | SKU do parceiro |
| trackingNumber | ID do objeto na transportadora |
| carrier | Informações sobre transportadora |
| carrier.name | Nome da transportadora |
| carrier.cnpj | CNPJ da transportadora |
| tracking | Informações de sobre o Tracking |
| tracking.controlPoint | Status do pedido |
| tracking.description | Descrição adicional sobre tracking |
| tracking.occurredAt | Data da ocorrência |

```json
[
  {
    "item": {
      "skuSellerId": "string"
    },
    "trackingNumber": "string",
    "carrier":{
	"name":"string",
	"cnpj":"string"
    },
    "tracking": {
      "controlPoint": "string",
      "description": "string",
      "occurredAt": "datetime"
      
    }
  }
]
```
### 
6 - Mensagens das requisições

#### 6.1 - Orders 

```json
{
   "orderID":"string",
   "orderStatus":"string",
   "orderedItems":[
      {
         "sku":"number",
         "skuSellerId":"string",
         "quantity":"number",
         "price":"number",
         "discount":"number"
      }
   ],
   "paymentMethods":[
      {
         "method":"string",
         "amount":"number",
         "installments":"number",
         "paymentDueAt":"datetime",
         "approvedAt":"datetime"
      }
   ],
   "sellerOrder":"string",
   "totalFreight":"number",
   "purchaseAt":"datetime",
   "lastUpdateAt":"datetime",
   "clientProfileData":{
      "email":"string",
      "firstName":"string",
      "lastName":"string",
      "documentType":"string",
      "document":"string",
      "phone":"number",
      "corporate":{
         "name":"string",
         "tradeName":"string",
         "stateInscription":"string",
         "phone":"string"
      }
   },
   "shippingInfo":[
      {
         "address":{
            "addressType":"string",
            "receiverName":"string",
            "postalCode":"string",
            "city":"string",
            "state":"string",
            "country":"string",
            "street":"string",
            "number":"string",
            "neighborhood":"string",
            "complement":"string",
            "reference":"string"
         },
         "deliveries":[
            {
               "item":{
                  "sku":"string",
                  "skuSellerId":"string"
               },
               "selectedSla":"string",
               "lockTTL":"string",
               "otd":{
                  "shippingEstimate":"number",
                  "transitTime":"number",
                  "crossDockingTime":"number",
                  "scheduledAt":"datetime",
                  "scheduledPeriod":"string"
               },
               "trackingNumber":"string",
               "sellerDeliveryId":"string",
               "freightPrice":"number",
               "freightCost":"number",
               "additionalInfo":"string",
               "carrier":{
                  "name":"string",
                  "cnpj":"string"
               },
               "tracking":{
                  "description":"string",
                  "occurredAt":"datetime",
                  "controlPoint":"string"
               },
               "invoice" : {
                  "number": "number",
                  "value": "number",
                  "url": "string",
                  "issuanceDate": "datetime",
                  "invoiceKey": "string"
               },
               "giftWrap":{
                  "available":"number",
                  "value":"number",
                  "giftCard":{
                     "from":"string",
                     "to":"string",
                     "message":"string"
                  }
               }
            }
         ]
      }
   ]
}
```


| **Atributo** | **Descrição** |
| ------------ | ------------- |
| orderID | ID Buscapé |
| orderStatus | Status do pedido |
| sellerOrder | ID pedido no parceiro |
| totalFreight | Total do frete |
| purchaseAt | Data da compra |
| lastUpdateAt | Data de atualização |
| paymentMethods | Forma de pagamento |
| paymentMethods.method | Métodos de pagamento ("BOLETO", "CARTAO")  |
| paymentMethods.amount | Total do pedido |
| paymentMethods.installments | Número de parcelas  |
| paymentMethods.paymentDueAt | Data do pagamento |
| paymentMethods.approvedAt | Data da aprovação do pagamento  |
| orderedItems | Informação sobre os itens do pedido |
| orderedItems.sku | SKU do Buscapé |
| orderedItems.skuSellerId | SKU do parceiro |
| orderedItems.quantity | Quantidade do item |
| orderedItems.price | Preço unitário do item |
| orderedItems.discount | Desconto aplicado no item |
| clientProfileData | Informação sobre o cliente |
| clientProfileData.email | E-mail do cliente |
| clientProfileData.firstName | Nome do cliente |
| clientProfileData.lastName | Sobrenome do cliente |
| clientProfileData.documentType | Tipo do documento pessoa física (CPF ou RG) |
| clientProfileData.document | Número do documento |
| clientProfileData.phone | Telefone do cliente |
| clientProfileData.corporate.name | Nome da empresa |
| clientProfileData.corporate.tradeName | Nome Fantasia |
| clientProfileData.corporate.stateInscription | Inscrição estadual  |
| clientProfileData.corporate.Phone | Telefone comercial  |
| shippingInfo | Informação da entrega |
| shippingInfo.address | Informação sobre o endereço de entrega |
| shippingInfo.address.addressType | Tipo do endereço (Residencial ou Comercial) |
| shippingInfo.address.receiverName | Nome da pessoa que irá receber o pedido |
| shippingInfo.address.postalCode | CEP do endereço |
| shippingInfo.address.city | Cidade |
| shippingInfo.address.state | Estado |
| shippingInfo.address.country | País |
| shippingInfo.address.street | Logradouro  |
| shippingInfo.address.number | Número ("S/N" quando não houver)  |
| shippingInfo.address.neighborhood | Bairro |
| shippingInfo.address.complement | Complemento |
| shippingInfo.address.reference | Ponto de referência |
| shippingInfo.deliveries | Informações de entregas |
| shippingInfo.deliveries.item | Item alterado pela operação de tracking |
| shippingInfo.deliveries.item.sku | SKU do Buscapé |
| shippingInfo.deliveries.item.skuSellerId | SKU do parceiro |
| shippingInfo.deliveries.selectedSla | SLA de entrega |
| shippingInfo.deliveries.lockTTL | Máximo de tempo que o vendedor deve deixar o estoque reservado até que tenha a confirmação do pagamento. (Exemplo: 5d = 5 dias) |
| shippingInfo.deliveries.trackingNumber | ID do objeto na transportadora |
| shippingInfo.deliveries.sellerDeliveryId | ID único da entrega para o lojista no parceiro, não pode haver repetição deste ID |
| shippingInfo.deliveries.freightPrice | Valor nominal do frete (sem descontos e abatimentos, com margem comercial) |
| shippingInfo.deliveries.freightCost | Valor cobrado pelo frete |
| shippingInfo.deliveries.additionalInfo | Informações adicionais sobre a entrega. |
| shippingInfo.deliveries.otd | On Time Delivery-Tempo de separação/expedição e da transportadora |
| shippingInfo.deliveries.otd.shippingEstimate | Estimativa de entrega. Ex: 5d = 5 dias, 5bd = 5 dias úteis |
| shippingInfo.deliveries.otd.transitTime | Prazo de transporte para entrega do produto em dias úteis |
| shippingInfo.deliveries.otd.crossDockingTime | Tempo de preparação/fabricação do produto em dias. Esse tempo é incluído no cálculo de frete. |
| shippingInfo.deliveries.otd.scheduledAt | Data de agendamento da entrega |
| shippingInfo.deliveries.otd.scheduledPeriod | Período que a entrega foi agendada ['MANHÃ', 'TARDE', 'NOITE'] |
| shippingInfo.deliveries.carrier | Informações sobre transportadora |
| shippingInfo.deliveries.carrier.name | Nome da transportadora |
| shippingInfo.deliveries.carrier.cnpj | CNPJ da transportadora |
| shippingInfo.deliveries.tracking | Informações de sobre o Tracking |
| shippingInfo.deliveries.tracking.controlPoint | Status do pedido |
| shippingInfo.deliveries.tracking.description | Descrição adicional sobre tracking |
| shippingInfo.deliveries.tracking.occurredAt | Data da ocorrência |
| shippingInfo.deliveries.invoice | Informações sobre a Nota Fiscal da compra |
| shippingInfo.deliveries.invoice.number | Número da Nota Fiscal |
| shippingInfo.deliveries.invoice.value | Valor da Nota Fiscal da compra |
| shippingInfo.deliveries.invoice.url | Url para consulta da DANFE |
| shippingInfo.deliveries.invoice.issuanceDate | Data de emissão da Nota Fiscal |
| shippingInfo.deliveries.invoice.invoiceKey | Número da chave de acesso à nota fiscal.  |
| shippingInfo.deliveries.giftWrap | Informações de embrulho para presente |
| shippingInfo.deliveries.giftWrap.available | Flag que indica se existe embrulho para presente para o produto |
| shippingInfo.deliveries.giftWrap.value | Valor cobrado pelo embrulho |
| shippingInfo.deliveries.giftWrap.giftCard | Mensagem que deverá ser enviada juntamente com o embrulho de presente |
| shippingInfo.deliveries.giftWrap.giftCard.from | Nome de quem está dando o presente |
| shippingInfo.deliveries.giftWrap.giftCard.to | Nome de quem irá receber o presente |
| shippingInfo.deliveries.giftWrap.giftCard.message | Mensagem que deverá ser enviada no cartão juntamente com o embrulho de presente |

#### 6.2 - Stock

##### 6.2.1 - Stock Request

```json
{
    "buscapeID" : "string",
    "orderedItems": [
      {
        "skuSellerId" : "string",
        "quantity" : "number",
        "postalCode" : "string"
      }
    ]
}
```


##### 6.2.1 - Stock Response


```json
[  
   {  
      "buscapeID":"number",
      "skuSellerId":"number",
      "available":"number",
      "crossDockingTime":"number",
      "message":"string"
   }
]
```



#### 6.3 - Notification

```json
{
   "eventDate":"datetime",
   "sellerId":"number",
   "orderUri":"string",
   "order":"order"
}
```


#### 6.4 - Acceptance

```json
{
    "eventDate":"datetime",
    "accepted" : "boolean",
    "sellerOrder" : "string",
    "message" : "string"
}
```

 

#### 6.5 - Tracking


```json
[
   {
      "item":{
         "sku":"string",
         "skuSellerId":"string"
      },
      "selectedSla":"string",
      "lockTTL":"string",
      "otd":{
         "shippingEstimate":"number",
         "transitTime":"number",
         "crossDockingTime":"number",
         "scheduledAt":"date",
         "scheduledPeriod":"string"
      },
      "trackingNumber":"string",
      "sellerDeliveryId":"string",
      "freightPrice":"number",
      "freightCost":"number",
      "additionalInfo":"string",
      "carrier":{
         "name":"string",
         "cnpj":"string"
      },
      "invoice":{
         "number":"number",
         "value":"number",
         "url":"string",
         "issuanceDate":"datetime",
         "invoiceKey":"string"
      },
      "tracking":{
         "description":"string",
         "occurredAt":"date",
         "controlPoint":"string"
      }
   }
]
```


### 7 - Tecnologias e Padrões

Para facilitar o consumo dos serviços pelos parceiros Marketplace, padrões foram criados para tornar todo fluxo de comunicação claro e objetivo.

- Modelo de arquitetura: REST*
- Protocolo: HTTP 1.1
- Charset: UTF-8
- Content-type: JSON**
- Formatação de data/hora: yyyy-MM-ddThh:mm:ss.sssTZD[ ( Exemplo: 2013-06-28T08:54:00.000Z )](http://mmss.stzd/)
     * YYYY = quatro dígitos - ano
     * MM   = dois dígitos - mês (01=Janeiro, etc.)
     * DD   = dois dígitos - dia do mês
     * hh   = dois dígitos horas (00 até 23)
     * mm   = dois dígitos - minutos (00 até 59)
     * ss   = dois dígitos - segundos (00 até 59)
     * sss  = 3 dígitos - decimais de segundo
     * TZD  = timezone (Z or +hh:mm ou -hh:mm)

* REST é um modelo arquitetural que foca na simplicidade e que não estipula um formato rígido, proporcionando flexibilidade no desenvolvimento e uso dos serviços.

** JSON (JavaScript Object Notation) é um padrão aberto para troca de mensagem entre sistemas, que será usado na API tanto para enviar (POST/PUT) como para receber (GET) informações.

#### 7.1 – Tokens de autenticação

Todas as requisições a API deverão conter no header os tokens de autenticação, conforme abaixo:

**Application Token (header: app-token):** Token que identifica a Aplicação que está efetuando a chamada aos recursos disponibilizados através da API.

**Authorization Token (header: auth-token):** Token que identifica o Lojista que está utilizando a Aplicação para efetuar chamadas aos recursos disponibilizados através da API (Não obrigatório em ambiente de sandbox).

##### 7.1.1 - Erros de autenticação

Alguns erros serão tratados durante a autenticação dos Tokens. Abaixo a lista dos erros:

**Ausência de um dos Tokens:** os dois Tokens devem ser passados em todas as requisições. Caso um deles esteja ausente, será retornado o erro 401 - Unauthorized.
**Token inexistente/errado:** se qualquer um dos Tokens passados não existir ou estiver com algum erro (caso tenha sido alterado), será retornado o erro 401 - Unauthorized.
**Tokens revogados (inválidos):** se qualquer um dos Tokens tiver sido revogado ele será considerado inválido e será retornado o erro 403 - Forbidden.

#### 7.2 – Paginação

A API Buscapé Marketplace, suporta nas operações que envolvem as coleções de cada recurso, a possibilidade de paginar os resultados.

Para obter resultados paginados ou então para especificar a faixa de registros a serem retornados pela API, o consumidor deverá informar dois parâmetros:

**limit:** quantidade de registros resultados a serem retornados ao consumidor.
O valor default é 50 e o valor máximo permitido também é 50\. Caso o consumidor passe um valor maior que 50, automaticamente a API sobrescreverá o valor pelo máximo permitido.

**offset:** indica a primeira posição da coleção de registros que será retornada ao consumidor.

Exemplo de busca utilizando limit e offset:

GET <a>http://api.buscape.com.br/orders/v2/status/new</a>[?limit=25&offset=50](http://api-marketplace.bonmarketplace.com.br/product?limit=25&offset=50)

No exemplo acima, a API Buscapé Marketplace irá retornar um total de 25 produtos, começando a partir do produto que está na posição de número 50, ou seja, irá recuperar todos os produtos no intervalo 50-75.

#### 7.3 – Códigos de retorno (HTTP status code)

Todos os serviços retornam códigos para indicar sucesso ou erro no processamento da solicitação. Além disso, os códigos servem para deixar claro ao consumidor da API qual foi o motivo do erro, sendo retornado mensagens explicando exatamente quais informações estão sendo enviadas erradas e devem ser repassadas para o lojista, caso tenha ocorrido.

A API Marketplace suporta os seguintes códigos de status HTTP:

| Código | Descrição | Tipo | Métodos HTTP permitidos |
| --- | --- | --- | --- |
| 200 (OK) | Requisição foi efetuada com sucesso. | Sucesso | Todos |
| 201 (Created) | Requisição efetuada com sucesso e indicando que um novo recurso foi criado e retornando o que foi criado e sua localização. | Sucesso | POST |
| 202 (Accepted) | Requisição de criação de recurso foi aceita com sucesso e retornando o recurso de controle de status do recurso a ser criado. | Sucesso | POST |
| 204 (No Content) | Requisição efetuada com sucesso sem mensagem de resposta. | Sucesso | PUT |
| 301 (Moved Permanently) | Indica que um recurso mudou seu identificador/localização. Deve ser feito um redirect para onde está a nova localização desse recurso. | Redirecionamento | Todos |
| 400 (Bad Request) | A solicitação não pôde ser entendida pelo servidor devido à sintaxe mal formada, (validação do request de acordo com o esperado). | Erro do Consumidor | POST, PUT |
| 401 (Unauthorized) | Usado especificamente quando uma tentativa de autenticação tentou ter sido efetuada, mas foi invalidada. | Erro do Consumidor | POST, PUT |
| 403 (Forbidden) | Algum recurso tentou ser acessado, mas é protegido e necessita de uma autenticação prévia. Usado também quando um recurso está indisponível. | Erro do Consumidor | Todos |
| 404 (Not Found) | O servidor não encontrou nenhum recurso na URI acessada pelo consumidor. | Erro do Consumidor | Todos |
| 405 (Method Not Allowed) | Deve ser utilizado caso a operação a ser realizada não seja suportada pelo recurso. | Erro do Consumidor | Todos |
| 409 (Conflict) | A requisição não pôde ser concluída devido a um conflito com o estado atual do recurso. Este código só é permitido em situações onde se espera que o consumidor possa ser capaz de resolver o conflito e reenviar o pedido (ex: submit duplo). | Erro do Consumidor | PUT |
| 415 (Unsupported Media Type) | O consumidor enviou um content-type diferente do que a API opera. Atualmente a API Marketplace só aceita o content-type application/json. | Erro do Consumidor | POST, PUT |
| 422 (Unprocessable Entity) | Significa que o servidor entende o tipo de conteúdo enviado (sintaxe da entidade está correta), mas não foi capaz de processar as instruções contidas (ex: validações de negócio). | Erro do Consumidor | Todos |
| 500 (Internal Server Error) | É usado quando o processamento falhar devido a circunstâncias imprevistas no lado do servidor. | Erro da API | Todos |

#### 7.4 – Retorno de erro

```json
{
  "code": "number",
  "error": "string",
  "details": []
}
```
| **Atributo** | **Descrição** |
| ------------ | ------------- |
| code | Código HTTP |
| error | Descrição do erro |
| details | Lista de detalhes do erro |

**Busca de pedidos por status ou id**

| **Caso** | **Mensagem** | **Code** |
|----------|--------------|----------|
| Não seja informado o status | Parametro STATUS não informado. | 400 | 
| Não seja informado o ID do pedido | ID do Pedido não informado. | 400 | 
| Parametro sellerId não seja encontrado na base | Seller não encontrado. | 400 | 
| Parametro sellerId seja diferente do cadastrado no pedido | Parametro Seller ID invalido. | 400 | 
| Id do produto da busca não seja econtrado na base | Produto não encontrado. | 400 | 

**Endpoint "Acceptance"**

| **Caso** | **Mensagem** | **Code** |
|----------|--------------|----------| 
| Parametro "accepted" vazio | Parametros inválidos. | 400 | 
| Campos 'id', 'eventDate', 'sellerOrderId', 'sellerId' vazio ou não enviado | Parametros inválidos. | 400 | 
| Pedido não seja encontrado na base | Pedido inválido. | 400 | 
| Parametro sellerId não seja encontrado na base | Seller não encontrado. | 400 | 
| Parametro sellerId seja diferente do cadastrado no pedido | Parametro Seller ID invalido. | 400 | 
| Exista os status no historico do pedido (seller_accepted, seller_waiting_payment) | Pedido ja aceito pelo Seller. | 200 | 
| No aceite, não exista informação de integração com a MOIP | Pedido não integrado com MOIP | 400 | 
| Haja algum erro interno de processamento | Erro interno durante alteração de status. | 500 | 
| Aceite do pedido ocorra com sucesso | Pedido aceito com sucesso. | 200 | 
| Recusa do pedido realizada com sucesso | Pedido recusado com sucesso. | 200 | 
| Erro interno durante operação de recusa do pedido. |  | 200 | 

**"Envio de Invoice e Tracking"**

| **Caso** | **Mensagem** | **Code** |
|----------|--------------|----------| 
| ID do pedido não seja informado | Pedido não informado. | 400 | 
| ID do pedido não seja encontrado | Pedido não encontrado. | 400 | 
| SellerID não seja informado ou seja diferente do cadastrado no pedido | Parametros inválidos. | 400 |

**Invoice**

| **Caso** | **Mensagem** | **Code** |
|----------|--------------|----------|
| Parametros (number, value, issuanceDate, invoiceKey) estejam vazios | Dados da Nota Fiscal inválidos. | 400 | 
| Parâmetro invoiceKey não for numérico e não houver 44 caracteres | Número da Nota Fiscal incorreto, utilize somente números e 44 caracteres. | 400 | 
| Já exista nota fiscal cadastrada ao pedido | Nota já existente para esse pedido. | 400 | 
| Não seja possível faturar pedido | Não é possível faturar pedido. | 400 | 
| Número da invoiceKey seja inválido | Nota Fiscal inválida, solicitado correção. | 400 | 
| Invoice Key esteja duplicada | A Nota Fiscal enviada já foi enviada para outro pedido, solicitado correção. | 400 | 
| Aconteça algum erro na criação da invoice / registro dos valores | Erro interno ao gravar Nota Fiscal. | 500 | 

**Tracking**

| **Caso** | **Mensagem** | **Code** | 
|----------|--------------|----------|
| Não seja econtrado nota fiscal para o pedido | Erro em atualizar tracking - Pedido sem nota fiscal cadastrada. | 400 | 
| Parametro tracking ou controlpoint esteja vazio | Parametros inválidos. | 400 | 
| Transportadora seja correios e trackingNumber sejá inválido | Tracking do Correios enviado inválido. | 400 | 
| Status do pedido não permita que seja cadastrado tracking para pedido  | Não é possível cadastrar tracking para este pedido. | 400 | 
| CNPJ enviado para transportadora seja inválido | CNPJ da transportadora inválido. | 400 |
| Seja cadastrado somente Invoice | Nota Fiscal cadastrada. | 200 | 
| Seja cadastrado somente Tracking | Tracking cadastrado. | 200 | 
| Seja cadastrado Invoice e Tracking | Nota Fiscal e Tracking cadastrados. | 200 | 
| Seja não seja cadastrado invoice nem Tracking | Sem alterações no pedido. | 200 | 
