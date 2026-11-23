# FinAlly — AI Trading Workstation

## Especificación del Proyecto

## 1. Visión

FinAlly (Finance Ally) es una estación de trabajo de trading impulsada por IA, visualmente espectacular, que transmite datos de mercado en tiempo real, permite a los usuarios operar con una cartera simulada e integra un asistente de chat con LLM capaz de analizar posiciones y ejecutar operaciones en nombre del usuario. Tiene el aspecto y la sensación de una terminal Bloomberg moderna con un copiloto de IA.

Este es el proyecto final de un curso de programación con IA agéntica. Está construido íntegramente por Agentes de Programación, demostrando cómo unos agentes de IA orquestados pueden producir una aplicación full-stack de calidad de producción. Los agentes interactúan a través de archivos en `planning/`.

## 2. Experiencia de Usuario

### Primer Lanzamiento

El usuario ejecuta un único comando Docker (o un script de inicio proporcionado). El usuario abre `http://localhost:8000` en su navegador. Sin inicio de sesión, sin registro. Inmediatamente ve:

- Una watchlist de 10 tickers predeterminados con precios actualizándose en vivo en una cuadrícula
- $10,000 en efectivo virtual
- Una estética de terminal de trading oscura y rica en datos
- Un panel de chat de IA listo para ayudar

### Qué Puede Hacer el Usuario

- **Ver el flujo de precios en directo** — los precios destellan en verde (subida) o rojo (bajada) con sutiles animaciones CSS que se desvanecen
- **Ver mini-gráficos de tipo sparkline** — la evolución del precio junto a cada ticker en la watchlist, acumulada en el frontend a partir del stream SSE desde la carga de la página (los sparklines se completan progresivamente)
- **Hacer clic en un ticker** para ver un gráfico más grande y detallado en el área principal de gráficos
- **Comprar y vender acciones** — solo órdenes de mercado, ejecución instantánea al precio actual, sin comisiones, sin diálogo de confirmación
- **Monitorizar su cartera** — un mapa de calor (treemap) que muestra las posiciones dimensionadas por peso y coloreadas por P&L, además de un gráfico de valor de cartera que sigue el valor total (efectivo + posiciones) a lo largo del tiempo
- **Ver una tabla de posiciones** — ticker, cantidad, coste medio, precio actual, P&L no realizado, % de cambio
- **Chatear con el asistente de IA** — preguntar sobre su cartera, obtener análisis y hacer que la IA ejecute operaciones y gestione la watchlist mediante lenguaje natural
- **Gestionar la watchlist** — añadir/eliminar tickers manualmente o a través del chat de IA

### Diseño Visual

- **Tema oscuro**: fondos en torno a `#0d1117` o `#1a1a2e`, bordes grises apagados, sin negro puro
- **Animaciones de destello de precio**: breve resalte de fondo verde/rojo al cambiar el precio, desapareciendo en ~500ms mediante transiciones CSS
- **Indicador de estado de conexión**: un pequeño punto de color (verde = conectado, amarillo = reconectando, rojo = desconectado) visible en el encabezado
- **Diseño profesional y denso en datos**: inspirado en terminales Bloomberg/de trading — cada píxel cumple una función
- **Pensado primero para escritorio**: optimizado para pantallas anchas; no es objetivo dar soporte a tablet/móvil en esta fase

### Esquema de Colores
- Amarillo de Acento: `#ecad0a` (encabezados de sección y selección activa)
- Azul Primario: `#209dd7`
- Púrpura Secundario: `#753991` (botones de envío)
- Verde (subida de precio / beneficio): `#16c784`
- Rojo (bajada de precio / pérdida): `#ea3943`

## 3. Visión General de la Arquitectura

### Contenedor Único, Puerto Único

```
┌─────────────────────────────────────────────────┐
│  Docker Container (port 8000)                   │
│                                                 │
│  FastAPI (Python/uv)                            │
│  ├── /api/*          REST endpoints             │
│  ├── /api/stream/*   SSE streaming              │
│  └── /*              Static file serving         │
│                      (Next.js export)            │
│                                                 │
│  SQLite database (volume-mounted)               │
│  Background task: market data polling/sim        │
└─────────────────────────────────────────────────┘
```

