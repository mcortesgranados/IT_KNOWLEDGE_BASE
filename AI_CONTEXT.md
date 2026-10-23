# AI CONTEXT — IT Knowledge Base

Documento de traspaso para asistentes de IA. Resume la arquitectura del repositorio, las convenciones que se aplican y todo lo construido en la sesión del 22–23 de octubre de 2026. Léelo antes de proponer cambios: varias decisiones no son obvias y revertirlas rompe enlaces publicados.

---

## 1. Mapa del proyecto

| Repositorio | Ruta local | Remoto | Contenido |
|---|---|---|---|
| Principal | `E:\IT_KNOWLEDGE_BASE` | `github.com/mcortesgranados/IT_KNOWLEDGE_BASE` | sitio publicado, hub `IT.html`, bancos de preguntas |
| AWS | `E:\IT_KNOWLEDGE_BASE_AWS` | `github.com/mcortesgranados/IT_KNOWLEDGE_BASE_AWS` | 96 cursos, ~9.4 GB |
| GCP | `E:\IT_KNOWLEDGE_BASE_GCP` | `github.com/mcortesgranados/IT_KNOWLEDGE_BASE_GCP` | 55 cursos, 402 capítulos, 2.8 GB |
| Azure | `E:\IT_KNOWLEDGE_BASE_AZURE` | — | material Azure separado |
| CV | `E:\MCORTESGRANADOS_GITHUB_IO` | `github.com/mcortesgranados/CV_MCG` | `CV_MCG/2025/*`, procesos de selección |

Sitio: `https://mcortesgranados.github.io/IT_KNOWLEDGE_BASE/`
Hub: `IT.html` — tarjetas numeradas que enlazan cada tema.

Los tres repos de cloud están fuera del principal a propósito: el `.git` del repo único llegó a ~28 GB y las operaciones tardaban minutos. Sacar carpetas **no** reduce el `.git` (la historia conserva los blobs); eso requeriría `git filter-repo`, decisión que sigue pendiente.

---

## 2. Convenciones

- **Idioma del contenido: inglés.** Aunque el pedido llegue en español, las páginas del sitio se escriben en inglés. Los documentos meta (este archivo, el diario) van en español.
- **Nombres sin espacios** en carpetas y archivos del sitio; se usan guiones bajos.
- **Rutas relativas** en todos los HTML del sitio. Nada de `file:///E:/...` ni rutas absolutas.
- **Repos de cursos:** un commit por capítulo, un push por curso, mensajes en inglés con curso y capítulo:
  `AWS | <course> | <chapter>: add chapter content`
- **Antes de cualquier migración de cursos en Windows:**
  ```
  git config core.longpaths true
  ```
  Sin esto, Git omite en silencio archivos con rutas > 260 caracteres. En la migración de AWS quedaron 29 carpetas fuera por este motivo y hubo que hacer un cierre aparte; en GCP se activó desde el inicio y no se perdió nada.
- **Commit/push:** el usuario pide confirmación explícita antes de publicar. Después del push se verifica la URL en vivo, no solo el estado local.
- **GitHub Pages sirve `Cache-Control: max-age=600`.** Un cambio recién desplegado puede tardar hasta 10 minutos en verse, y el navegador puede seguir mostrando la copia anterior. Para verificar de verdad, pedir la URL con un parámetro aleatorio (`?cb=<timestamp>`).

---

## 3. Patrón de banco de preguntas (reutilizable)

Todos los bancos nuevos comparten la misma estructura. El shell se copia de `FRONTEND/ANGULAR/01_angular_all_questions.html`.

### 3.1 Estructura del HTML

```
<section class="hero">      → título, descripción, chips con conteos
<div class="qsearch">       → input#search-input, contador, lista de resultados
<div class="filterbar">     → botones data-level="all|beginner|intermediate|advanced|blocks"
<section class="lvl" id="beginner" data-level="beginner">
  <h2 class="lvlhead">Beginner <span>280 questions</span></h2>
  <p class="lead">…</p>
  <div class="question">
    <h2 id="q1" data-num="1">1. Question text?</h2>
    <div class="answer"><p>…</p><pre><code>…</code></pre></div>
  </div>
```

### 3.2 Comportamiento del buscador

- Índice en vivo sobre pregunta **y** respuesta; el contador muestra `N matches`.
- Salto directo a la pregunta y resaltado temporal.
- **Auto-seleccionado del texto** al hacer clic en la casilla: `focus` selecciona todo; `click` vuelve a seleccionar solo si no hubo arrastre (comparación de `mousedown` vs `click`, umbral 4 px).
- **Deep link `?q=<término>`**: rellena la casilla, ejecuta la búsqueda, sube al inicio y selecciona el texto. Se reaplica en `pageshow` para sobrevivir a restauraciones de bfcache.

### 3.3 Producción por lotes

