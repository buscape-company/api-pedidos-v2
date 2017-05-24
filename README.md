Buscapé Marketplace

# API Pedidos - V2 

### 1 - Fluxo de informações 

Principais interações que podem ser realizadas entre parceiros (consumidores) e a API Pedidos Buscapé.

![Fluxo de interações](/images/Fluxo_Buscape_Marketplace.png)

Obs.: Os e-mails transacionais que reportam o status do pedido para os clientes devem ser desabilitados. Esse tipo de e-mail já é enviado pelo Buscapé Marketplace.

### 2 - Consulta pedido

A consulta de pedido retorna o pedido no momento atual. O pedido pode está nos seguintes status:

| Status de pedido | Descrição |
| -----------------| ----------|
| new | Pedido novo no Buscapé Marketplace |
| accept | Pedido aceito pelo parceiro |
| not_accept | Pedido recusado pelo parceiro |
| pending | Aguardando meio de pagamento |
| approved | Pagamento aprovado |
| not_approved | Pagamento não aprovado |
| cancelled | Pedido cancelado |
| invoiced | Pedido Faturado |
| in_hosting | Item na transportadora |
| in_route | Item em rota de entrega |
| retrying | Retentativas |
| reversal | Devolução |
| delivered | Pedido totalmente entregue |

#### 2.1 - Consulta pedido por Status

O Buscapé Marketplace disponibiliza o serviço para recuperar uma lista de pedidos por status que estão relacionados com o token do parceiro informado. Este serviço irá retornar um ou mais pedidos conforme o status.

- HTTP Request Method: GET

- HTTP Request URL: <a>http://api.buscape.com.br/orders/status/{STATUS}?limit={NUMBER}&offset={NUMBER</a>}&lastUpdate={DATE}