- **Frontend**: Next.js con TypeScript, compilado como exportación estática (`output: 'export'`), servido por FastAPI como archivos estáticos
- **Backend**: FastAPI (Python), gestionado como un proyecto `uv`
- **Base de datos**: SQLite, un único archivo en `db/finally.db`, montado como volumen para persistencia
- **Datos en tiempo real**: Server-Sent Events (SSE) — más sencillo que WebSockets, envío unidireccional servidor→cliente, funciona en cualquier entorno
- **Integración de IA**: LiteLLM → OpenRouter (Cerebras para inferencia rápida), con salidas estructuradas para la ejecución de operaciones
- **Datos de mercado**: controlados por variables de entorno — simulador por defecto, datos reales mediante la API de Massive si se proporciona una clave

### Por Qué Estas Decisiones

| Decisión | Justificación |
|---|---|
| SSE en lugar de WebSockets | Solo necesitamos envío unidireccional; más simple, sin complejidad bidireccional, soporte universal en navegadores |
| Exportación estática de Next.js | Mismo origen, sin problemas de CORS, un solo puerto, un solo contenedor, despliegue simple |
| SQLite en lugar de Postgres | Sin autenticación = sin multiusuario = sin necesidad de un servidor de base de datos; autocontenido, configuración cero |
| Contenedor Docker único | Los estudiantes ejecutan un solo comando; sin docker-compose en producción, sin orquestación de servicios |
| uv para Python | Gestión de proyectos Python rápida y moderna; lockfile reproducible; lo que los estudiantes deben aprender |
| Solo órdenes de mercado | Elimina el libro de órdenes, la lógica de órdenes limitadas y las ejecuciones parciales — matemática de cartera drásticamente más simple |

---

## 4. Estructura de Directorios

```
finally/
├── frontend/                 # Proyecto Next.js TypeScript (exportación estática)
├── backend/                  # Proyecto FastAPI uv (Python)
│   └── schema/               # Definiciones de esquema SQL y datos semilla
├── planning/                 # Documentación del proyecto para los agentes
│   ├── PLAN.md               # Este documento
│   └── ...                   # Documentos de referencia adicionales para agentes
├── scripts/
│   ├── start_mac.sh          # Inicia el contenedor Docker (macOS/Linux)
│   ├── stop_mac.sh           # Detiene el contenedor Docker (macOS/Linux)
│   ├── start_windows.ps1     # Inicia el contenedor Docker (Windows PowerShell)
│   └── stop_windows.ps1      # Detiene el contenedor Docker (Windows PowerShell)
├── test/                     # Tests E2E con Playwright + docker-compose.test.yml
├── db/                       # Bind mount del volumen (el archivo SQLite vive aquí en tiempo de ejecución)
│   └── .gitkeep              # El directorio existe en el repo; finally.db está en .gitignore
├── Dockerfile                # Build multi-etapa (Node → Python)
├── .env                      # Variables de entorno (en .gitignore, se versiona .env.example)
└── .gitignore
```

### Límites Clave

- **`frontend/`** es un proyecto Next.js autocontenido. No sabe nada de Python. Se comunica con el backend a través de los endpoints `/api/*` y los endpoints SSE `/api/stream/*`. La estructura interna queda a criterio del agente Ingeniero de Frontend.
- **`backend/`** es un proyecto uv autocontenido con su propio `pyproject.toml`. Es responsable de toda la lógica del servidor, incluyendo la inicialización de la base de datos, el esquema, los datos semilla, las rutas de la API, el streaming SSE, los datos de mercado y la integración con el LLM. La estructura interna queda a criterio de los agentes de Backend/Datos de Mercado.
- **`backend/schema/`** contiene las definiciones SQL del esquema y la lógica de datos semilla. El backend inicializa la base de datos al arrancar (lifespan de FastAPI) — creando las tablas y poblando los datos predeterminados si el archivo SQLite no existe o está vacío.
- **`db/`** en el nivel superior es el punto de montaje del volumen en tiempo de ejecución. El archivo SQLite (`db/finally.db`) es creado aquí por el backend y persiste entre reinicios del contenedor gracias al volumen Docker.
- **`planning/`** contiene la documentación de todo el proyecto, incluido este plan. Todos los agentes usan los archivos de aquí como contrato compartido.
- **`test/`** contiene los tests E2E con Playwright e infraestructura de soporte (por ejemplo, `docker-compose.test.yml`). Los tests unitarios viven dentro de `frontend/` y `backend/` respectivamente, siguiendo las convenciones de cada framework.
- **`scripts/`** contiene los scripts de inicio/parada que envuelven comandos Docker.

