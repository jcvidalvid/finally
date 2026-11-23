## Hallazgos por severidad

1. **Alta — El contrato de acciones del chat permite ambigüedad sobre cuándo se ejecuta una operación.** En §9 se dice que el LLM ejecuta operaciones automáticamente y que las operaciones deben ejecutarse cuando el usuario las solicite o esté de acuerdo ([líneas 366–375](</C:/projectes/finally/planning/PLAN.md:366>), [líneas 377–385](</C:/projectes/finally/planning/PLAN.md:377>)). “Esté de acuerdo” puede referirse a una propuesta previa, pero el plan no define cómo se comprueba ese consentimiento ni qué contexto conserva el chat. Aclararía si solo se ejecutan acciones pedidas explícitamente en el mensaje actual, o también una aceptación de una propuesta anterior.

2. **Alta — No está definido cómo se determina el precio válido para ejecutar operaciones.** La operación usa el último precio en caché incluso con mercado cerrado o sin datos nuevos ([líneas 213–217](</C:/projectes/finally/planning/PLAN.md:213>)); para Massive, el precio puede quedar obsoleto y la API devuelve estado de conexión degradado ([líneas 165–172](</C:/projectes/finally/planning/PLAN.md:165>)). Falta definir qué antigüedad se considera aceptable y si una operación debe mostrar la marca temporal o avisar al usuario cuando el precio está desactualizado.

3. **Alta — La afirmación de que el chat sin autenticación no queda expuesto en la red depende del modo de ejecución.** El ejemplo limita el puerto a `127.0.0.1` ([líneas 451–455](</C:/projectes/finally/planning/PLAN.md:451>)), pero el plan también contempla despliegue en plataformas de contenedores ([línea 480](</C:/projectes/finally/planning/PLAN.md:480>)). En ese contexto puede estar accesible desde una red pública. Conviene marcar el despliegue en nube como no compatible con el chat de operaciones o definir protección de acceso para ese modo.

4. **Media — El alcance de “tiempo real” no coincide del todo con los intervalos especificados.** El simulador actualiza cada ~500 ms, Massive consulta cada 2–15 s o cada 15 s en el nivel gratuito, y la cartera toma snapshots cada 30 s ([líneas 152–169](</C:/projectes/finally/planning/PLAN.md:152>), [líneas 244–247](</C:/projectes/finally/planning/PLAN.md:244>)). Aclararía en la interfaz que la frecuencia depende de la fuente y que el gráfico de valor total tiene una resolución distinta al flujo de precios.

5. **Media — Hay una posible incoherencia entre la tabla de rutas y los flujos de datos descritos.** La watchlist se devuelve con los últimos precios ([líneas 277–282](</C:/projectes/finally/planning/PLAN.md:277>)), pero el endpoint SSE solo se describe como flujo de precios ([líneas 181–195](</C:/projectes/finally/planning/PLAN.md:181>)). No se explica cómo el cliente descubre que cambió la watchlist o cómo obtiene precio inicial para un ticker recién añadido. Especificaría si el cliente vuelve a consultar `/api/watchlist` después de cada cambio o si el SSE admite suscripciones dinámicas.

6. **Media — La inicialización de la base de datos no define claramente cómo maneja cambios de esquema.** Se indica crear tablas y sembrar datos si la base no existe o faltan tablas, y a la vez se descarta un paso de migración independiente ([líneas 201–207](</C:/projectes/finally/planning/PLAN.md:201>)). Si una versión futura cambia una tabla existente, “faltan tablas” no cubre esa actualización. Aclararía si el proyecto acepta reinicializar la base en desarrollo o si necesita migraciones versionadas desde el inicio.

7. **Baja — El historial de chat crece sin política de retención.** Se cargan como máximo 20 mensajes para el prompt, pero todos se almacenan en `chat_messages` ([líneas 249–254](</C:/projectes/finally/planning/PLAN.md:249>), [líneas 334–343](</C:/projectes/finally/planning/PLAN.md:334>)). Para una app local de demostración puede bastar con no resolverlo ahora; si se mantiene el historial indefinidamente, conviene documentarlo.

## Simplificaciones posibles

