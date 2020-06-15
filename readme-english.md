PuntoPagos API

Technical Manual (REST API)
Simplified Version




Version: 0.98 (Draft)
Last change: 18/5/2010 
=============

## Introduction

The objective of this technical manual is to describe the functionalities and required parameters for the
web services provided by PuntoPagos.com, for the correct operation of the payment buttons of our
electronic collection services.
For the particular case of the REST documentation, it will provide the REST services on JSON and
XML format. The JSON format is the default one.
This document has been created for developers. 


## Technical Specs

Through our integration kit (API), any commerce can access all the funtionalities of Puntopagos.com. The
kit has several methods that can be invoked through the web services.


The following diagram shows how is the communication process for a single sale transaction:

![Diagrama de llamadas](https://raw.githubusercontent.com/PuntoPagos/documentacion/master/img/flujo-llamadas.png "Diagrama de llamadas")

All the information must be transmited encrypted with an SSL certifícate under the https protocol. In case
the commerce shop does not have a digital certifícate, they will need to request the documentation
manual using public keys.
Every request made to the web service, should be signed to ensure the integrity and the authenticity of
the request. 


### Modo Sandbox (desarrollo) versus Producción

Los 2 ambientes funcionan exactamente de la misma manera. La idea es que primero se realicen pruebas en modo sandbox y, una vez que la integración funcione correctamente, pasar a produccion.

La url en modo sanbox es:
```
https://sandbox.puntopagos.com
```

La url de producción es:
```
https://www.puntopagos.com
```

Las rutas son las mismas en modo Sandbox y produccion, por ejemplo, en modo sanbox la url para crear una transaccion es
```
https://sandbox.puntopagos.com/transaccion/crear
```
Mientras que en producción sería:
```
https://www.puntopagos.com/transaccion/crear
```
Por el momento los únicos medios de pago que se pueden usar en modo sandbox, son [Webpay y Ripley](#c%C3%B3digos-de-los-medios-de-pago)

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
<transactionID>>\n
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
* token: Punto Pagos ID for the transaction
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
* Autorizacion: Punto Pagos will sign the message using the client secret key. The sign message will have the following format: 


```
“transaccion/notificacion\n
<token>\n
<identificador transacción>\n
<monto operación con dos decimales>\n
<fecha mismo formato del header>”
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
* token: Punto Pagos ID for the transaction
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
* token: Unique Identifier of the transaction at Punto Pagos
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

If the client couldn't complete the payment, Punto Pagos will redirect to the commerce failure URL, where the
commerce can show the client that the transaction could not be processed. 

```
URL: https://url_fracaso<token>
Method: GET
```

Example:

Commerce failure URL: ``https://micomercio.com/transacciones/fracaso/``
Redirection: ``https://micomercio.com/transacciones/exito/9XJ08401WN0071839``

### Functions

El comercio en todo momento podrá verificar el pago de una determinada transacción.

```
Función: https://servidor/transaccion/<token>
Método: GET
```

Headers:

* Fecha: Fecha del request según la especificación RFC1123.
  * Fecha: Mon, 15 Jun 2009 20:50:30 GMT

* Autorizacion: Punto Pagos firmará el mensaje utilizando la llave secreta del cliente, el mensaje a firmar tendrá el siguiente formato:

```
“transaccion/traer\n
<token>\n
<identificador transacción>\n
<monto operación con dos decimales>\n
<fecha mismo formato del header>”
```

Ejemplo:

```
“transaccion/traer\n
9XJ08401WN0071839\n
9787415132\n
1000000.00\n
Mon, 15 Jun 2009 20:50:30 GMT”
```

Después de firmado el mensaje el header tendrá el siguiente formato: Autorizacion: PP <LlaveID>:<MensajeFirmado>

Ejemplo:

Parámetros:

* LlaveID=0PN5J17HBGZHT7ZZ3X82
* LlaveSecreta=uV3F4YluFJax1cKnvbcGwgjvx4QpvB+leU8dUj2o
  * Autorizacion: PP 0PN5J17HBGZHT7ZZ3X82:AVrD3e9idIqAxRSH+15Yqz7qQkc=

Respuesta:

* respuesta: 00 = OK (Otras según tabla errores)
* token: Identificador único de la transacción en Punto Pagos
* trx_id:Identificadorúnicodelatransaccióndelcliente (opcional)
* medio_pago:Identificadordelmediodepago(opcional)
* monto:Montototaldelatransacción(opcional)
* fecha_aprobacion:Fechadeaprobacióndelatransacción(Formato:yyyy-MM-ddTHH:mm:ss) (opcional)
* numero_tarjeta: 4 últimos dígitos de la tarjeta (opcional)
* num_cuotas: número de cuotas (opcional)
* tipo_cuotas: tipo de cuotas (opcional)
* valor_cuota: valor de cada cuota (opcional)
* primer_vencimiento:primervencimiento(Formato:yyyy-MM-dd) (opcional)
* numero_operacion: Número de operación en la institución financiera (opcional)
* codigo_autorizacion:Códigodeautorizacióndelatransacción(opcional)
* error: mensaje de error en caso que la respuesta sea distinta de 00 (opcional)

Ejemplos json:

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

##Requisitos para implementar PuntoPagos

Para comenzar la integración necesitamos que nos envíen las 3 URLs que PuntoPagos utiliza para interactuar con cada comercio. Estas son:

* **Url Notificacion**: Es la url a la cual PuntoPagos notificará el resultado de la transacción en curso. Más detalles en el [Paso 4](#paso-4)
* **Url Exito**: Esta es la url a la cual PuntoPagos redirigirá al comprador si la transacción fue exitosa. Más detalles en el [Paso 5](#paso-5)
* **Url Fracaso**: Ídem anterior, pero para el caso que la transacción no sea exitosa. Más detalles en el [Paso 5](#paso-5)

En un principio las 3 urls serán utilizadas en el ambiente de Sandbox para realizar pruebas. Una vez que nos confirmen que todo está funcionando bien y verifiquemos que la integracion está ok, PuntoPagos creará el ambiente de producción.

Cada ambiente, SandBox y Producción, puede tener un set diferente de urls. Esto permite a un comercio que ya está en producción, seguir haciendo pruebas de integración ante eventuales mejoras sin afectar el ambiente de producción.

## Anexos

### Códigos de los medios de pago

Código | Descripción
-------|------------------------
2      | Tarjeta Presto
3      | Webpay Transbank (tarjetas de crédito y débito)
4      | Botón de Pago Banco de Chile
5      | Botón de Pago BCI
6      | Botón de Pago TBanc
7      | Botón de Pago Banco Estado
16     | Botón de Pago BBVA
10     | Tarjeta Ripley
15     | Paypal

### Códigos de error

Estos son los diferentes codigos de error que puede informar la API

Código | Descripcion
-------|------------------------
1      | Transaccion Rechazada
2      | Transaccion Anulada
6      | Transaccion Incompleta
7      | Error del financiador

### Datos de prueba para el modo sandbox

Números de tarjeta para WebPay

Tarjeta | Numero | CCV | Expiracion | Resultado Esperado
--------|--------|-----|------------|--------------------
Visa    | 4051885600446623 | 123 | cualquiera | Exito
Mastercard | 5186059559590568 | 123 | cualquiera |Fracaso 

Al pedir un RUT se debe ingresar ``11111111-1`` y la clave ``123``

#### Ripley

Para usar Ripley en modo sandbox estos son los datos:

```
Rut: 16389806-3
Clave: 1234
Numero de tarjeta: 6281561467501027
Codigo de verificacion: 360
Vencimiento: 08-20
Coordenadas: 77 77 77 (uno en cada box que solicita)
Mail: test@test.com
```