---

## 5. Variables de Entorno

```bash
# Obligatoria salvo que LLM_MOCK=true: clave de API de OpenRouter para la funcionalidad de chat con LLM
OPENROUTER_API_KEY=your-openrouter-api-key-here

# Opcional: clave de API de Massive (Polygon.io) para datos de mercado reales
# Si no se establece, se usa el simulador de mercado integrado (recomendado para la mayoría de usuarios)
MASSIVE_API_KEY=

# Opcional: establecer en "true" para respuestas de LLM simuladas y deterministas (pruebas)
LLM_MOCK=false
```

### Comportamiento

- Si `MASSIVE_API_KEY` está establecida y no está vacía → el backend usa la API REST de Massive para los datos de mercado
- Si `MASSIVE_API_KEY` está ausente o vacía → el backend usa el simulador de mercado integrado
- Si `LLM_MOCK=true` → el backend devuelve respuestas de LLM simuladas y deterministas (para tests E2E), y no requiere `OPENROUTER_API_KEY`
- Si `LLM_MOCK=false` (o no está definida) → `OPENROUTER_API_KEY` es obligatoria para que el chat funcione
- El backend recibe las variables del `.env` de la raíz mediante `--env-file` de docker; el `.env` nunca entra en la imagen

---

## 6. Datos de Mercado

### Dos Implementaciones, Una Interfaz

Tanto el simulador como el cliente de Massive implementan la misma interfaz abstracta. El backend selecciona cuál usar en función de la variable de entorno. Todo el código posterior (streaming SSE, caché de precios, frontend) es agnóstico respecto a la fuente.

### Simulador (Predeterminado)

- Genera precios utilizando movimiento browniano geométrico (GBM) con deriva (drift) y volatilidad configurables por ticker
- Se actualiza en intervalos de ~500ms
- Movimientos correlacionados entre tickers (por ejemplo, las acciones tecnológicas se mueven juntas)
- "Eventos" aleatorios ocasionales — movimientos súbitos del 2-5% en un ticker para dar dramatismo
- Comienza desde precios semilla realistas (por ejemplo, AAPL ~$190, GOOGL ~$175, etc.)
- Cuando se añade un ticker nuevo (no predefinido) a la watchlist, el simulador le asigna un precio semilla aleatorio entre 50 y 300 USD y empieza a simularlo igual que a los demás
- Validación de símbolos: regla única de §8
- Se ejecuta como una tarea en segundo plano dentro del propio proceso — sin dependencias externas

### API de Massive (Opcional)

- Sondeo (polling) mediante API REST (no WebSocket) — más simple, funciona en todos los niveles de suscripción
- Sondea la unión de todos los tickers vigilados con una sola llamada: `GET /v2/snapshot/locale/us/markets/stocks/tickers?tickers=AAPL,MSFT,...`
- Cliente HTTP `httpx` directo contra `https://api.massive.com`; sin SDK `massive`
- Nivel gratuito (5 llamadas/min): sondeo cada 15 segundos. Las validaciones de existencia de §8 consumen la misma cuota
- Niveles de pago: sondeo cada 2-15 segundos según el nivel
- Parsea la respuesta REST al mismo formato que el simulador (`price = lastTrade.p`, `timestamp = lastTrade.t`)
- Validación de altas: ver la regla única de §8
- Ante `429`, `5xx` o respuesta vacía: se mantiene la última caché válida; el estado de conexión pasa a amarillo y a rojo tras 60 s sin datos válidos

### Caché de Precios Compartida

- Una única tarea en segundo plano (el simulador o el poller de Massive) escribe en una caché de precios en memoria
- La caché guarda el último precio, el precio anterior y la marca de tiempo de cada ticker
- Los streams SSE leen de esta caché y envían actualizaciones a los clientes conectados
- Esta arquitectura permite futuros escenarios multiusuario sin cambios en la capa de datos

### Streaming SSE

- Endpoint: `GET /api/stream/prices`
- Conexión SSE de larga duración; el cliente usa la API nativa `EventSource`
- El servidor envía actualizaciones de precio a un ritmo regular (~500ms) para la **unión** de los tickers de la watchlist del usuario y los tickers con una posición abierta (aunque ya no estén en la watchlist) — así el P&L no realizado y el mapa de calor siempre tienen precios actualizados
- Cada evento SSE contiene `ticker`, `price`, `previous_price`, `timestamp` y `direction` (`up`, `down` o `flat`). Ejemplo:

```json
{"ticker": "AAPL", "price": 190.42, "previous_price": 190.10, "timestamp": "2026-11-20T10:00:00Z", "direction": "up"}
```

- `timestamp` es la hora de observación de la fuente (`lastTrade.t` en Massive; instante del tick en el simulador), no la de emisión
- Con Massive el precio cambia solo en cada ciclo de sondeo; el SSE emite un evento solo cuando el precio ha cambiado desde el ciclo anterior, y la UI no destella si el precio no cambió
- El % de cambio diario usa como referencia `prevDay.c` (Massive) o el precio de arranque del proceso (simulador); `previous_price` solo sirve para la dirección del destello
- Al conectar o reconectar, el servidor envía primero el último precio conocido de cada ticker de la unión, aunque no haya cambiado; así la watchlist no queda sin precio tras una reconexión
- El cliente gestiona la reconexión automáticamente (EventSource tiene reintento incorporado)

---

## 7. Base de Datos

### SQLite con Inicialización al Arrancar

El backend comprueba la existencia de la base de datos SQLite al arrancar. Si el archivo no existe o faltan tablas, crea el esquema y puebla los datos predeterminados. Esto significa:

- Sin paso de migración independiente
- Sin configuración manual de la base de datos
- Los volúmenes Docker nuevos arrancan automáticamente con una base de datos limpia y poblada

### Esquema

El sistema es de usuario único. No hay columnas `user_id`; si en el futuro se necesita multiusuario, se añadirán mediante una migración.

Reglas de consistencia:
- Importes de efectivo redondeados a 2 decimales; cantidades con hasta 6 decimales (más de 6 → `422`) y `quantity > 0`
- Cada operación (manual o del LLM) valida y escribe saldo, posición y trade en una única transacción SQLite, serializada con un `asyncio.Lock` compartido que cubre solo esa escritura (nunca la llamada al LLM)
- Las operaciones se ejecutan al último precio en caché. Con mercado cerrado o Massive sin datos nuevos se usa ese último precio; no se rechaza la operación. Si la fuente lleva más de 60 s sin datos válidos, la UI marca el precio como desactualizado (ver §10)
- El importe del trade (`quantity × price`) y el cargo en efectivo se redondean a 2 decimales; `avg_cost` se guarda sin redondear
- El P&L no realizado no se almacena: se calcula como `(precio actual − avg_cost) × quantity`

Las tablas `watchlist` y `positions` usan el ticker como clave primaria, ya que es único por definición en este sistema.

**users_profile** — Estado del usuario (saldo de efectivo)
- `id` TEXT PRIMARY KEY (predeterminado: `"default"`)
- `cash_balance` REAL (predeterminado: `10000.0`)
- `created_at` TEXT (marca de tiempo ISO)

**watchlist** — Tickers que el usuario está vigilando
- `ticker` TEXT PRIMARY KEY
- `added_at` TEXT (marca de tiempo ISO)

**positions** — Posiciones actuales (una fila por ticker)
- `ticker` TEXT PRIMARY KEY
- `quantity` REAL (se admiten acciones fraccionarias)
- `avg_cost` REAL (coste medio ponderado; en las ventas no cambia; al vender hasta cantidad 0 se elimina la fila, y una nueva compra empieza con `avg_cost` nuevo)
- `updated_at` TEXT (marca de tiempo ISO)

**trades** — Historial de operaciones (registro de solo adición)
- `id` TEXT PRIMARY KEY (UUID)
- `ticker` TEXT
- `side` TEXT (`"buy"` o `"sell"`)
- `quantity` REAL (se admiten acciones fraccionarias)
- `price` REAL
- `executed_at` TEXT (marca de tiempo ISO)

**portfolio_snapshots** — Valor de la cartera a lo largo del tiempo (para el gráfico de valor de cartera). Se registra cada 30 segundos mediante una tarea en segundo plano, e inmediatamente después de cada ejecución de operación.
- `id` TEXT PRIMARY KEY (UUID)
- `total_value` REAL
- `recorded_at` TEXT (marca de tiempo ISO)