- **Reducir las dependencias de interfaz.** El plan pide Next.js en modo exportación estática, Recharts y Tailwind ([líneas 67–72](</C:/projectes/finally/planning/PLAN.md:67>), [líneas 419–423](</C:/projectes/finally/planning/PLAN.md:419>)). Si no se necesitan funciones propias de Next.js, una SPA más simple podría reducir configuración y pasos de build; si Next.js es un objetivo pedagógico, mantenerlo está justificado.
- **Aplazar el soporte de Massive.** El simulador ya cubre el flujo funcional completo ([líneas 152–161](</C:/projectes/finally/planning/PLAN.md:152>)); la integración real añade polling, límites de suscripción, validación de símbolos y gestión de datos obsoletos ([líneas 163–172](</C:/projectes/finally/planning/PLAN.md:163>)). Para una primera entrega del curso, podría dejarse como extensión.
- **Concretar el alcance del gráfico principal.** Solo se exige mostrar el precio a lo largo del tiempo, mientras que el plan no define persistencia ni ventana temporal de esos datos ([líneas 25–26](</C:/projectes/finally/planning/PLAN.md:25>), [líneas 408–411](</C:/projectes/finally/planning/PLAN.md:408>)). Usar el historial de ticks recibido desde que se abrió la página evitaría añadir otra tabla o endpoint.
- **Revisar la complejidad de la validación de símbolos.** En modo simulado se acepta cualquier ticker con formato válido; con Massive se consulta la API de referencia al añadir o comprar ([líneas 158–160](</C:/projectes/finally/planning/PLAN.md:158>), [líneas 302–306](</C:/projectes/finally/planning/PLAN.md:302>)). Si no se necesita comprobar existencia real, se podría usar una regla uniforme basada solo en formato y evitar una llamada adicional por ticker.

## Preguntas de aclaración

- ¿El usuario debe confirmar una propuesta del asistente antes de que se ejecute, o basta con pedirla en el mismo mensaje? (§9)
- ¿Qué debe hacer la app si el último precio disponible tiene más de cierto tiempo? (§7, §8)
- ¿El despliegue en nube queda expresamente fuera del alcance funcional por operar sin autenticación? (§11)
- ¿El gráfico de precio debe conservar datos entre sesiones o solo acumular ticks desde que se abre la página? (§2, §10)

No edité el plan; este es feedback de revisión.

---

## Revisión de cambios sin commit desde HEAD (2026-11-23)

**Alcance:** `.gitignore`, `CLAUDE.md`, `backend/app/market_data/` (`massive_client.py`, `simulator.py`, `simulator_config.py`), `backend/tests/market_data/test_massive_client.py`, `planning/PLAN.md`, `planning/market_data_review.md`, `planning/market_simulator.md`, `planning/massive_api.md`, y `.claude/agents/`, `.claude/commands/`. No se ha revisado `planning/review_codex.md`.

**Método y limitaciones:** esta sesión no dispone de herramienta Bash, así que no se ha podido ejecutar `git diff HEAD`. La revisión se hizo sobre el contenido actual de cada archivo; no se puede aislar el delta exacto frente a HEAD. Tampoco se han ejecutado los tests (`uv run pytest`); los hallazgos sobre tests son de lectura.

**Estado del repositorio:** en el árbol de trabajo solo existe `backend/app/market_data/`. No hay `main.py`, rutas API, base de datos, integración LLM, frontend, `Dockerfile`, `.dockerignore`, `scripts/` ni `.env.example`. Lo revisado es solo la capa de datos de mercado.

### Alta

1. **Una respuesta no JSON de Massive detiene el sondeo para siempre.** `backend/app/market_data/massive_client.py:82` solo captura `httpx.HTTPError`, pero `resp.json()` (línea 106) puede lanzar `json.JSONDecodeError` (`ValueError`). Con un cuerpo vacío y HTTP 200, que PLAN §6 (`PLAN.md:172`) exige tratar como "mantener la última caché válida", la tarea termina sin log y los precios quedan congelados. No hay test para este caso. Corrección: validar la respuesta dentro de `poll_once` o capturar también `ValueError`, y añadir un test con cuerpo vacío.

2. **Massive emite un tick por ticker en cada ciclo, aunque el precio no cambie.** `massive_client.py:110-114`. PLAN §6 (`PLAN.md:193`) exige que el SSE emita solo cuando el precio cambió y que la UI no destelle si no cambió. La regla debería vivir en un único punto (p. ej. `PriceCache.update`), no en el provider.

