PuntoPagos API
=============

Technical Manual (REST API)
Simplified Version

Version: 1.00

Last change: 15/06/2020 

## Introduction

The objective of this technical manual is to describe the functionalities and requirements for a commerce to integrate with our 
payment gateway.

## Technical Specs

Through our API, any commerce can access all the funtionalities of Puntopagos.com. The
kit has several methods that can be invoked through the web services.


The following diagram shows how is the communication process for a single sale transaction:

![API calls](https://raw.githubusercontent.com/PuntoPagos/documentacion/master/img/PP_eng.PNG "API calls")

All the information must be transmited encrypted with an SSL certificate under the https protocol. In case
the commerce shop does not have a digital certificate, they will need to request the documentation
manual using public keys.
Every request made to the web service, should be signed to ensure the integrity and the authenticity of
the request. 


### Environments

We have two environments, Sandbox and Production.

Sandbox base URL
```
https://sandbox.puntopagos.com
```

Production base URL
```
https://www.puntopagos.com
```

All services have the same URL scheme on both environments, example:
Create transaction service in Sandbox env:
```
https://sandbox.puntopagos.com/transaccion/crear
```
Create transaction service in Production env:
```
https://www.puntopagos.com/transaccion/crear
```
Sanbox has a reduced array of available payment methods, the only available methods are [Webpay y Ripley](#c%C3%B3digos-de-los-medios-de-pago)

###Integration requirements

The commerce must match our environments, having an environment for staging where the integration will be tested. In both enviroments the commerce must implement:
* **Notification URL**: This URL will be used by PuntoPagos to send the information about the outcome of a payment, see  [Step 4](#Step-4) for detailed info.
* **Success URL**: When the client succesfully completes a payment it will be redirected to this URL, see [Step 6](#Step-6)
* **Failure URL**: When the client cant complete a payment it will be redirected to this URL, see [Step 6](#step-6)



Upon creating an account in PuntoPagos the commerce will receive a IDKey and SecretKey wich will be used to sign the calls. 

Integration
=============
### Step 1

On step 1, the transaction is created on PuntoPagos through the web service.

```
URL: https://servidor/transaccion/crear
Method: POST
```
Headers:

* Fecha:  Date of request (specification RFC1123). 
  * Example: Fecha: Mon, 15 Jun 2009 20:45:30 GMT 

* Autorizacion:  Message signature using algorithm HMAC-SHA1 with the secret key given by
PuntoPagos, and then the message is coded on base64. The message format that should be signed
is the following: 


```
“transaccion/crear\n
<transactionID>\n
<amount with 2 decimals>\n
<date (specification RFC1123)>”
```

Example:

```
“transaccion/crear\n 9787415132\n
1000000.00\n
Mon, 15 Jun 2009 20:45:30 GMT”
```

After the message has been signed, the header will have the following format:
Autorizacion: PP <IDKey>:<SignedMessage> 

Example:

Parameters:

* IDKey=0PN5J17HBGZHT7ZZ3X82
* SecretKey=uV3F4YluFJax1cKnvbcGwgjvx4QpvB+leU8dUj2o

 * Autorizacion: PP 0PN5J17HBGZHT7ZZ3X82:AVrD3e9idIqAxRSH+15Yqz7qQkc=

The service will validate the signature message and as an additional security measure it will
validate that the message date won’t be older than 2 minutes. For this last measure, it is
essential that the commerce servers can be sincronized with an SNTP service.

Variables:

* trx_id: Client ID for the transaction, can't be longer than 15 characters.
* medio_pago: Payment method ID [Available payment methods](#c%C3%B3digos-de-los-medios-de-pago)
* monto: Total transaction amount
* detalle: Detailed description of product or services (optional)

JSON Example:

```json
{
	"trx_id":9787415132,
	"medio_pago":"999",
	"monto":1000000.00
}
```

### Step 2

Api response to Step 1 request:

Response:

* respuesta: 00 = OK (00 means the transaction was created, for other values see errors table)
* token: PuntoPagos ID for the transaction
* trx_id: Client ID for the transaction
* monto: Total amount of the transaction
* error: error message in case the answer is different of 00 (optional) 

JSON Example:

```json
{
	"respuesta":"00",
	"token":"9XJ08401WN0071839",
	"trx_id":9787415132,
	"medio_pago":"999",
	"monto":1000000.00
}
```

### Step 3

PuntoPagos redirects the client to the page of the chosen payment method to complete the payment.


### Step 4

When the client completes the payment, PuntoPagos will notify the commerce of the payment result through the notification service previosly defined by the commerce.
In case the commerce cannot implement the notification service, it should request the NVP POST documentation

```
URL: https://url_notificacion (commerce notification URL)
Method: POST
```

Headers:

* Fecha: Date of request (RFC1123 specification). 
  * Fecha: Mon, 15 Jun 2009 20:45:30 GMT
* Autorizacion: PuntoPagos will sign the message using the client secret key. The sign message will have the following format: 


```
“transaccion/notificacion\n
<token>\n
<transactionID>\n
<amount with 2 decimals>\n
<date (specification RFC1123)>”
```

Example:

```
“transaccion/notificacion\n 9XJ08401WN0071839\n
9787415132\n
1000000.00\n
Mon, 15 Jun 2009 20:48:30 GMT”
```

Just as in Step 1, after the message has been signed, the header will have the following format:

Autorizacion: PP <IDKey>:<SignedMessage>
Example:

* Autorizacion: PP 0PN5J17HBGZHT7ZZ3X82:fU6+JLYWzOSGuo76XJzT/Z596Qg=

The commerce must validate the message signature in order to confirm the origin of it. If the message is wrong signed, it should return an authentification error. 


Variables:
* token: PuntoPagos ID for the transaction
* trx_id: Client ID for the transaction
* medio_pago: Payment method ID
* monto: Total transaction amount
* fecha_aprobacion: Transaction approval date (Format: yyyy-MM-ddTHH:mm:ss)
* numero_tarjeta: Last 4 digits of the credit card (optional) 
* num_cuotas: Installments number (optional) 
* tipo_cuotas: Installment type (opcional)
* valor_cuota: Installment value (opcional)
* primer_vencimiento: First installment due date (Formato:yyyy-MM-dd) (optional)
* numero_operacion: : Payment Method transaction ID
* codigo_autorizacion: Payment Method  payment authorization code

JSON Example:
	
```json
{
	"token":"9XJ08401WN0071839",
	"trx_id":9787415132,
	"medio_pago":"999",
	"monto":1000000.00,
	"fecha":"2009-06-15T20:49:00",
	"numero_operacion":"7897851487",
	"codigo_autorizacion":"34581"
}
```

If the commerce can´t implement an HTTPS notification service, then only the token will be notified.
The commerce should make a GET call to our secured transaction information service (``transaccion/traer``) to get the complete information.

```
ULR: https://url_notificacion<token> (lado del comercio)
Method: GET
```

### Step 5

Notification service Response:
* respuesta: 00 = OK, 99 = error
* token: Unique Identifier of the transaction at PuntoPagos
* error: error message in case the answer is different from 00 (optional)

JSON Example: 

```json
{
	"respuesta":"00",
	"token":"9XJ08401WN0071839"
} 
```


### Step 6

After a successful payment, PuntoPagos will redirect the client to the commerce success URL. 


```
URL: https://url_exito<token> (commerce side)
Method: GET
```

Example:

Commerce Success URL: ``https://micomercio.com/transacciones/exito/``
Redirection: ``https://micomercio.com/transacciones/exito/9XJ08401WN0071839``

Before the transaction is approved, the commerce should verify that the notification of the
transaction has arrived correctly throught the token. In case the notification has not been received the commerce could verify the payment state with the following function:
 https://server/transaction/<token> and show the corresponding proofs.  

If the client couldn't complete the payment, PuntoPagos will redirect to the commerce failure URL, where the
commerce can show the client that the transaction could not be processed. 

```
URL: https://url_fracaso<token>
Method: GET
```

Example:

Commerce failure URL: ``https://micomercio.com/transacciones/fracaso/``
Redirection: ``https://micomercio.com/transacciones/exito/9XJ08401WN0071839``

Functions
=============

### Transaction State

With this function the commerce can at any time verify the payment status of a given transaction. 

```
URL: https://servidor/transaccion/<token>
Method: GET
```

Headers:

* Fecha: Date of request (RFC1123 specification). 
  * Fecha: Mon, 15 Jun 2009 20:50:30 GMT

* Autorizacion: PuntoPagos will sign the message using the client secret key. The signed
message will have the following format: 


```
“transaccion/traer\n
<token>\n
<identificador transacción>\n
<monto operación con dos decimales>\n
<fecha mismo formato del header>”
```

Example:

```
“transaccion/traer\n
9XJ08401WN0071839\n
9787415132\n
1000000.00\n
Mon, 15 Jun 2009 20:50:30 GMT”
```

After the message is signed, the header will have the following format:  
Autorizacion: PP <IDKey>:<SignedMessage>

Example:

Parámetros:

* IDKey=0PN5J17HBGZHT7ZZ3X82
* LlaveSecreta=uV3F4YluFJax1cKnvbcGwgjvx4QpvB+leU8dUj2o
  * Autorizacion: PP 0PN5J17HBGZHT7ZZ3X82:AVrD3e9idIqAxRSH+15Yqz7qQkc=

Response:

* respuesta: 00 = OK (00 means the transaction was created, for other values see errors table)
* token: PuntoPagos ID for the transaction
* trx_id: Client ID for the transaction
* medio_pago: Payment method ID
* monto: Total transaction amount
* fecha_aprobacion: Transaction approval date (Format: yyyy-MM-ddTHH:mm:ss)
* numero_tarjeta: Last 4 digits of the credit card (optional) 
* num_cuotas: Installments number (optional) 
* tipo_cuotas: Installment type (opcional)
* valor_cuota: Installment value (opcional)
* primer_vencimiento: First installment due date (Formato:yyyy-MM-dd) (optional)
* numero_operacion: : Payment Method transaction ID
* codigo_autorizacion: Payment Method  payment authorization code
* error: error message in case the answer is different from 00 (optional)

JSON example:

```json
{
	"respuesta":"00",
	"token":"9XJ08401WN0071839",
	"trx_id":9787415132,
	"medio_pago":"999",
	"monto":1000000.00,
	"fecha":"2009-06-15T20:49:00",
	"numero_operacion":"7897851487",
	"codigo_autorizacion":"34581"
}
```

```json
{
	"respuesta":"99",
	"token":"9XJ08401WN0071839",
	"error":"Pago Rechazado"
}
```

## Appendix

### Payment Methods IDs

ID     | Description
-------|------------------------
2      | Tarjeta Presto
3      | Webpay Transbank (chilean debit and credit card)
4      | Banco de Chile button
5      | Banco BCI button
7      | Banco Estado button
10     | Ripley Card


### Error codes


Code   | Description
-------|------------------------
1      | Rejected Transaction
2      | Void Transaction
6      | Incomplete Transaction
7      | Payment Method Error

### Staging Test Data 

Debit and credit card TEST data for WebPay

Tarjeta | Numero | CCV | Expiracion | Resultado Esperado
--------|--------|-----|------------|--------------------
Visa    | 4051885600446623 | 123 | cualquiera | Exito
Mastercard | 5186059559590568 | 123 | cualquiera |Fracaso 

Webpay Test Environment credentials:
RUT ``11111111-1``
Password ``123``

#### Ripley card


Credentials
```
Rut: 16389806-3
Clave: 1234
Numero de tarjeta: 6281561467501027
Codigo de verificacion: 360
Vencimiento: 08-20
Coordenadas: 77 77 77 (uno en cada box que solicita)
Mail: test@test.com
```