**chat_messages** — Historial de conversación con el LLM
- `id` TEXT PRIMARY KEY (UUID)
- `role` TEXT (`"user"` o `"assistant"`)
- `content` TEXT
- `actions` TEXT (JSON — operaciones ejecutadas, cambios realizados en la watchlist; null para mensajes del usuario)
- `created_at` TEXT (marca de tiempo ISO)

### Datos Semilla Predeterminados

- Un perfil de usuario: `id="default"`, `cash_balance=10000.0`
- Diez entradas en la watchlist: AAPL, GOOGL, MSFT, AMZN, TSLA, NVDA, META, JPM, V, NFLX

---

## 8. Endpoints de la API

### Datos de Mercado
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/stream/prices` | Stream SSE de actualizaciones de precios en vivo |

### Cartera
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/portfolio` | Posiciones actuales, saldo de efectivo, valor total, P&L no realizado |
| POST | `/api/portfolio/trade` | Ejecuta una operación: `{ticker, quantity, side}` |
| GET | `/api/portfolio/history` | Capturas del valor de la cartera (para el gráfico de valor de cartera). Query `?limit=N` (por defecto 1000, las más recientes, en orden cronológico; fuera de 1–1000 → `422`) |

### Watchlist
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/watchlist` | Tickers actuales de la watchlist con sus últimos precios |
| POST | `/api/watchlist` | Añade un ticker: `{ticker}`. `409` si ya está en la watchlist |
| DELETE | `/api/watchlist/{ticker}` | Elimina un ticker |

### Chat
| Método | Ruta | Descripción |
|--------|------|-------------|
| POST | `/api/chat` | Envía un mensaje, recibe una respuesta JSON completa (mensaje + acciones ejecutadas) |

### Sistema
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/health` | Comprobación de estado (para Docker/despliegue). Respuesta `200` con `{"status": "ok"}` |

### Códigos de Error

- `422`: datos inválidos (formato de ticker incorrecto, cantidad no positiva)
- `404`: ticker no encontrado (en Massive) o no presente en la watchlist al eliminarlo
- `400`: saldo insuficiente para comprar, o acciones insuficientes para vender
- `409`: el ticker ya está en la watchlist al añadirlo
- `502`: el LLM devolvió JSON inválido tras el reintento (solo `/api/chat`)
- `503`: la validación de existencia en Massive recibió `429` (cuota agotada); la alta o compra no se ejecuta y `/api/chat` la reporta como `status: "error"`

Reglas comunes:
- Orden de validación de una operación: formato (`422`) → existencia (`404`) → saldo o acciones (`400`)
- Los tickers se normalizan a mayúsculas (`aapl` → `AAPL`) en la capa de API antes de validar
- Validación de formato: 1 a 5 letras (`A-Z`), con sufijo opcional de clase `.X` (una letra, p. ej. `BRK.B`); si no cumple, `422`. `side` distinto de `buy`/`sell` también es `422`
- Validación de existencia (altas y compras, nunca ventas), igual para simulador y Massive: un ticker sin precio en caché se comprueba con `GET /v3/reference/tickers/{ticker}` (una llamada por alta o compra, nunca en el ciclo de sondeo); si no existe en Massive, `404`. Con el simulador no hay llamada externa: cualquier ticker con formato válido recibe precio semilla (50–300 USD). Con Massive, el resultado de cada validación (existente o no) se cachea en memoria por ticker para no repetir la llamada
- Formato de error: `{"detail": "..."}` (respuesta por defecto de FastAPI)

### Respuesta de `/api/chat`

```json
{
  "message": "Compra ejecutada.",
  "trades": [
    {"ticker": "AAPL", "side": "buy", "quantity": 1, "status": "executed", "price": 190.42}
  ],
  "watchlist_changes": [
    {"ticker": "PYPL", "action": "add", "status": "error", "error": "Ticker no válido"}
  ]
}
```

Cada acción tiene `status` (`"executed"` o `"error"`); las que tienen error incluyen `error`. El mismo contenido se guarda en `chat_messages.actions`.

---

## 9. Integración con el LLM

Al escribir código que realice llamadas a LLMs, usa la skill cerebras-inference para utilizar LiteLLM a través de OpenRouter hacia el modelo `openrouter/openai/gpt-oss-120b`, con Cerebras como proveedor de inferencia. Se deben usar Salidas Estructuradas (Structured Outputs) para interpretar los resultados.