- Body Response: Array [Orders](#IntegraçãoPedidos-V2-Orders)

#### 2.2 - Consulta pedido por ID Buscapé

O Buscapé Marketplace disponibiliza o serviço para recuperar detalhes do pedido informado

- HTTP Request Method: GET

- HTTP Request URL Pattern: [http://api.buscape.com.br/orders](http://api.buscape.com.br/orders) /{ID_Buscape}

- Body Response: [Orders](#IntegraçãoPedidos-V2-Orders)

### 3 - Consulta de estoque

O Buscapé Marketplace realiza a consulta de estoque, que consiste em perguntar ao parceiro (sistematicamente), se existe estoque para os itens do pedido.

Caso o parceiro confirme a disponibilidade de estoque, o Buscapé irá reduzir internamente o estoque dos itens associados ao pedido, conforme a quantidade de cada item, e o pedido continuará o fluxo normal; caso contrário o pedido será cancelado. Para isto o parceiro deverá implementar um serviço de resposta para a consulta de estoque.

Para isto o parceiro deve implementar um Webservice REST que use formato JSON. Este serviço será responsável por receber as requisições de consulta de estoque vindos do Buscapé, e o parceiro deverá responder se tem estoque ou não para os itens do pedido. Caso o parceiro confirme a disponibilidade de estoque, o Buscapé irá reduzir internamente o estoque dos itens associados ao pedido, conforme a quantidade de cada item.

#### 3.1 - Requisição REST:

A requisição que o parceiro irá receber por método POST terá o seguinte formato:

- HTTP Method: POST

- JSON Request: [Stock](#IntegraçãoPedidos-V2-StockRequest)

#### 3.2 - Resposta REST:

A requisição que o parceiro irá enviar por método POST terá o seguinte formato:

- HTTP Method: POST

- JSON Request: [Stock](#IntegraçãoPedidos-V2-StockResponse)

### 4 - Notificação de pedido

O Buscapé Marketplace disponibiliza a funcionalidade de Notificação de Eventos: sempre que um Evento ocorrer (por exemplo um novo pedido), iremos notificar a sua Aplicação

Para integrar seu sistema e aproveitar ao máximo os recursos de notificações, é necessário que sua aplicação esteja preparada para receber um POST de um JSON no formato:

- HTTP Method: POST

- JSON Request: [Notification](#IntegraçãoPedidos-V2-Notification)

#### 4.1 - URL de Callback

É o endereço que será chamado pelo Buscapé Marketplace para notificar o parceiro que ocorreu algo novo no sistema. Se resume ao endereço de um serviço rest que a aplicação desenvolvida pelo parceiro saberá interpretar. Todas as chamadas deverão ter suporte ao método POST e deverão retornar o status 200 ou 201 em caso de sucesso na requisição.

#### 4.2 - Aceite

Ao receber um pedido novo o parceiro deverá enviar uma notificação de Aceite para o Buscapé Marketplace.

É necessário que sua aplicação esteja preparada para enviar um POST de um JSON no formato:

- HTTP Method: POST

- JSON Request: [Acceptance](#IntegraçãoPedidos-V2-Acceptance)

#### 4.3 - Situações de Exceção

Ocorrerão até 5 tentativas de notificação, em intervalos regulares. Se em nenhuma delas o sistema do Buscapé receber código esperado, a notificação será armazenada como não entregue e será cancelada. Todos os históricos das chamadas são armazenados por até 60 dias e poderão ser disponibilizados caso solicitado.

É recomendado que o sistema do parceiro consulte os serviços em intervalos regulares, não inferiores á 30 minutos, para checar se existe algum novo recurso disponível, pedido realizado, pagamento aprovado, etc. Por convenção, as notificações não possuem todos os detalhes de uma transação, necessitando assim a chamada aos respectivos serviços relacionados.

### 5 - Atualiza o tracking do pedido

O Buscapé Marketplace disponibiliza o serviço para registrar uma nova operação de tracking de Entrega para os itens do pedido

- HTTP Request Method: POST

- HTTP Request URL Pattern: [http://api.buscape.com.br/orders](http://api.buscape.com.br/orders) /{ID_Buscape}/tracking

- JSON Request: [Tracking](#IntegraçãoPedidos-V2-Tracking)

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
| item.sku | SKU do Buscapé |
| item.skuSellerId | SKU do parceiro |
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

*   Status **in_hosting**: 
    Nesse status pode enviar <span>**trackingNumber** e/ou **carrier**, além dos demais atributos.</span>

| **Atributo** | **Descrição** |
| ------------ | ------------- |
| item | Item alterado pela operação de tracking |
| item.sku | SKU do Buscapé |
| item.skuSellerId | SKU do parceiro |
| trackingNumber | ID do objeto na transportadora |
| carrier | Informações sobre transportadora |
| carrier.name | Nome da transportadora |
| carrier.cnpj | CNPJ da transportadora |
| tracking | Informações de sobre o Tracking |
| tracking.controlPoint | Status do pedido |
| tracking.description | Descrição adicional sobre tracking |
| tracking.occurredAt | Data da ocorrência |

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
         "paymentDueAt":"date",
         "approvedAt":"date"
      }
   ],
   "sellerOrder":"string",
   "totalFreight":"number",
   "purchaseAt":"date",
   "lastUpdateAt":"date",
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
               "lockTTL":"number",
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
               "tracking":{
                  "description":"string",
                  "occurredAt":"date",
                  "controlPoint":"string"
               },
               "invoice" : {
                  "number": "number",
                  "value": "number",
                  "url": "string",
                  "issuanceDate": "date",
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
{
   [
      "buscapeID" : "number", 
      "skuSellerId" : "number",
      "available" : "number",
      "crossDockingTime":"number",    
      "message" : "string"
   ]
}
```



#### 6.3 - Notification

```json
{
   "eventDate":"date",
   "sellerId":"number",
   "orderUri":"string",
   "order":"order"
}
```


#### 6.4 - Acceptance

```json
{
    "eventDate":"date",
    "accepted" : "boolean",
    "sellerOrder" : "string",
    "message" : "string"
}
```

 |

#### 6.5 - Tracking


```json
[
   {
      "item":{
         "sku":"string",
         "skuSellerId":"string"
      },
      "selectedSla":"string",
      "lockTTL":"number",
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
         "issuanceDate":"date",
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
- Formatação de data: YYYY-MM-DD
- Formatação de data/hora: YYYY-MM-DDThh:mm:ss.sTZD[ ( Exemplo: 2013-06-28T08:54:00.000-03:00 )](http://mmss.stzd/)

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

GET <a>http://api.buscape.com.br/orders/status/new</a>[?limit=25&offset=50](http://api-marketplace.bonmarketplace.com.br/product?limit=25&offset=50)

No exemplo acima, a API Buscapé Marketplace irá retornar um total de 25 produtos, começando a partir do produto que está na posição de número 50, ou seja, irá recuperar todos os produtos no intervalo 50-75.

#### 7.3 – Códigos de retorno (HTTP status code)

Todos os serviços retornam códigos para indicar sucesso ou erro no processamento da solicitação. Além disso, os códigos servem para deixar claro ao consumidor da API qual foi o motivo do erro, caso tenha ocorrido.

A API Marketplace suporta os seguintes códigos de status HTTP:

| Código | Descrição | Tipo | Métodos HTTP permitidos |
| --- | --- | --- | --- |
| 200
(OK) | Requisição foi efetuada com sucesso. | Sucesso | Todos |
| 201
(Created) | Requisição efetuada com sucesso e indicando que um novo recurso foi criado e retornando o que foi criado e sua localização. | Sucesso | POST |
| 202
(Accepted) | Requisição de criação de recurso foi aceita com sucesso e retornando o recurso de controle de status do recurso a ser criado. | Sucesso | POST |
| 204
(No Content) | Requisição efetuada com sucesso sem mensagem de resposta. | Sucesso | PUT |
| 301
(Moved Permanently) | Indica que um recurso mudou seu identificador/localização. Deve ser feito um redirect para onde está a nova localização desse recurso. | Redirecionamento | Todos |
| 400
(Bad Request) | A solicitação não pôde ser entendida pelo servidor devido à sintaxe mal formada, (validação do request de acordo com o esperado). | Erro do Consumidor | POST, PUT |
| 401
(Unauthorized) | Usado especificamente quando uma tentativa de autenticação tentou ter sido efetuada, mas foi invalidada. | Erro do Consumidor | POST, PUT |
| 403
(Forbidden) | Algum recurso tentou ser acessado, mas é protegido e necessita de uma autenticação prévia. Usado também quando um recurso está indisponível. | Erro do Consumidor | Todos |
| 404
(Not Found) | O servidor não encontrou nenhum recurso na URI acessada pelo consumidor. | Erro do Consumidor | Todos |
| 405
(Method Not Allowed) | Deve ser utilizado caso a operação a ser realizada não seja suportada pelo recurso. | Erro do Consumidor | Todos |
| 409
(Conflict) | A requisição não pôde ser concluída devido a um conflito com o estado atual do recurso. Este código só é permitido em situações onde se espera que o consumidor possa ser capaz de resolver o conflito e reenviar o pedido (ex: submit duplo). | Erro do Consumidor | PUT |
| 415
(Unsupported Media Type) | O consumidor enviou um content-type diferente do que a API opera. 
Atualmente a API Marketplace só aceita o content-type application/json. | Erro do Consumidor | POST, PUT |
| 422
(Unprocessable Entity) | Significa que o servidor entende o tipo de conteúdo enviado (sintaxe da entidade está correta), mas não foi capaz de processar as instruções contidas (ex: validações de negócio). | Erro do Consumidor | Todos |
| 500
(Internal Server Error) | É usado quando o processamento falhar devido a circunstâncias imprevistas no lado do servidor. | Erro da API | Todos |
