PuntoPagos API
=============
Technical Manual (REST API)
Simplified Version
Version: 0.98 (Draft)
Last change: 18/5/2010 
=============
## Introduction

The objective of this technical manual is to describe the functionalities and required parameters for the
web services provided by PuntoPagos.com, for the correct operation of the payment buttons of our
electronic collection services.
Fort he particular case of the REST documentation, it will provide the REST services on JSON and
XML format. The JSON format is the default one.
This document has been created for developers.

## Technical Specifications

Through our integration kit (API), any commerce can access all the funtionalities of Puntopagos.com. The
kit has several methods that can be invoke through the web services.

The following diagram shows how is the communication process for a single sale transaction:

![Diagrama de llamadas](https://raw.githubusercontent.com/PuntoPagos/documentacion/master/img/flujo-llamadas.png "Diagrama de llamadas")

All the information must be transmited encrypted with an SSL certifícate under the https protocol. In case
the commerce shop does not have a digital certifícate, they will need to request the documentation
manual using public keys.
Every request made to the web service, should be signed to ensure the integrity and the authenticity of
the request. 
Step 1
On step 1, the transaction is created on PuntoPagos through the web service.
Function: https://server/transaction/create
Method: POST
Headers:
• Date: Date request (specification RFC1123).
◦ Date: Mon, 15 Jun 2009 20:45:30 GMT
• Authorization: Message signature using algorithm HMAC-SHA1 with the secret key given by
PuntoPagos, and then the message is coded on base64. The message format that should be signed
is the following:
◦ “transaccion/crear\n
<transaction ID>\n
<amount with 2 decimals>\n
< header date format>”
Example:
“transaccion/crear\n
9787415132\n
1000000.00\n
Mon, 15 Jun 2009 20:45:30 GMT”
After the message has been signed, the header will have the following
format:
Authorization: PP <IDKey>:<SignedMessage>
Example:
Parameters: IDKey=0PN5J17HBGZHT7ZZ3X82
SecretKey=uV3F4YluFJax1cKnvbcGwgjvx4QpvB+leU8dUj2o
◦ Autorizacion: PP 0PN5J17HBGZHT7ZZ3X82:AVrD3e9idIqAxRSH+15Yqz7qQkc=

The service will validate the signature message and as an additional security measure it will
validate that the message date won’t be older than 2 minutes. For this last measure, it is
essential that the commerce servers can be sincronized with an SNTP service.
Variables:
• trx_id: Unique Identifier of the client transaction
• medio_pago: Identifier of the payment gateway (the web service is available for getting the
availables payment gateways for the client.
• monto: Total transaction amount
• detalle: Detail description of the ítem or product that has been paid.
Example json: {"trx_id":9787415132,"medio_pago":"999","monto":1000000.00}
Step 2
Answer (respuesta=answer)
• respuesta: 00 = OK (Other please look the error table)
• token: Unique Identifier of the transaction at Punto Pagos
• trx_id: Unique Identifier of the client transaction
• monto: Total amount of the transaction
• error: error message in case the answer is different of 00 (optional)
Example json:
{"respuesta":"00","token":"9XJ08401WN0071839","trx_id":9787415132,"medio_pago":"999",
"monto":1000000.00}
Step 3
On step 2, the client is redirected to the page of the chosen institution for payment.
Funtion: https://server/transaction/procesar/<token>
Method: GET Example:
https://server/transaction/procesar/9XJ08401WN0071839 

Step 4
After the client makes the payment on the institution, PuntoPagos will notify the commerce that the
payment has been made to the service previosly defined by the commerce. In case the client cannot
implement the REST service, he should request the NVP POST documentation.
Funtion: https://url_notificacion (commerce side)
Method: POST
Headers:
• Date: Date request (RFC1123 specification).
◦ Date: Mon, 15 Jun 2009 20:45:30 GMT
• Authorization: Punto Pagos will sign the message using the client secret key. The sign
message will have the following format:
◦ “transaccion/notificacion\n
<token>\n
<transaction ID>\n
<total amount with 2 decimals>\n
<date with same format as header>”
Example: “transaccion/notificacion\n
9XJ08401WN0071839\n
9787415132\n
1000000.00\n
Mon, 15 Jun 2009 20:48:30 GMT”
Just as in Step 1, after the message has been signed, the header will have the following
format:
Autorizacion: PP <IDKey>:<SignedMessage>
◦ Example:
Authorizacion: PP 0PN5J17HBGZHT7ZZ3X82:fU6+JLYWzOSGuo76XJzT/Z596Qg=
The commerce should validate the message signature in order to confirm the origin of it. If
the message is wrong signed, it should return an authentification error.
Variables:
• token: Unique Identifier of the transaction at Punto Pagos
• trx_id: Unique Identifier of the client transaction.
Page 5 

• medio_pago: Payment Gateway Identifier
• monto: Total transaction amount
• fecha_aprobacion: Transaction approval date (Format: yyyy-MM-ddTHH:mm:ss)
• numero_tarjeta: Last 4 digits of the credit card (optional)
• num_cuotas: Number of quotas (optional)
• tipo_cuotas: Kind de cuotas (optional)
• valor_cuota: value of each quota (optional)
• primer_vencimiento: first payment date (Format: yyyy-MM-dd) (optional)
• numero_operacion: Operation number on the institutiona (optional)
• codigo_autorizacion: Authorization code of the transaction
Example json:
{"token":"9XJ08401WN0071839","trx_id":9787415132,"medio_pago":"999","monto":1000000.0
Step 5
0,"fecha":"2009-06-15T20:49:00","numero_operacion":"7897851487",
"codigo_autorizacion":"34581"}
Answer:
• respuesta: 00 = OK, 99 = error
• token: Unique Identifier of the transaction at Punto Pagos
• error: error message in case the answer is different from 00 (optional)
Example json: {"respuesta":"00","token":"9XJ08401WN0071839"}
Step 6
After the processed payment, and in case it is successful, PuntoPagos will redirect the client to the
commerce page.
Funtion: https://url_exito<token> (lado del comercio)
Method: GET
Example: https://url_exito9XJ08401WN0071839
Before the transaction is approved, the commerce should verify that the notification of the
transaction has arrived correctly throught the token. In case the notification has not been received

The commerce could verify the payment state with the following function:
 https://server/transaction/<token> and show the corresponding proofs.
In case the payment could not be processed, Punto Pagos will redirect to the failure url where the
commerce can show the client that the transaction could not be processed.
Funtion: https://url_fracaso<token>
Method: GET
Example: https://url_ fracaso9XJ08401WN0071839 

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

### Paso 1

En el paso 1 se crea la transacción en punto pagos, a través del servicio web.

```
URL: https://servidor/transaccion/crear
Método: POST
```
Headers:

* Fecha: Fecha del request según la especificación RFC1123.
  * Ejemplo: Fecha: Mon, 15 Jun 2009 20:45:30 GMT

* Autorizacion: Firma del mensaje utilizando el algoritmo HMAC-SHA1 con la llave secreta entregada por Punto Pagos y luego el mensaje se codifica en base64, el formato del mensaje a firmar es el siguiente:

```
“transaccion/crear\n
<identificador transacción>\n
<monto operación con dos decimales>\n
<fecha mismo formato del header>”
```

Ejemplo:

```
“transaccion/crear\n 9787415132\n
1000000.00\n
Mon, 15 Jun 2009 20:45:30 GMT”
```

Después de firmado el mensaje el header tendrá el siguiente formato: Autorizacion: PP <LlaveID>:<MensajeFirmado>

Ejemplo:

Parametros:

* LlaveID=0PN5J17HBGZHT7ZZ3X82
* LlaveSecreta=uV3F4YluFJax1cKnvbcGwgjvx4QpvB+leU8dUj2o
  * Autorizacion: PP 0PN5J17HBGZHT7ZZ3X82:AVrD3e9idIqAxRSH+15Yqz7qQkc=

El servicio validará la firma del mensaje y como medida de seguridad adicional validará que la fecha del mensaje no tenga una antigüedad superior a 2 minutos, para esta comprobación es de vital importancia que los servidores del cliente estén sincronizados con algún servicio SNTP, esto con el fin de evitar ataques de fuerza bruta sobre un mismo mensaje.

Variables:

* trx_id: Identificador único de la transacción del cliente. Su longitud no debe ser mayor a 15 caracteres
* medio_pago: Identificador del medio de pago. [Medios disponibles](#c%C3%B3digos-de-los-medios-de-pago)
* monto: Monto total de la transacción
* detalle: Descripción del producto o servicio pagado (opcional)

Ejemplo json:

```json
{
	"trx_id":9787415132,
	"medio_pago":"999",
	"monto":1000000.00
}
```

### Paso 2

Nuestra API devuelve la respuesta al request realizado en el paso 1.

Respuesta:

* respuesta: 00 = OK (Otras según tabla errores)
* token: Identificador único de la transacción en Punto Pagos
* trx_id: Identificador único de la transacción del cliente. Su longitud no debe ser mayor a 15 caracter
* monto: Monto total de la transacción
* error: mensaje de error en caso que la respuesta sea distinta de 00 (opcional)

Ejemplo json:

```json
{
	"respuesta":"00",
	"token":"9XJ08401WN0071839",
	"trx_id":9787415132,
	"medio_pago":"999",
	"monto":1000000.00
}
```

### Paso 3

Se redirecciona al cliente a la URL de procesamiento usando el token obtenido en el paso 2

```
Función: https://servidor/transaccion/procesar/<token>
Método: GET
```

Ejemplo: ``https://servidor/transaccion/procesar/9XJ08401WN0071839``

### Paso 4

Luego que el cliente realiza el pago en la institución financiera, Punto Pagos procederá a notificar al comercio que se efectuó el pago, a la URL de notificacion previamente definida por el comercio.

En el caso que la url de notificacion sea via HTTPS (utilizando encriptación SSL), el servidor hace una llamada a la URL de notificacion:

```
Función: https://url_notificacion (lado del comercio)
Método: POST
```

Headers:

* Fecha: Fecha del request según la especificación RFC1123.
  * Fecha: Mon, 15 Jun 2009 20:45:30 GMT
* Autorizacion: Punto Pagos firmará el mensaje utilizando la llave secreta del cliente, el mensaje a firmar tendrá el siguiente formato:

```
“transaccion/notificacion\n
<token>\n
<identificador transacción>\n
<monto operación con dos decimales>\n
<fecha mismo formato del header>”
```

Ejemplo:

```
“transaccion/notificacion\n 9XJ08401WN0071839\n
9787415132\n
1000000.00\n
Mon, 15 Jun 2009 20:48:30 GMT”
```

Al igual que en el paso 1, después de firmado el mensaje el header tendrá el siguiente formato:

Autorizacion: PP <LlaveID>:<MensajeFirmado>
Ejemplo:

* Autorizacion: PP 0PN5J17HBGZHT7ZZ3X82:fU6+JLYWzOSGuo76XJzT/Z596Qg=

El comercio deberá validar la firma del mensaje para corroborar el origen del mismo, si el
mensaje está firmado incorrectamente deberá devolver un error de autentificación.

Variables:
* token: Identificador único de la transacción en Punto Pagos
* trx_id: Identificador único de la transacción del cliente
* medio_pago: Identificador del medio de pago
* monto:Montototaldelatransacción
* fecha_aprobacion:Fechadeaprobacióndelatransacción(Formato:yyyy-MM-ddTHH:mm:ss)
* numero_tarjeta: 4 últimos dígitos de la tarjeta (opcional)
* num_cuotas: número de cuotas (opcional)
* tipo_cuotas: tipo de cuotas (opcional)
* valor_cuota: valor de cada cuota (opcional)
* primer_vencimiento:primervencimiento(Formato:yyyy-MM-dd) (opcional)
* numero_operacion: Número de operación en la institución financiera (opcional)
* codigo_autorizacion: Código de autorización de la transacción

Ejemplo json:

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

En el caso que no se cuente con un certificado SSL valido y la url de notificación del comercio sea solamente http, el servidor no envía los datos de la notificacion completos, sino que hace una llamada a la url informando el token. El comercio debera, ya con el token en su poder, realizar otra llamada a nuestra api (como se explica en la descripcion de ``transaccion/traer``), esta vez si, conectándose a nuestra api via SSL de manera segura.

```
Función: https://url_notificacion<token> (lado del comercio)
Método: GET
```

### Paso 6

Luego de procesado el pago y en caso de que sea exitoso, Punto Pagos procederá a redireccionar al cliente hacia la página del comercio.

```
Función: https://url_exito<token> (lado del comercio)
Método: GET
```

Ejemplo:

Url de exito del comercio: ``https://micomercio.com/transacciones/exito/``
Redireccion en caso de exito: ``https://micomercio.com/transacciones/exito/9XJ08401WN0071839``

Antes de dar la transacción por aprobada, el comercio deberá verificar que la notificación de dicha transacción haya llegado correctamente a través del token, en caso de que la notificación no haya sido recibida, el comercio podrá verificar el estado del pago con la función: https://servidor/transaccion/<token> y mostrar los comprobantes correspondientes.

En el caso que no se haya podido procesar el pago, Punto Pagos redireccionará hacia el url de fracaso donde el cliente mostrara una pantalla en donde se comunica al cliente que el pago de la transacción no ha sido completado con éxito.

```
Función: https://url_fracaso<token>
Método: GET
```

Ejemplo:

Url de exito del comercio: ``https://micomercio.com/transacciones/fracaso/``
Redireccion en caso de exito: ``https://micomercio.com/transacciones/exito/9XJ08401WN0071839``

### Obtener el estado de una transaccion

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