Existe una OPENROUTER_API_KEY en el archivo .env en la raíz del proyecto.

### Cómo Funciona

Cuando el usuario envía un mensaje de chat, el backend:

1. Carga el contexto actual de la cartera del usuario (efectivo, posiciones con P&L, watchlist con precios en vivo, valor total de la cartera)
2. Carga el historial de conversación reciente desde la tabla `chat_messages` (máximo los últimos 20 mensajes)
3. Construye un prompt con un mensaje de sistema, el contexto de la cartera, el historial de conversación y el nuevo mensaje del usuario
4. Llama al LLM a través de LiteLLM → OpenRouter, solicitando salida estructurada, usando la skill cerebras-inference
5. Parsea la respuesta JSON estructurada. Si el JSON no es válido, reintenta la llamada una vez; si vuelve a fallar, responde `502` con `{"detail": "..."}`, no ejecuta ninguna operación y el frontend muestra el error en el chat
6. Ejecuta automáticamente cualquier operación o cambio de watchlist especificado en la respuesta
7. Almacena el mensaje y las acciones ejecutadas en `chat_messages`
8. Devuelve la respuesta JSON completa al frontend (sin streaming token a token — la inferencia de Cerebras es suficientemente rápida como para que un indicador de carga sea suficiente)

### Esquema de Salida Estructurada

Se instruye al LLM para que responda con un JSON que coincida con este esquema:

```json
{
  "message": "Your conversational response to the user",
  "trades": [
    {"ticker": "AAPL", "side": "buy", "quantity": 10}
  ],
  "watchlist_changes": [
    {"ticker": "PYPL", "action": "add"},
    {"ticker": "NFLX", "action": "remove"}
  ]
}
```

- `message` (obligatorio): el texto conversacional mostrado al usuario
- `trades` (siempre presente, `[]` si no hay): array de operaciones a ejecutar automáticamente. Cada operación pasa por la misma validación que las operaciones manuales (efectivo suficiente para compras, acciones suficientes para ventas)
- `watchlist_changes` (siempre presente, `[]` si no hay): array de modificaciones de la watchlist. `action` admite `"add"` o `"remove"`. Se procesan primero las `watchlist_changes` y después los `trades`, así un ticker añadido puede comprarse en la misma respuesta. Añadir un ticker ya existente o quitar uno ausente produce `status: "error"` con el mismo texto que los códigos `409` y `404` de §8

### Ejecución Automática

Las operaciones especificadas por el LLM se ejecutan automáticamente — sin diálogo de confirmación. Esta es una decisión de diseño deliberada:
- Es un entorno simulado con dinero ficticio, por lo que el riesgo es nulo
- Crea una experiencia de demostración impresionante y fluida
- Demuestra capacidades de IA agéntica — el tema central del curso

Las operaciones son independientes: si una falla (por ejemplo, efectivo insuficiente), las demás se ejecutan igualmente. El error de cada operación fallida se incluye en la respuesta del chat para que el LLM pueda informar al usuario. Los cambios de watchlist siguen el mismo criterio.

Límite por respuesta: máximo 5 operaciones y 5 cambios de watchlist por mensaje. Lo que exceda se descarta y se registra como `status: "error"` en `actions`. Un ticker o cantidad no válidos (incluida una cantidad no positiva) produce error en esa acción, no en toda la respuesta.

### Orientación del Prompt de Sistema

Se debe indicar al LLM que actúe como "FinAlly, un asistente de trading con IA", con instrucciones para:
- Analizar la composición de la cartera, la concentración de riesgo y el P&L
- Sugerir operaciones con su razonamiento
- Ejecutar solo las operaciones pedidas explícitamente en el mensaje actual; nunca ejecutar propuestas de mensajes anteriores
- Gestionar la watchlist de forma proactiva
- Ser conciso y basarse en datos en sus respuestas
- Responder siempre con un JSON estructurado válido

### Modo Simulado del LLM

Cuando `LLM_MOCK=true`, el backend devuelve respuestas simuladas deterministas en lugar de llamar a OpenRouter:
- Si el mensaje contiene "vender", propone vender 1 AAPL (si no hay posición, la operación falla con error, lo que permite probar el camino de error)
- Si el mensaje contiene "comprar", propone comprar 1 AAPL
- Si contiene ambas palabras, gana "vender"
- En cualquier otro caso, devuelve un texto fijo sin operaciones