3. **El `timestamp` es la hora de emisión, no la de observación.** `backend/app/market_data/types.py:32` fija `datetime.now()` dentro de `PriceTick.create`, y `massive_client.py:111` lo usa. PLAN §6 (`PLAN.md:192`) y `planning/massive_api.md §6` piden `lastTrade.t` (epoch en milisegundos). Esto afecta a la regla de "desactualizado tras 60 s" (§10). `create` debería aceptar el instante observado y convertir el epoch a ISO UTC. Los tests no comprueban el timestamp.

4. **Contrato SSE: `prev_price` en lugar de `previous_price`.** `types.py:40` emite `prev_price`. PLAN §6 (`PLAN.md:186-189`) y `planning/market_data_interface.md:35` usan `previous_price`. `backend/tests/market_data/test_types.py:32` fija el nombre incorrecto, así que la divergencia está codificada en el test. Conviene decidir el nombre antes de escribir el cliente del frontend.

5. **Calibración del simulador: el movimiento por tick es irreal a largo plazo.** `simulator_config.py:42` define `EFFECTIVE_DT_PER_TICK = 1/360`, que equivale a 1 año simulado cada 360 ticks (3 min reales), no a un día como indican `planning/market_simulator.md:76-82` y el comentario de `market_data_review.md:84`. Consecuencias aproximadas:
   - Por tick, AAPL (σ = 0.28) se mueve σ·√(1/360) ≈ 1.5 %; por minuto, ≈ 16 %.
   - En una hora (7200 ticks ≈ 20 "años"), la mediana de AAPL (S0 = 190, μ = 0.10) sería ≈ 190·exp((0.10 − 0.28²/2)·20) ≈ 640 USD. TSLA (σ = 0.55, μ = 0.05) bajaría a ≈ 33 USD.
   - Esto contradice "dramático pero no absurdo" (`market_data_review.md:84`) y "sin ser absurdo" (`market_simulator.md:82`).
   - Si la intención es "un día por 3 min", el dt debería ser (1/252)/360 ≈ 1/90720, con ≈ 0.1 % por tick y ≈ 1 % por minuto para AAPL. Requiere decisión y validación visual.

### Media

6. **Frecuencia de saltos mal documentada.** `simulator_config.py:36` dice "~1 cada ~20 min a 500 ms/tick", pero `simulator.py:106` sortea la probabilidad por ticker y por tick. Con 0.0008 × 2 ticks/s, cada ticker tiene un salto cada ~10 min, y con 10 tickers hay ~1 por minuto en la watchlist. `market_simulator.md:89` propone 0.003 por ticker y tick. Hay que elegir un valor y corregir el comentario; `market_data_review.md:85` repite la cifra errónea.

7. **El documento de diseño del simulador no coincide con el código.**
   - `market_simulator.md` §3 describe Cholesky y §7 un `SimulatedProvider` con numpy, `PriceCache`, `Broadcaster`, `SEED_PRICES` y `DEFAULT_CORRELATION_MATRIX`. Nada de eso existe; el código usa el modelo de tres factores de `correlation.py`.
   - Las σ de §4 no coinciden con `simulator_config.py`: AAPL 25 % vs 0.28, GOOGL 28 vs 0.30, MSFT 22 vs 0.26, AMZN 30 vs 0.32, META 32 vs 0.34, JPM 20 vs 0.22, V 18 vs 0.20, NFLX 35 vs 0.33.
   - La deriva "5–8 %" no coincide con 0.05–0.18 de la config (NVDA 0.18, AMZN 0.11).
   - §3 pone AMZN en "Consumo/EV"; la config lo asigna a "tech".
   - PLAN define los documentos de `planning/` como contrato compartido; conviene actualizarlos o marcarlos como históricos.

8. **Fallback a cierres anteriores como precio actual.** `massive_client.py:129-133` usa `day.c` o `prevDay.c` si falta `lastTrade.p`. PLAN §6 (`PLAN.md:170`, `172`) define `price = lastTrade.p` y que una respuesta vacía mantenga la caché. El fallback emite el cierre de ayer como precio actual, produce un destello falso y el test `test_massive_client.py:81-96` lo codifica. Además, `or` trata `p == 0` como ausente. Recomiendo eliminar el fallback: sin `lastTrade.p`, no se emite tick.

9. **La referencia del "% de cambio diario" no existe en el modelo de datos.** PLAN §6 (`PLAN.md:194`) usa `prevDay.c` (Massive) o el precio de arranque (simulador). `PriceTick` y `CachedPrice` solo tienen `price` y `prev_price`, y ningún componente guarda esa referencia por ticker. Sin ella no se puede calcular el % de cambio de §10 (`PLAN.md:412`).

