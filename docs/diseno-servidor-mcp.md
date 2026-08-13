# Diseño: servidor MCP de startup-next

> Documento de diseño (mismo criterio que el resto del proyecto: se escribió
> antes de implementar). Define la pieza que conecta el conocimiento de
> startup-next con un agente personal del emprendedor (Hermes Agent de Nous,
> u otro host MCP). **Implementado y verificado**: las dos herramientas
> (`siguiente_accion` y `consejos_para_actividad`) funcionan de punta a punta
> contra Hermes Agent real, verificadas por Telegram (commit `ceaec5e`).

## El pivote que este documento formaliza

Durante una parte del proyecto se construyó `hermes-startup-next` como un agente
autónomo: un proceso persistente con su propio canal, su propia memoria y su
propio empaquetado Docker. Esa construcción se apoyaba en un malentendido: que
el "Hermes" de la arquitectura había que construirlo.

No hay que construirlo. **Hermes Agent existe** — es un agente personal de IA de
código abierto (MIT), de Nous Research, que un emprendedor instala en su propia
máquina (local, Docker, VPS…). Ya trae de fábrica lo que aquel agente propio
intentaba replicar: proceso persistente, memoria multi-nivel entre sesiones,
creación de "skills", y canales de mensajería (Telegram, Discord, Slack,
WhatsApp, Signal…). Y —lo decisivo— se extiende conectándose a **servidores MCP**
(Model Context Protocol).

La contribución de este proyecto nunca fue el agente. Fue, y sigue siendo, el
**conocimiento especializado** (startup-next) y **la forma de ponerlo a
disposición de un agente**. Con Hermes Agent en escena, esa forma es un servidor
MCP. Este documento lo diseña.

Lo que se jubila (el agente autónomo, su canal, su memoria propia, su Docker, su
adaptador LLM) se jubila porque Hermes Agent ya lo aporta. Lo que se conserva
—startup-next entero, su cliente HTTP, la tesis, el conocimiento OKF/RAG— es lo
que de verdad valía.

## Qué es este servidor (y qué no es)

- **Es un servidor MCP estándar.** Habla el Model Context Protocol, que es un
  estándar abierto. No es "un plugin de Hermes": es un servidor MCP que
  *cualquier* host MCP puede consumir (Hermes Agent, Claude Code, Cursor, el que
  venga). Esto desacopla la contribución de este proyecto de cualquier agente
  concreto — se construye para el estándar, no para un producto.
- **Es de consulta, read-only.** Expone conocimiento; no escribe nada en ningún
  sitio. No tiene estado. Cada llamada es independiente.
- **Es fino.** Traduce entre el protocolo MCP y la API HTTP de startup-next que
  ya existe y está verificada. Cada herramienta MCP, por dentro, hace una llamada
  HTTP al startup-next desplegado. No reimplementa lógica de conocimiento — la
  delega en el central.
- **No guarda memoria de la startup.** La memoria vive en el agente (Hermes). El
  contexto histórico llega como *entrada* de cada llamada; startup-next lo procesa
  y lo olvida. startup-next sigue siendo, como siempre, conocimiento puro sin
  estado.

## Transporte y una regla técnica que condiciona el diseño

- **Transporte: stdio.** El servidor corre localmente, junto al agente, en la
  máquina del emprendedor. El host (Hermes) lo lanza como proceso hijo y le habla
  por stdin/stdout mediante JSON-RPC. Es el transporte que MCP define para
  servidores locales, y el que Hermes espera para servidores propios (no del
  catálogo).
- **Regla crítica — nunca escribir a stdout.** Con transporte stdio, stdout está
  reservado para los mensajes JSON-RPC del protocolo. Cualquier `console.log` a
  stdout **corrompe el protocolo y rompe el servidor**. Todo el logging debe ir a
  **stderr**. Esta regla es absoluta y hay que respetarla desde la primera línea
  (es, irónicamente, lo contrario del agente jubilado, que usaba stdout para
  hablar con el fundador).

## Las herramientas

El flujo conversacional real que el servidor debe soportar es de dos pasos:

```
1P: Soy la startup A, esta es mi situación + mi contexto histórico. ¿Cuál es mi siguiente acción?
1R: Tu siguiente actividad es X.

2P: Para la actividad X (o Y), ¿cuáles son los mejores consejos disponibles?
2R: Para tu actividad X (o Y), los mejores consejos son estos.
```