Esto permite:
- Tests E2E rápidos, gratuitos y reproducibles
- Desarrollo sin una clave de API
- Pipelines de CI/CD

---

## 10. Diseño del Frontend

### Disposición (Layout)

El frontend es una aplicación de una sola página con una disposición densa, inspirada en una terminal. La arquitectura de componentes específica y el sistema de disposición quedan a criterio del Ingeniero de Frontend, pero la interfaz debe incluir estos elementos:

- **Panel de watchlist** — cuadrícula/tabla de tickers vigilados con: símbolo del ticker, precio actual (destellando en verde/rojo al cambiar; marcado como desactualizado si la fuente lleva más de 60 s sin datos válidos), % de cambio diario, y un mini-gráfico tipo sparkline (acumulado desde el SSE desde la carga de la página)
- **Área principal de gráficos** — gráfico más grande para el ticker actualmente seleccionado, mostrando como mínimo el precio a lo largo del tiempo. Al hacer clic en un ticker de la watchlist se selecciona aquí.
- **Mapa de calor de la cartera** — visualización treemap donde cada rectángulo es una posición, dimensionado por peso en la cartera y coloreado por P&L (verde = beneficio, rojo = pérdida)
- **Gráfico de valor de cartera** — gráfico de líneas que muestra el valor total de la cartera a lo largo del tiempo, usando datos de `portfolio_snapshots`
- **Tabla de posiciones** — vista tabular de todas las posiciones: ticker, cantidad, coste medio, precio actual, P&L no realizado, % de cambio
- **Barra de operaciones** — área de entrada sencilla: campo de ticker, campo de cantidad, botón de compra, botón de venta. Órdenes de mercado, ejecución instantánea.
- **Panel de chat de IA** — barra lateral acoplada/colapsable. Entrada de mensajes, historial de conversación con desplazamiento, indicador de carga mientras se espera la respuesta del LLM. Las ejecuciones de operaciones y los cambios de watchlist se muestran en línea como confirmaciones.
- **Encabezado** — valor total de la cartera (actualizándose en vivo), indicador de estado de conexión, saldo de efectivo. El valor total es efectivo + suma de (cantidad × precio actual) de cada posición.

### Notas Técnicas

- Usar `EventSource` para la conexión SSE a `/api/stream/prices`
- Recharts para todos los gráficos (línea de precio, sparklines y treemap), una sola dependencia de gráficos
- Efecto de destello de precio: al recibir un nuevo precio, aplicar brevemente una clase CSS con transición de color de fondo, y luego eliminarla
- Todas las llamadas a la API van al mismo origen (`/api/*`) — no se necesita configuración de CORS
- Tailwind CSS para los estilos, con un tema oscuro personalizado

---

## 11. Docker y Despliegue

### Dockerfile Multi-Etapa

```
Stage 1: Node 22 LTS slim
  - Copy frontend/
  - npm ci && npm run build (produces static export; usa el lockfile)

Stage 2: Python 3.12 slim
  - Install uv
  - Copy backend/
  - uv sync (install Python dependencies from lockfile)
  - Copy frontend build output into a static/ directory
  - Expose port 8000
  - CMD: uvicorn --host 0.0.0.0 --port 8000 (dentro del contenedor; el bind a 127.0.0.1 se aplica en `docker run`)
```

FastAPI sirve los archivos estáticos del frontend y todas las rutas de la API en el puerto 8000.

### Volumen Docker

La base de datos SQLite persiste mediante un bind mount del directorio `db/` de la raíz del proyecto:

```bash
docker run -v $(pwd)/db:/app/db -p 127.0.0.1:8000:8000 --env-file .env finally
```

El puerto se publica solo en `127.0.0.1`: el chat ejecuta operaciones sin autenticación y no debe quedar expuesto en la red local. El directorio `db/` en la raíz del proyecto se mapea a `/app/db` en el contenedor.

`.dockerignore` excluye `.env`, `.env.*`, `.git`, `.venv/`, `*.db`, `db/`, `node_modules/`, `.claude/`, `planning/` y `test/`, para que ningún secreto acabe en la imagen. El backend escribe `finally.db` en esta ruta, por lo que el archivo es directamente inspeccionable/respaldable desde el host.