10. **Estado de conexión (verde/amarillo/rojo) sin implementar.** PLAN §6 (`PLAN.md:172`) exige pasar a amarillo ante 429/5xx y a rojo tras 60 s sin datos válidos. Ni el cliente ni la caché registran la última observación válida ni el último error. Hace falta un punto de estado que exponga la API.

11. **Validación de altas sin caché y sin traducción de 429.** `massive_client.py:92-95` no cachea el resultado de `validate_ticker`, que PLAN §8 (`PLAN.md:309`) exige por ticker, existente o no. El 429 sale como `HTTPStatusError` genérica, que la capa API deberá traducir a `503` (`PLAN.md:303`). Hay que decidir dónde vive la caché de validación. Además, en el plan gratuito el presupuesto es de 5 llamadas/min: 4 sondeos/min dejan 1/min para altas y compras. Conviene documentarlo.

12. **`poll_once` con lista vacía.** `massive_client.py:103-104` envía `tickers=""` si no hay tickers. `_run_loop` lo evita con `if self._tickers`, pero `poll_once` es público. Debería retornar antes de llamar a la API.

13. **Cobertura y calidad de `test_massive_client.py`.**
    - `:145` modifica `provider._poll_interval_seconds` en lugar de pasar `poll_interval_seconds` al constructor, que ya lo admite.
    - `:191` accede a `provider._client.is_closed`.
    - Faltan tests para: cuerpo no JSON (ver 1), lista vacía (ver 12), respuesta sin `lastTrade.p` (ver 8), supresión de ticks sin cambio (ver 2), timestamp observado (ver 3) y 429 en el sondeo.
    - `planning/market_data_review.md:136` cuenta 10 tests para `massive_client`; hay 13.

### Baja

14. **`.gitignore:210` contiene `.end`.** Es un typo. `.env` ya está en la línea 138, así que la entrada nueva no protege nada. Debería corregirse a `.env` o eliminarse.

15. **Falta `.env.example`.** PLAN §4 (`PLAN.md:106`) lo exige y `.gitignore:212` ya prevé la excepción `!.env.example`. Es necesario para documentar `MASSIVE_POLL_INTERVAL_SECONDS`.

16. **Configuración no especificada.** `massive_config.py:4-5` añade `MASSIVE_BASE_URL` y `MASSIVE_POLL_INTERVAL_SECONDS`, que no están en PLAN §5. El intervalo tiene sentido para planes de pago; `MASSIVE_BASE_URL` probablemente no. Si se mantienen, documentarlos.

17. **`factory.py:14` vuelve a leer el entorno.** Duplica `massive_config.MASSIVE_API_KEY` y aplica `.strip()`, que el cliente no aplica (`massive_config.py:3`, `massive_client.py:40`, parámetro por defecto evaluado al importar). Un valor con espacios se elige como Massive pero la cabecera de autorización llevaría esos espacios. Conviene una sola fuente de verdad.

18. **`planning/market_data_review.md` tiene afirmaciones obsoletas.** Marca como "Correcto" `massive_client.py` y `simulator_config.py` pese a los puntos 5, 6 y 8; cita "47/47" y "10" tests por módulo, que ya no cuadran con el código.

19. **Agentes de revisión solapados y mal configurados.**
    - `.claude/agents/reviewer.md:4` declara `tools: Read, Glob, Grep, Write`, sin Bash, así que no puede ejecutar `git diff`. Esta sesión lo ha demostrado.
    - `codex_reviewer.md:2` usa `name: reviewer`, igual que `reviewer.md`, y necesita Bash para `codex exec` (`codex_reviewer.md:7`), que no tiene.
    - `codex_reviewer.md:7` escribe en `planning/review.md`, el mismo archivo que `reviewer.md`; se pisan la salida.
    - `change-reviewer.md` y `reviewer.md` se solapan casi por completo; `doc-review.md` es una tercera variante.
    - Typo en `change-reviewer.md:3` ("últipo").
    - Propuesta: un único agente con Read/Glob/Grep/Write/Bash y un archivo de salida por agente.

### Simplificaciones

