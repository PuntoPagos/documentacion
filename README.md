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