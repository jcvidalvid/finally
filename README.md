# FinAlly - El Aliado Financiero

Estación de trading con IA: precios en vivo, cartera simulada con $10.000 virtuales y un asistente de chat que analiza la cartera y ejecuta operaciones en lenguaje natural.

## Estado

En desarrollo. Implementado: módulo de datos de mercado del backend (simulador GBM y cliente de Massive, caché de precios y tests). Pendiente: cartera, watchlist, API REST, streaming SSE, chat con LLM, frontend y Docker. Especificación completa en [`planning/PLAN.md`](planning/PLAN.md).

## Stack

- **Backend:** FastAPI (Python 3.12) gestionado con `uv`, SQLite.
- **Frontend:** Next.js (TypeScript) con exportación estática, Tailwind CSS y Recharts.
- **Tiempo real:** Server-Sent Events.
- **IA:** LiteLLM a través de OpenRouter (Cerebras como proveedor), con salidas estructuradas.

## Estructura

```
backend/    API y lógica de negocio (proyecto uv)
frontend/   Aplicación Next.js
planning/   Documentación y especificación para los agentes
scripts/    Scripts de arranque y parada (previstos)
test/       Tests E2E con Playwright (previstos)
db/         Volumen con la base de datos SQLite en tiempo de ejecución
```

## Configuración

Copia `.env` en la raíz y define:

| Variable | Descripción |
|---|---|
| `OPENROUTER_API_KEY` | Clave de OpenRouter para el chat. Obligatoria salvo con `LLM_MOCK=true`. |
| `MASSIVE_API_KEY` | Opcional. Si está vacía se usa el simulador de mercado. |
| `LLM_MOCK` | `true` para respuestas deterministas del LLM (tests, sin clave). |

## Desarrollo

Tests del backend:

```bash
cd backend
uv run pytest
```

Arranque con Docker (previsto, según el plan):

```bash
docker run -v $(pwd)/db:/app/db -p 127.0.0.1:8000:8000 --env-file .env finally
```

La aplicación se sirve en `http://localhost:8000`. El puerto se publica solo en `127.0.0.1` porque el chat ejecuta operaciones sin autenticación.