De ahí salen **dos herramientas**. "Entender la situación" no es una herramienta
aparte: va fundida dentro de la primera (se le pasa la situación y devuelve la
acción, habiéndola entendido por dentro).

### Herramienta 1 — `siguiente_accion`

**Qué hace:** dado el estado de la startup y su contexto histórico (ambos
aportados por el agente), devuelve la actividad más recomendable a continuación.

**Entrada:**
- `situacion` (texto): la descripción de dónde está la startup, en lenguaje
  natural, tal como el agente la compone desde lo que el fundador cuenta.
- `contexto_historico` (texto, opcional): el historial consolidado que el agente
  mantiene en su memoria (qué se intentó, qué se validó, qué se falseó). Es
  *entrada*, no algo que startup-next recuerde. Opcional porque una startup nueva
  no tiene historial aún.

**Salida:** la actividad recomendada X (y, según lo que startup-next ya devuelve,
las recomendaciones asociadas con sus fuentes).

**Por dentro:** envuelve el flujo HTTP ya existente y verificado de startup-next
—`POST /informes/parse` para estructurar la situación, y el flujo de
`runs`/`start`/`poll` para obtener la recomendación del especialista—. Reutiliza
el cliente HTTP que ya se construyó y verificó contra producción (no se reescribe:
se transfiere del agente jubilado a este servidor).

### Herramienta 2 — `consejos_para_actividad`

**Qué hace:** dada una actividad concreta, devuelve los mejores consejos
disponibles para ejecutarla bien. La actividad puede ser la recomendada (X) o una
que el fundador ha decidido por su cuenta (Y), con independencia del estado de la
startup y aunque contradiga la metodología.

**Entrada:**
- `actividad` (texto): la acción a ejecutar. Da igual su origen —recomendada o
  soberana—; es solo un parámetro.
- `contexto_historico` (texto, opcional): el mismo historial, para situar los
  consejos.
- `actividad_recomendada` (texto, opcional): **la señal de tensión.** Si el agente
  sabe cuál era la actividad recomendada (X), la pasa aquí. El especialista compara
  `actividad` contra `actividad_recomendada`:
  - Si coinciden → contexto favorable, consejos "limpios".
  - Si difieren → contexto de tensión: el fundador va a hacer algo distinto de lo
    recomendado, y el especialista **modula sus consejos en consecuencia** —no da
    los mismos consejos con una advertencia pegada, sino consejos conscientes del
    riesgo de haberse apartado de X—.
  - Si se omite → el especialista aconseja sin señal de tensión (caso neutro).

**Salida:** los consejos de ejecución, modulados según la tensión, con la
honestidad epistémica del proyecto: cuando hay tensión, el especialista **incorpora
al consejo** su lectura de lo que la metodología opina del camino elegido —ni la
oculta (sería deshonesto) ni se niega a ayudar (sería paternalista)—. La autoridad
es del fundador; el conocimiento se pone a su servicio, con transparencia sobre la
distancia entre lo elegido y lo recomendado.

**Por dentro:** envuelve la capacidad de recuperación de conocimiento de
startup-next (RAG/OKF de los especialistas), pasándole la actividad y —cuando
existe— la señal de tensión como parte de la consulta.

**Asunción verificada:** este diseño asumía que el especialista, siendo un LLM
que razona sobre conocimiento recuperado, **modula de verdad** al recibir la
señal de tensión (`actividad` ≠ `actividad_recomendada`) — sin necesidad de
cambiar el mecanismo de recuperación, solo con darle esa señal en la consulta.
Era la asunción más razonable, pero era una asunción, y se nombró como tal hasta
verificarla: mandar una consulta con tensión y confirmar que los consejos
devueltos son genuinamente distintos y conscientes del riesgo, no los de siempre
con una coletilla. Verificado (commit `ceaec5e`): la modulación es real.

## Qué se reaprovecha y qué se jubila

**Se reaprovecha (del código actual, se transfiere a este servidor):**
- El **cliente HTTP a startup-next** (parse / runs / start / poll), verificado
  contra producción. Es el motor de las dos herramientas.
- El **entendimiento del flujo** (cómo se compone la consulta, cómo se interpreta
  la respuesta) que se ganó diseñando el agente — se traslada al diseño de las
  herramientas.

**Se jubila (Hermes Agent ya lo aporta de fábrica):**
- El entrypoint autónomo (`src/server.ts`), el canal stdin, la memoria OKF propia
  del agente, el adaptador LLM (Pieza 1), el empaquetado Docker propio. No se
  borra (es evidencia de proceso y su lógica informó este diseño), pero deja de
  ser el camino principal.