### Scripts de Inicio/Parada

**`scripts/start_mac.sh`** (macOS/Linux):
- Construye la imagen Docker si aún no está construida (o si se pasa el flag `--build`)
- Ejecuta el contenedor con el montaje del volumen, el mapeo de puertos y el archivo `.env`
- Imprime la URL para acceder a la aplicación
- Abre el navegador solo con el flag `--open`

**`scripts/stop_mac.sh`** (macOS/Linux):
- Detiene y elimina el contenedor en ejecución
- NO elimina el volumen (los datos persisten)

**`scripts/start_windows.ps1`** / **`scripts/stop_windows.ps1`**: equivalentes en PowerShell para Windows.

Todos los scripts deben ser idempotentes — seguros de ejecutar varias veces:
- Si el contenedor ya está en ejecución, el script de inicio no hace nada y muestra la URL
- Si la imagen ya existe, no se reconstruye salvo con `--build`
- Si el contenedor no está en ejecución, el script de parada no hace nada y termina sin error

### Despliegue en la Nube (Opcional)

El contenedor está diseñado para desplegarse en AWS App Runner, Render o cualquier plataforma de contenedores. Fuera de alcance de la construcción principal: una configuración de Terraform para App Runner en `deploy/` (stretch goal, no requerida).

---

## 12. Estrategia de Pruebas

### Tests Unitarios (dentro de `frontend/` y `backend/`)

**Backend (pytest)**:
- Datos de mercado: el simulador genera precios válidos, la matemática del GBM es correcta, el parseo de la respuesta de la API de Massive funciona, ambas implementaciones cumplen con la interfaz abstracta
- Cartera: lógica de ejecución de operaciones, cálculos de P&L, casos límite (vender más de lo que se posee, comprar con efectivo insuficiente, vender con pérdidas)
- LLM: el parseo de la salida estructurada maneja todos los esquemas válidos, manejo correcto de respuestas malformadas (reintento y error según §9), validación de operaciones dentro del flujo de chat
- Rutas de la API: códigos de estado correctos (incluidos `422`, `404` y `400` de §8), formas de respuesta, manejo de errores

**Frontend (React Testing Library o similar)**:
- Renderizado de componentes con datos simulados
- La animación de destello de precio se activa correctamente ante cambios de precio
- Operaciones CRUD de la watchlist
- Cálculos de visualización de la cartera
- Renderizado de mensajes de chat y estado de carga

### Tests E2E (en `test/`)

**Infraestructura**: un `docker-compose.test.yml` independiente en `test/` que levanta el contenedor de la aplicación junto con un contenedor de Playwright. Esto mantiene las dependencias del navegador fuera de la imagen de producción.

**Entorno**: los tests E2E fijan `LLM_MOCK=true` en su entorno, para mayor velocidad y determinismo. El valor por defecto de la aplicación sigue siendo `false`.

**Escenarios Clave**:
- Inicio limpio: aparece la watchlist predeterminada, se muestra el saldo de $10k, los precios se están transmitiendo
- Añadir y eliminar un ticker de la watchlist
- Comprar acciones: el efectivo disminuye, aparece la posición, la cartera se actualiza
- Vender acciones: el efectivo aumenta, la posición se actualiza o desaparece
- Visualización de la cartera: el mapa de calor se renderiza con los colores correctos, el gráfico de valor de cartera tiene puntos de datos
- Chat de IA (simulado): enviar un mensaje, recibir una respuesta, la ejecución de la operación aparece en línea
- Resiliencia del SSE: desconectar y verificar la reconexión

---

## 13. Pendientes

Resueltos en la revisión del 2026-11-21 (incorporados en el cuerpo del plan):
- **Snapshot multi-ticker de Massive (§6)**: `GET /v2/snapshot/locale/us/markets/stocks/tickers?tickers=...`, una llamada para toda la unión. Pendiente solo verificarlo contra la documentación vigente al implementar, porque no se consultó en esta revisión.
- **Enrutamiento Cerebras (§9)**: `extra_body={"provider": {"order": ["cerebras"]}}`, según la skill `cerebras-inference`. Pendiente al implementar: confirmar con la documentación de OpenRouter si hace falta `require_parameters` para garantizar `response_format`.

Abiertos:
- **P&L realizado**: no se muestra en la primera versión. Se puede derivar de `trades` si más adelante se quiere mostrar.