1. Los fragmentos se generan en paralelo con sub-agentes (`sessions_spawn`), cada uno con un rango temático disjunto.
2. Cada fragmento contiene solo bloques `<div class="question">…</div>`, **sin numerar**, con `<`, `>` y `&` escapados dentro de `<code>`.
3. Un script Python ensambla: numera, asigna `id="qN"` y `data-num`, inserta las secciones y pega el shell.
4. Se borra la carpeta de fragmentos al terminar.

Scripts de referencia en `%TEMP%\webp_migration\`:
`build_dotnet_questions.py`, `build_graphql_questions.py`, `build_kafka_questions.py`, `check_ai_pages.py`.

### 3.4 Verificación obligatoria

| Comprobación | Criterio |
|---|---|
| Ids | `q1..qN` secuenciales, sin huecos ni repetidos |
| Duplicados | 0 textos de pregunta repetidos entre lotes |
| Etiquetas | `<div>` abiertos = cerrados; `<pre>` y `<code>` balanceados |
| Escapado | Ningún `<` sin escapar dentro de `<code>` (rompe el render) |
| Búsqueda | Prueba real en Edge headless con `?q=` y lectura del contador |
| Enlaces del hub | Todos los `href` relativos de `IT.html` resuelven archivo y ancla |

---

## 4. Construido en esta sesión

### 4.1 Migraciones de repositorios

- **AWS** → `E:\IT_KNOWLEDGE_BASE_AWS`. Commits por capítulo, push por curso, ~719 commits, 0 fallos tras el cierre de la brecha de rutas largas.
- **GCP** → `E:\IT_KNOWLEDGE_BASE_GCP`. 55 cursos, 402 capítulos, **457 commits, 55 pushes**, 0 fallos, 32,4 minutos.
- **Azure**: 8 MP4 (~1 GB) movidos a `E:\IT_KNOWLEDGE_BASE\MEDIA\MS_AZURE_VIDEOS`; ninguna referencia HTML apuntaba a ellos.

### 4.2 Bancos de preguntas

| Tecnología | Archivo | Preguntas | Composición |
|---|---|---|---|
| Angular | `FRONTEND/ANGULAR/01_angular_all_questions.html` | 1011 | 280 / 340 / 377 / 14 |
| .NET 8+, C#, REST | `DOT_NET/01_dotnet_all_questions.html` | 1011 | 280 / 340 / 377 / 14 |
| GraphQL | `GRAPHQL/BASICO.html` (reemplazado) | 1011 | 280 / 340 / 377 / 14 |
| Kafka | `KAFKA/KAFKA_ALL_QUESTIONS.html` | 644 | 210 / 210 / 210 / 14 |
| MongoDB | `MONGODB/01_mongodb_all_questions.html` | 1120 | 300 / 380 / 426 / 14 |

Kafka se consolidó en una sola página desde 90 guías temáticas; el contenido está escrito contra **Kafka 4.x**, donde KRaft es el único modo y ZooKeeper desapareció en 4.0 (marzo 2025).

### 4.3 Páginas de referencia

| Archivo | Contenido |
|---|---|
| `AI/generative_ai.html` | transformer, entrenamiento, decodificación, RAG, evaluación, economía de tokens, modos de fallo |
| `AI/agentic_ai.html` | loop ReAct, patrones de orquestación, diseño de herramientas, memoria, $p^n$, guardrails |
| `AI/mcp_model_context_protocol.html` | spec **2026-07-28**: sin sesiones ni handshake, `server/discover`, multi round-trip, OAuth 2.1 + resource indicators, CIMD, deprecación de roots/sampling/logging, extensiones |
| `DOT_NET/data_access_efcore_dapper.html` | EF Core 10 (LTS nov-2025, soporte hasta nov-2028), Dapper, NHibernate, linq2db, Marten, RepoDB, matriz comparativa, guía de decisión |

### 4.4 Índice `IT.html`

- Tarjetas nuevas: **34** Generative AI, **35** Agentic AI, **36** MCP, **37** Data access, **38** MongoDB.
- Actualizadas: **06** .NET (banco de 1011), **11** GraphQL (banco de 1011 + anclas), **13** Kafka (página única + anclas).

### 4.5 Página del proceso SMASH

`CV_MCG/2025/smash_full_stack_dotnet_angular_p169_process.html`

- Un recuadro rojo dentro de cada una de las 10 tarjetas de concepto del JD, con enlaces a las preguntas técnicas del proceso (`smash_p169_technical_questions.html#q1…#q10`).
- Los 8 ítems de "Hands-on Angular experience with:" son enlaces que abren el banco de Angular con el término en la casilla:

| Tópico | Término | Resultados |
|---|---|---|
| Lazy-loaded modules | `lazy loading` | 19 |
| Route guards | `guard` | 18 |
| Interceptors | `interceptor` | 26 |
| RxJS | `rxjs` | 68 |
| State management | `state management` | 7 |
| Reusable components | `reusable` | 21 |
| Custom directives and pipes | `directive` | 248 |
| Reactive forms | `reactive form` | 34 |

### 4.6 Página de revenue management

`CV_MCG/2025/revenue_management.html` — creada antes de esta sesión, sigue sin commitear.

---

## 5. Incidentes y lecciones

1. **Rutas largas en Windows.** `git add -A` no falla: simplemente omite. Verificar con `git status` después de cada lote. Solución: `core.longpaths true`.
2. **Caché de GitHub Pages.** `?q=` "no funcionaba" en producción; el HTML servido tenía `max-age=600` y el navegador mostraba la versión previa al despliegue. Se comprobó con perfil limpio y se añadió reaplicación en `pageshow` (commit `b32ab45bb`).
3. **Escapado dentro de `<code>`.** Un fragmento generado contenía `restore.consumer.<client config>` sin escapar; el navegador lo interpretaba como etiqueta y perdía el texto. La verificación de "ningún `<` sin escapar" lo detectó.
4. **Mantenimiento automático de Git.** Los procesos largos se pausan cuando Git lanza `gc`/`repack`; no es un cuelgue. Al reiniciar se encontró un `gc.pid` huérfano y procesos `git add -A` concurrentes de GitHub Desktop y VS Code.
5. **PowerShell 5.1.** `Set-Content -Encoding UTF8` escribe BOM: hay que eliminarlo después para no alterar los HTML. Los JSON para CLIs van por archivo, no inline.
6. **Duplicados heredados.** El chequeo de duplicados hay que lanzarlo sobre **todos** los fragmentos, no por lote: al ensamblar MongoDB aparecieron 4 títulos repetidos, y 3 venían de lotes antiguos que nunca se habían cruzado (dos en `m_b1`/`m_b2` y uno en `m_i1`/`m_i2`). Se reescribió la segunda copia de cada par; el conteo total no cambia.

---

## 6. Estado actual y pendientes

### Repo principal `E:\IT_KNOWLEDGE_BASE`

```
 M  GRAPHQL/BASICO.html                       banco de 1011
 M  IT.html                                   tarjetas 06, 11, 13, 34-38
?? AI_CONTEXT.md
?? DOT_NET/data_access_efcore_dapper.html
?? KAFKA/KAFKA_ALL_QUESTIONS.html
?? MONGODB/
```

### MongoDB — completo (1120)

`MONGODB/01_mongodb_all_questions.html`, ensamblado con el shell de Angular. **1120 preguntas: 300 / 380 / 426 / 14.** Es la única página con 1120; el resto de bancos usa 1011 o 644.

Nota de composición: al pasar de 1011 a 1120 se mantuvieron los tramos ya escritos (beginner 280, intermediate 315) y el resto se concentró en advanced. Por eso el reparto no es proporcional al de Angular.

Lotes de origen en `MONGODB/MONGODB_ALL_Q_PARTS/` (13 archivos): `m_b1` 140, `m_b2` 140, `m_b3` 20 · `m_i1` 170, `m_i2` 145, `m_i3` 65 · `m_a1` sharding (71), `m_a2` transacciones y consistencia (71), `m_a3` WiredTiger y motor de consultas (71), `m_a4` replicación y HA (71), `m_a5` backup y seguridad (71), `m_a6` Atlas y operaciones (71) · `k01` recetas (14).

Dos detalles que no son obvios:
- Los fragmentos **no** llevan numeración ni `id`; el script de ensamblado añade `id="qN"`, `data-num` y el prefijo `N. ` dentro del `<h2>`, igual que Kafka y Angular.
- La sección de recetas usa `id="blocks"` / `data-level="blocks"` (convención de Kafka), no `id="reuse"` como Angular.

### Otros repos

| Repo | Estado |
|---|---|
| AWS | 20 archivos pendientes y 3 commits sin empujar |
| GCP | limpio |
| CV | JD modificado, `revenue_management.html` y varios PNG sin rastrear |

---

## 7. Comandos útiles

```powershell
# contar preguntas de un banco
(Select-String -LiteralPath .\ARCHIVO.html -Pattern '<div class="question">' -AllMatches).Matches.Count

# verificar que un banco abre y busca (Edge headless)
& "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" --headless=new --disable-gpu `
  --virtual-time-budget=6000 --dump-dom "file:///E:/IT_KNOWLEDGE_BASE/RUTA.html?q=termino"

# comprobar el despliegue sin caché
Invoke-WebRequest "https://mcortesgranados.github.io/IT_KNOWLEDGE_BASE/RUTA.html?cb=$(Get-Random)"
```

---

_Última actualización: 2026-10-23. Si cambias convenciones (idioma, numeración, estructura del buscador), actualiza este archivo en el mismo commit._