**Se conserva intacto:**
- startup-next entero, los siete especialistas, los mecanismos OKF/RAG, ontology-
  engine, la web, y la documentación sobre el conocimiento.

## startup-next no cambia su naturaleza

Un punto que el diseño protege: **startup-next sigue sin estado.** El servidor MCP
no le añade memoria ni le hace recordar startups. El contexto histórico entra como
parámetro en cada llamada y se procesa sin persistir. La memoria de cada startup
—sus experimentos, sus resultados— vive en el agente del emprendedor (Hermes), que
ya sabe recordar. Esta separación es la misma de siempre, ahora más nítida:
conocimiento compartido y sin estado en el centro; estado privado en el agente
distribuido.

## El aprendizaje asíncrono (horizonte, no este documento)

Los resultados que el agente acumula —respuestas de startup-next más los
resultados de los experimentos que el fundador realice— forman, en la memoria del
agente, un registro estructurado. Ese registro podría, **de forma asíncrona y
fuera del flujo conversacional**, alimentar de vuelta el conocimiento de
startup-next (mejorar sus especialistas RAG/OKF con la experiencia agregada de
muchas startups).

Esto es deliberadamente **un flujo separado** del servidor MCP de consulta:
- El servidor MCP de consulta es **read-only**: pregunta y responde, nada se
  escribe.
- El aprendizaje agregado es **otro mecanismo**, futuro, que cosecha registros
  estructurados fuera de banda. No forma parte de este diseño; se nombra para
  dejar claro que la arquitectura lo permite sin contaminar la consulta.

Separar "usar el conocimiento" (síncrono, read-only) de "generar conocimiento
nuevo" (asíncrono, agregado) mantiene ambos limpios. Es, además, el primer caso de
uso real del principio "aprender del conjunto sin ver al individuo" que gobierna la
ambición última del proyecto.

## Lo que el emprendedor configura

Para que su agente use este servidor, el emprendedor necesita, a alto nivel:
- El **comando** que lanza el servidor MCP (y sus argumentos).
- La **URL de startup-next** (el central desplegado) y la **credencial**
  (`API_KEY_HERMES` u homóloga) para autenticarse contra él.

La *sintaxis exacta* de dónde se pega esa configuración (el `config.yaml` de
Hermes, o `hermes mcp add`) la define Nous y puede cambiar entre versiones — queda
fuera del control de este proyecto. El documento define **qué** hay que configurar
(comando, URL, credencial); el **dónde** exacto es del host. Las instrucciones de
instalación darán el snippet de referencia para la versión actual de Hermes, sin
casarse con ella.

## Archivos que implica este diseño (para la implementación)

- Un proyecto/servidor MCP nuevo en TypeScript, con `@modelcontextprotocol/sdk` y
  `zod` (esquemas de entrada). Probablemente un repo o subcarpeta propia
  (`startup-next-mcp` o similar), no dentro del agente jubilado.
- Registro de las dos herramientas (`siguiente_accion`, `consejos_para_actividad`)
  con sus esquemas Zod.
- El cliente HTTP a startup-next, transferido del código actual.
- Transporte stdio, con **todo el logging a stderr**.
- Config por entorno (URL de startup-next, credencial).

## Pendientes explícitos, no resueltos aquí

**Confirmado — ya no es un pendiente:** la asunción de modulación del
especialista ante la señal de tensión se verificó (mismo commit `ceaec5e` que
verificó `consejos_para_actividad`): ante `actividad` ≠ `actividad_recomendada`,
los consejos devueltos son genuinamente distintos y conscientes del riesgo de
apartarse de lo recomendado, no los de siempre con una coletilla.

- **El aprendizaje asíncrono agregado** — mecanismo futuro, solo nombrado.
- **La sintaxis de instalación** para la versión concreta de Hermes Agent — se
  documenta al implementar, sin atarse a una versión.
- **Credenciales por-startup** contra startup-next — hoy la `API_KEY_HERMES` es
  compartida; el diseño per-instancia sigue siendo trabajo futuro, igual que en la
  arquitectura anterior.
- **Verificar contra el SDK real** los detalles exactos de registro de
  herramientas y forma de la respuesta MCP (`content: [{ type: "text", ... }]`),
  leyendo la documentación del SDK al implementar.
