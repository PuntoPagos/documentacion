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

Toda la información deberá ser transmitida encriptada con un certificado SSL bajo el protocolo https, en caso que el comercio no cuente con un certificado digital vigente y proporcionado por una empresa autorizada, se deberá solicitar la documentación con encriptación a través de llaves públicas.

Cada request realizado al servicio web debería ir firmado para comprobar la integridad y la autenticidad de la petición.

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

* trx_id: Identificador único de la transacción del cliente
* medio_pago: Identificador del medio de pago (existe servicio web para consultar todos los medios de pagos disponibles para el cliente)
* monto:Montototaldelatransacción
* detalle: Descripción del producto o servicio pagado (opcional)

Ejemplo json:

```json
{"trx_id":9787415132,"medio_pago":"999","monto":1000000.00}
```
