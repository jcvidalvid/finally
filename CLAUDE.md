# Proyecto FinAlly - El Aliado Financiero

Toda la documentación del proyecto se encuentra en el directorio `planning`.

El documento clave es `planning/PLAN.md`, incluido en su totalidad a continuación. 

@planning/PLAN.md

## Decisiones vigentes

Resueltas en la revisión del plan. No reabrirlas sin consultar al usuario:

- Node 22 LTS para el build del frontend.
- Recharts para todos los gráficos (línea, sparklines y treemap).
- Sin columnas `user_id`: sistema de usuario único.
- Base de datos inicializada al arrancar el backend (no lazy).
- Operaciones del LLM independientes: si una falla, las demás se ejecutan.
- Tickers inválidos: `422` si el formato es incorrecto, `404` si no existe en Massive.
- Sin confirmación para operaciones ni cambios de watchlist.
- Modo `LLM_MOCK`: "comprar" compra 1 AAPL, "vender" vende 1 AAPL, resto texto fijo. Si el mensaje contiene ambas palabras, gana "vender".
- Seed de tickers nuevos en el simulador: uniforme entre 50 y 300 USD.
- Massive: sondeo con snapshot por lotes; validación de altas con `GET /v3/reference/tickers/{ticker}` (una llamada por alta o compra sin precio en caché).
- LLM: máximo 5 operaciones y 5 cambios de watchlist por respuesta.
- Gráfico de cartera etiquetado "valor de cartera"; el P&L realizado queda fuera de la primera versión.
- Docker: puerto publicado solo en `127.0.0.1`.