- **Subir a `MarketDataProvider`** (`base.py`) la gestión de tickers (`add_ticker`, `remove_ticker`, `tickers`), `start`/`stop` y el bucle. Cada implementación solo debe aportar `_produce_ticks()` y el intervalo. Hoy `simulator.py:39-82` y `massive_client.py:56-84` duplican casi todo.
- **Centralizar `prev_price`, `direction` y el filtro "solo si cambió" en `PriceCache`.** Los providers solo entregarían (ticker, precio, timestamp observado). Eliminaría `_prev_prices` (`massive_client.py:47,110-112`) y la duplicación de estado.
- **Quitar el `asyncio.Lock` de `PriceCache.update`** (`cache.py:24,26-32`). No hay `await` dentro y asyncio es monohilo; el lock no protege nada. El lock que pide PLAN §7 es el de las operaciones, que es otro componente.
- **Eliminar los fallbacks `day.c` / `prevDay.c`** (ver 8) y su test.
- **Integrar `wiring.py` en `factory.py`.** Es una función de una línea; `build_market_data_provider` ya recibe `on_tick`.
- **Quitar `MASSIVE_BASE_URL`** hasta que haga falta (ver 16).
- **Un solo agente de revisión** (ver 19).

### Consistencia con PLAN y CLAUDE.md

- `CLAUDE.md` (raíz) y `PLAN.md` no se contradicen en las decisiones vigentes (seed 50–300, validación de altas, `LLM_MOCK`, límites por respuesta).
- Desviaciones ya listadas: SSE (4), timestamp (3), emisión sin cambio (2), fallbacks (8), referencia de % diario (9), estado de conexión (10).

### Preguntas y aclaraciones

1. ¿Qué ritmo de simulación se quiere: 1 año simulado por 3 min (el código actual, ver 5) o 1 día por 3 min? ¿Hay un criterio visual para aceptar un valor?
2. ¿Quién guarda la referencia de "% de cambio diario" y cuál es: `prevDay.c` en Massive y precio de arranque en el simulador (ver 9)?
3. ¿Se confirma `previous_price` como nombre del campo SSE (ver 4)? Eso implica actualizar `test_types.py:32`.
4. ¿Dónde vive la caché de validación de altas y quién traduce el 429 de Massive a `503` (ver 11)?
5. ¿El endpoint de snapshot por lotes y los límites citados en `planning/massive_api.md` (§1, §5) se han verificado contra la documentación vigente? PLAN §13 lo deja pendiente y no he podido comprobarlo en esta revisión.
6. ¿El filtro "solo si cambió" debe aplicarse también al simulador? Con el simulador el precio casi siempre cambia, así que el efecto es mínimo, pero la regla debería ser igual para ambas fuentes.
7. ¿Cuándo se añade el resto del backend (API, base de datos, LLM)? El PLAN describe mucho más de lo que hay en el árbol.
8. ¿Deben versionarse `.claude/agents/` y `.claude/commands/`?

### Verificación pendiente

Tras corregir, ejecutar `uv run pytest` desde `backend/`. Los tests de `test_massive_client.py` no cubren los casos 1, 2, 3 y 12; conviene añadirlos antes de dar el módulo por cerrado.

---

## Pendientes consolidados (2026-11-23)

Estado de todos los puntos: **pendiente**. Origen: `[R]` = revisión por subagente de esta sesión (secciones anteriores de este archivo); `[C]` = revisión Codex (`planning/review_changes_codex.md`). Los puntos repetidos en ambas fuentes se listan una sola vez.

### Alta

- [ ] **[R1]** Respuesta no JSON de Massive detiene el sondeo (`massive_client.py:82`, `:106`). Capturar `ValueError` o validar respuesta; test con cuerpo vacío.
- [ ] **[R2]** Massive emite tick aunque el precio no cambie (`massive_client.py:110-114`). Centralizar la regla en `PriceCache.update`.
- [ ] **[R3]** `timestamp` es hora de emisión, no de observación (`types.py:32`). Usar `lastTrade.t` en ISO UTC; test del timestamp.
- [ ] **[R4]** `prev_price` vs `previous_price` (`types.py:40`, `test_types.py:32`). Decidir nombre y corregir código y test.
- [ ] **[R5]** Calibración del simulador: `EFFECTIVE_DT_PER_TICK = 1/360` equivale a 1 año por 3 min (`simulator_config.py:42`). Decidir ritmo (ver pregunta 1) y validar visualmente.

### Media

