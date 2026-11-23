# Revisión del plan FinAlly

**Fecha:** 2026-11-21  
**Documento revisado:** `planning/PLAN.md` (el archivo existente; `planning/plan.md` no existe con ese casing)  
**Alcance:** revisión de coherencia, vacíos de especificación y riesgos de implementación. No se reabren las decisiones vigentes enumeradas en `CLAUDE.md`.

## Resumen

El plan describe bien el flujo principal y separa con claridad frontend, backend, datos de mercado y persistencia. Antes de implementar el resto, conviene fijar algunos contratos que afectan a la corrección de la cartera y a lo que el usuario verá como P&L. El endpoint de Massive y el enrutamiento a Cerebras ya figuran como pendientes; deben seguir tratándose como dependencias explícitas de esas integraciones.

## Hallazgos

### 1. El gráfico llamado P&L representa valor total, no ganancias — prioridad alta

En §§7 y 10, `portfolio_snapshots` guarda `total_value` y el gráfico se describe como P&L. El valor total incluye el efectivo y el valor de mercado de las posiciones; por sí solo no es una ganancia o pérdida. Además, no se define el valor base contra el que calcular la variación.

**Recomendación:** definir el gráfico como «valor de cartera» o guardar/calcular también una serie de P&L con una base explícita. Documentar cómo se calcula P&L no realizado (valor de mercado menos coste), qué pasa al cerrar una posición y cómo se trata el P&L realizado. Mantener el P&L realizado fuera de la primera versión es compatible con el plan, pero el nombre y las etiquetas de la interfaz deben reflejarlo.

### 2. La matemática de operaciones necesita reglas de consistencia — prioridad alta

El saldo y las cantidades se almacenan como `REAL`, mientras que compras, ventas, actualizaciones de posición y escritura del trade modifican varias filas relacionadas. No se especifica cómo se evita que dos operaciones concurrentes gasten el mismo efectivo, por ejemplo una operación manual y otra emitida por el chat.

**Recomendación:** exigir que cada operación valide y actualice saldo/posición y registre el trade dentro de una transacción SQLite atómica, con serialización adecuada para operaciones concurrentes. Definir precisión/redondeo monetario, unidades de cantidad admitidas y el comportamiento al vender la posición completa (eliminar la fila o conservar cantidad cero). Añadir estos casos a los criterios de aceptación de cartera.

### 3. Faltan referencias para el cambio diario mostrado — prioridad media

La watchlist y la tabla de posiciones muestran un porcentaje de cambio diario, pero la caché solo define precio actual, precio anterior y timestamp. El precio anterior del último tick no equivale al cierre diario, especialmente con Massive y su sondeo cada 15 segundos.

**Recomendación:** especificar la fuente y la semántica del precio de referencia diario (por ejemplo, cierre anterior), cómo se obtiene para ambas fuentes y qué se muestra cuando no está disponible. Si no se implementará ese dato, cambiar el requisito a cambio desde el inicio de la sesión/local para que el indicador no sugiera una variación diaria real.

### 4. El ritmo SSE puede aparentar frescura que los datos no tienen — prioridad media

§6 propone enviar eventos cada ~500 ms, mientras que Massive actualiza los precios cada 15 segundos en el nivel gratuito. No queda claro si el servidor repetirá el último precio como si fuera una nueva actualización ni si `timestamp` refleja la observación original o la hora de emisión.

**Recomendación:** distinguir hora de observación y hora de emisión, conservar el timestamp de la fuente y definir si SSE publica solo cambios o también mensajes de heartbeat. La interfaz debe poder mostrar el retraso/frescura del dato sin animar un cambio de precio cuando el valor no ha cambiado.

### 5. Los contratos de respuesta de la API son insuficientes para frontend y agentes — prioridad media

§8 enumera rutas y errores, pero no fija esquemas para `/api/portfolio`, `/api/watchlist`, `/api/portfolio/history`, ni el formato común de errores. Tampoco aclara cómo obtiene el cliente un estado inicial coherente antes de recibir eventos SSE.

**Recomendación:** añadir ejemplos o modelos de respuesta con nombres, tipos, unidades y campos opcionales; fijar el formato de error; y describir la secuencia de carga inicial más SSE (incluido el caso de ticker sin precio en caché). Esto reduce interpretaciones distintas entre frontend y backend.

### 6. El formato del directorio descrito no coincide con el repositorio — prioridad baja

§4 documenta `frontend/`, pero en el árbol actual el directorio se llama `forntend/`. Además, `CLAUDE.md` incluye `planning/PLAN.md`, aunque la instrucción de usuario y el nombre en minúsculas se refieren a `plan.md`.

**Recomendación:** alinear el nombre real del frontend y las referencias de documentación, o registrar explícitamente que el nombre actual es intencional. En esta revisión se tomó `PLAN.md` como fuente porque es el archivo que existe y está referenciado por el repositorio.

## Pendientes que deben conservarse visibles

- El endpoint de snapshot de Massive sigue pendiente (§13) y condiciona el límite de llamadas/minuto y la cobertura de tickers. Cerrar la elección antes de dar por terminada la implementación Massive.
- El parámetro de LiteLLM/OpenRouter para seleccionar Cerebras también requiere verificación durante esa integración (§13).

## Pendientes de decisión (2026-11-23)

Abiertos tras la revisión cruzada con `reviewer` y `feature-dev:code-reviewer`. No se han aplicado en `PLAN.md`.

- **Despliegue en nube:** ¿queda fuera de alcance porque el chat ejecuta operaciones sin autenticación? El plan contempla App Runner/Render (§11), pero publicar sin protección expondría el chat (hallazgo 3 de este documento; hallazgo 20 del reviewer).
- **Protección local (CSRF / DNS rebinding):** `/api/portfolio/trade` y `/api/chat` aceptan peticiones de cualquier origen en `localhost:8000`. Pendiente decidir si se valida `Origin`/`Host` o se acepta el riesgo.
- **Simplificaciones propuestas:** sustituir Next.js por una SPA, aplazar la integración con Massive y validar solo el formato de ticker. Van contra decisiones ya tomadas en `CLAUDE.md` (Next.js, Recharts y validación por alta), así que no se aplican sin reabrirlas.
- **Historial de chat y migraciones:** `chat_messages` no tiene política de retención, y la inicialización de la base de datos no cubre cambios de esquema en versiones futuras (§7). Pendiente documentarlo o resolverlo ahora.

