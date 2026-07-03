# Evaluación Práctica UiPath - Marco Ricaurte

Solución RPA para el reto de conversión de monedas. Enunciado completo en el PDF adjunto en este repositorio.

## Robots

- **EvaluacionPractica-RicaurteMarco-CargaTransacciones**: descarga y valida `BaseCambios.xlsx`, y carga los registros válidos a la cola `COL_CONVERSIONES`.
- **EvaluacionPractica-RicaurteMarco-ConvertirMonedas**: procesa la cola, obtiene la tasa de cambio real desde [exchange-rates.org](https://www.exchange-rates.org/converter), valida cada conversión, genera `ReporteFinal.xlsx` y envía el resultado por correo al finalizar.

## Validaciones

- **CargaTransacciones**: cada fila se valida contra los campos obligatorios (`IdTransaccion`, `MonedaOrigen`, `ValorOrigen`, `MonedaDestino`, `ValorDestino`). Los registros incompletos se registran en un Log de tipo Error y no se encolan.
- **ConvertirMonedas**: por cada transacción se calcula `ValorDestino - ValorExtraido`. Si el resultado está entre -1 y 1, la conversión se marca como **Correcto**; si excede ese rango, se marca como **Fallido**.

## Envío de correo

Al finalizar el procesamiento de la cola, el robot calcula los totales (procesadas/correctas/fallidas), descarga la plantilla `PlantillaCorreo.html` desde el Storage Bucket, y envía el correo con `ReporteFinal.xlsx` adjunto mediante la integración nativa de Gmail (interfaz conectada de UiPath), tal como se especifica en el PDF, sin credenciales ni llamadas a API externas.

**Prueba de recepción:** validado con envío real a la cuenta configurada en el asset `StrCorreoUsuarios`, con el reporte correctamente adjunto y la plantilla renderizada con los totales de la ejecución.
![Correo recibido](Evidencia/correo-recibido.png)

## Configuración en Orchestrator

| Recurso | Nombre |
|---|---|
| Queue | `COL_CONVERSIONES` |
| Storage Bucket | `Requerimientos` (contiene `PlantillaCorreo.html`) |
| Asset | `StrCorreoUsuarios` |
| Asset | `StrUrlExchange` |

## Requisitos

UiPath Studio + paquetes `UiPath.Excel.Activities`, `UiPath.System.Activities`, `UiPath.UIAutomation.Activities`.
