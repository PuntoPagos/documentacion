PuntoPagos API
=============

## Perspectiva General

El manual ténico tiene como objetivo describir las funcionalidades y los parámetros necesarios de los servicios web proporcionados por Punto Pagos para el correcto funcionamiento de los botones de pago de nuestro sistema de recaudación electrónica.

En el caso particular de la documentación REST, proveerá los servicios REST disponibles tanto en formato JSON como XML, siendo el formato JSON el formato por defecto.

Este documento está dirigido a desarrolladores.

## Especificaciones Técnicas

Mediante nuestro kit de integración (API) el comercio puede acceder a todas las funcionalidades de PuntoPagos.com, el kit consta de una serie de métodos que pueden ser invocados a t ravés de nuestros servicios web.

El siguiente diagrama muestra a grandes rasgos como es la comunicación para una transacción de venta:

![Diagrama de llamadas](https://raw.githubusercontent.com/PuntoPagos/documentacion/master/img/flujo-llamadas.png "Diagrama de llamadas")

Se recomienda que toda la información sea transmitida encriptada con un certificado SSL bajo el protocolo https. El certificado debe ser válido y emitido por una entidad de certificacion autorizada (no puede ser autofirmado). La API también permite su utilización sin contar con un certificado SSL. Esta variante se explica en el paso 4, durante la notificacion al comercio.

Cada request realizado al servicio web debería ir firmado para comprobar la integridad y la autenticidad de la petición

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
* monto:Monto total de la transacción
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
* monto:Monto total de la transacción
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

Parametros:

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

## Anexos

### Códigos de los medios de pago

Código | Descripcion
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

### Codigos de error

Estos son los diferentes codigos de error que puede informar la API

Código | Descripcion
-------|------------------------
1      | Transaccion Rechazada
2      | Transaccion Anulada
6      | Transaccion Incompleta
7      | Error del financiador

### Datos de prueba para el modo sandbox

Numeros de tarjeta para WebPay

Tarjeta | Numero | CCV | Expiracion | Resultado Esperado
--------|--------|-----|------------|--------------------
Visa    | 4051885600446623 | 123 | cualquiera | Exito
Mastercard | 5186059559590568 | 123 | cualquiera |Fracaso 

Al pedir un RUT se debe ingresar ``11111111-1`` y la clave ``123``