- [ ] **[C1 / R11]** `validate_ticker()` sin callers (`massive_client.py:86`). PLAN §8 exige validar altas y compras sin precio en caché.
- [ ] **[C2 / R11]** `validate_ticker()` no cachea resultados positivos ni negativos (`massive_client.py:92`, PLAN.md:309).
- [ ] **[R11]** El 429 sale como `HTTPStatusError` genérica; traducir a `503` (PLAN.md:303). Documentar presupuesto gratuito (4 sondeos/min dejan 1/min para altas y compras).
- [ ] **[R6]** Comentario de `simulator_config.py:36` ("~1 cada ~20 min") incorrecto; la probabilidad es por ticker y tick. Corregir también `market_data_review.md:85`.
- [ ] **[R7]** `market_simulator.md` no coincide con el código (Cholesky, numpy, `Broadcaster`, σ de §4, deriva, sector de AMZN). Actualizar o marcar como histórico.
- [ ] **[R8]** Fallback `day.c` / `prevDay.c` como precio actual (`massive_client.py:129-133`). Eliminar y su test (`test_massive_client.py:81-96`). `or` trata `p == 0` como ausente.
- [ ] **[R9]** Falta la referencia del % de cambio diario (PLAN.md:194): `prevDay.c` en Massive y precio de arranque en el simulador.
- [ ] **[R10]** Estado de conexión verde/amarillo/rojo sin implementar (PLAN.md:172).
- [ ] **[R12]** `poll_once` con lista vacía envía `tickers=""` (`massive_client.py:103-104`). Retornar antes de llamar a la API.
- [ ] **[R13]** Tests de `test_massive_client.py`: `:145` modifica atributo privado en vez de usar el constructor; `:191` accede a `_client.is_closed`; faltan casos (JSON inválido, lista vacía, sin `lastTrade.p`, supresión sin cambio, timestamp, 429). `market_data_review.md:136` dice 10 tests; hay 13.

### Baja

- [ ] **[R14 / C]** `.gitignore:210` contiene `.end` (typo de `.env`). Corregir o eliminar.
- [ ] **[R15]** Falta `.env.example` (PLAN.md:106; `.gitignore:212` ya prevé la excepción).
- [ ] **[R16]** `massive_config.py:4-5` añade `MASSIVE_BASE_URL` y `MASSIVE_POLL_INTERVAL_SECONDS`, no especificados en PLAN §5. Documentar o quitar `MASSIVE_BASE_URL`.
- [ ] **[R17]** `factory.py:14` relee el entorno con `.strip()`, que el cliente no aplica. Una sola fuente de verdad (`massive_config`).
- [ ] **[R18]** `market_data_review.md` marca "Correcto" módulos con errores de los puntos R5, R6 y R8, y tiene recuentos de tests obsoletos.
- [ ] **[R19 / C]** Agentes de revisión con `name: reviewer` duplicado (`reviewer.md`, `codex_reviewer.md`, `change-reviewer.md`); `reviewer.md` sin Bash; `codex_reviewer.md` escribe en `planning/review.md` como `reviewer.md`; typo "últipo" en `change-reviewer.md:3`. Propuesta: un agente con Bash y un archivo de salida por agente.

### Simplificaciones

- [ ] Subir a `MarketDataProvider` (`base.py`) la gestión de tickers, `start`/`stop` y el bucle (duplicados en `simulator.py:39-82` y `massive_client.py:56-84`).
- [ ] Centralizar `prev_price`, `direction` y el filtro "solo si cambió" en `PriceCache`; eliminar `_prev_prices` (`massive_client.py:47,110-112`).
- [ ] Quitar el `asyncio.Lock` de `PriceCache.update` (`cache.py:24,26-32`); no hay `await` dentro.
- [ ] Integrar `wiring.py` en `factory.py`.
- [ ] Quitar `MASSIVE_BASE_URL` hasta que haga falta (R16).

### Preguntas abiertas

1. ¿Ritmo de simulación: 1 año por 3 min (actual) o 1 día por 3 min? (R5)
2. ¿Quién guarda la referencia del % de cambio diario? (R9)
3. ¿Se confirma `previous_price` como nombre SSE? (R4)
4. ¿Dónde vive la caché de validación de altas y quién traduce el 429 a `503`? (C2, R11)
5. ¿Se verificaron el snapshot por lotes y los límites de `planning/massive_api.md` (§1, §5) contra la documentación vigente? (PLAN §13)
6. ¿El filtro "solo si cambió" aplica también al simulador? (R2)
7. ¿Cuándo se añade el resto del backend (API, base de datos, LLM)?
8. ¿Se versionan `.claude/agents/` y `.claude/commands/`? (R19)
