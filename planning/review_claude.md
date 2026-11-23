# Revisión de PLAN.md

Revisión de `planning/PLAN.md` contrastada con las decisiones vigentes de `CLAUDE.md`. Las decisiones ya resueltas no se reabren.

## Incoherencias y huecos a resolver

1. **Estado de la watchlist ante tickers en minúsculas.** §6 dice que solo se aceptan de 1 a 5 letras mayúsculas (`422` si no). No queda claro si `aapl` se rechaza o se normaliza a `AAPL`. Propuesta: normalizar a mayúsculas antes de validar, para que el usuario no reciba un error por escribir en minúsculas.

2. **Añadir un ticker que ya está en la watchlist.** §8 no define el código de respuesta (no aparece `409` en §8). Propuesta: `409`, o bien `200` idempotente. Hay que elegir una.

3. **Vender la posición completa.** §7 no define qué pasa con `avg_cost` ni con la fila de `positions` cuando la cantidad llega a 0. Propuesta: borrar la fila al llegar a 0 y reiniciar `avg_cost` al comprar de nuevo.

4. **Precio de ejecución fuera de mercado.** §2 dice "ejecución instantánea al precio actual", pero con Massive el precio solo cambia en cada sondeo y con el mercado cerrado puede quedar obsoleto. Falta indicar qué precio se usa (último disponible en caché) y si se rechaza la operación si el dato es antiguo.

5. **Validación de operaciones sugeridas por el LLM.** §9 indica que cada operación pasa por la validación manual, pero no dice qué pasa si el LLM propone un ticker que no está en la caché de precios ni en Massive, o una cantidad no positiva. Propuesta: la operación falla con `error` en `actions` y el resto se ejecuta (coherente con la decisión de operaciones independientes).

6. **Modo `LLM_MOCK`.** §9 no define el comportamiento si el mensaje contiene a la vez "comprar" y "vender". Propuesta: prioridad fija (por ejemplo, "vender" gana) y documentarla.

7. **Cantidades fraccionarias.** §7 admite cantidades fraccionarias, pero no fija precisión ni mínimo. Con REAL en SQLite pueden aparecer errores de redondeo en el efectivo. Propuesta: fijar la precisión de cálculo (por ejemplo, redondear el importe a 2 decimales) y documentarlo.

8. **Historial de `portfolio_snapshots`.** §8 define `GET /api/portfolio/history` sin parámetros. Con un snapshot cada 30 s, la tabla crece sin límite (unas 2.880 filas al día). Propuesta: parámetro de rango o límite de filas en la respuesta.

9. **Errores de Massive distintos de `404`.** §6 solo define el `404`. No se contemplan límite de peticiones (`429`), caída del servicio o respuesta vacía en el sondeo. Propuesta: mantener la última caché válida y marcar el estado de conexión como amarillo o rojo.

10. **Coherencia del indicador de conexión.** §2 define verde/amarillo/rojo, pero §6 no indica qué evento pone el estado en amarillo (reconexión del SSE) frente a rojo (sin datos durante N segundos). Conviene definir el criterio.

## Simplificaciones

- **Dockerfile.** La etapa Node usa `npm install`. Para builds reproducibles, `npm ci` con el lockfile.
- **§13 pendiente de Massive.** Si el plan de pago no es necesario en la primera versión, basta con documentar el sondeo de un ticker por llamada con el límite de 5 llamadas/min, o con el snapshot multi-ticker. Decidir antes de implementar para no escribir dos caminos de código.
- **Doble definición de la validación de tickers.** §6 (simulador) y §8 (`422`) la describen por separado. Basta con una regla única en la capa de API que se aplique a ambas fuentes.

## Preguntas abiertas

- ¿Se debe permitir `POST /api/portfolio/trade` con `side` distinto de `buy`/`sell` (por ejemplo, mayúsculas)? Se propone rechazar con `422`.
- ¿El P&L de la tabla de posiciones usa `previous_price` (cambio diario) o el precio de apertura de la sesión? §10 pide "% de cambio diario" sin definir la referencia.
- ¿Qué pasa con los sparklines al recargar la página? §2 dice que se acumulan desde la carga; confirmar que no se reconstruyen desde el servidor (coherente con el texto actual).

## Riesgos

- **Clave de OpenRouter.** El backend escucha en el puerto 8000 sin autenticación. Al ser una aplicación local está bien, pero conviene dejar explícito que el contenedor debe publicarse solo en `localhost` (`-p 127.0.0.1:8000:8000`), para no exponer el chat (que ejecuta operaciones) en la red local.
- **Ejecución automática de operaciones del LLM.** Es una decisión vigente y no se reabre, pero el prompt de sistema debe limitar el número de operaciones por mensaje para evitar que una respuesta malformada ejecute cantidades grandes.
