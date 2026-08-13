# Observabilidad

> Qué se puede observar del sistema, desde dónde, y respetando qué límites. Un
> rediseño conceptual: el pivote a una arquitectura distribuida cambió la frontera
> de lo observable. Diseñado como dirección; la instrumentación completa es
> horizonte.

---

## El límite cambió

Un sistema agéntico de producción necesita observabilidad: saber qué hace por
dentro, no fiarse de que un "200 OK" signifique que todo fue bien. Una llamada puede
devolver éxito y esconder una alucinación, un dato incorrecto o una lógica errónea.
Observar es separar el estado de ejecución del estado del negocio.

Pero la arquitectura del sistema está repartida en dos partes, y eso define qué es
observable y por quién:

- **El centro** —el conocimiento especializado (startup-next), el servidor MCP,
  ontology-engine— es nuestro, compartido, y corre en infraestructura que operamos.
  Es lo que **podemos y debemos** instrumentar.
- **El borde** —el agente personal del emprendedor (Hermes), su memoria, sus
  conversaciones— corre en el entorno del emprendedor, bajo su control. No es nuestro,
  y **no debemos** querer observarlo.

Esta división no es una limitación técnica: es una decisión de confidencialidad. La
misma frontera que hace que la memoria de la startup viva en el borde (y nunca en el
centro) hace que la observabilidad se detenga en el borde. Observar lo que pasa
dentro del agente del emprendedor sería mirar dentro de su casa. No lo hacemos.

---

## Qué observamos: el centro

Del lado central, lo observable y valioso:

- **El servidor MCP** — qué herramienta se invoca (`siguiente_accion`,
  `consejos_para_actividad`), con qué forma de entrada, con qué resultado. La puerta
  por la que el conocimiento se consume.
- **El conocimiento (startup-next)** — qué especialista se activa, qué conocimiento
  se recupera, latencias, fallos, y —cuando importe— el coste por consulta.
- **ontology-engine** — su parte del trabajo, cuando participa.

El objetivo es el de siempre en observabilidad seria: registrar lo que el sistema
genera, las entradas y salidas de cada herramienta, y el coste por segmento —
separando el estado de ejecución (¿funcionó la mecánica?) del estado del negocio
(¿fue buena la recomendación?). Un registro que permita responder no solo "¿respondió?"
sino "¿respondió *bien*, y a qué coste?".

---

## Qué NO observamos: el borde

El agente del emprendedor ya trae su propia observabilidad — los agentes personales
maduros registran su actividad para su dueño. Esa observabilidad es **del
emprendedor, para el emprendedor**: él ve qué hace su agente, qué recuerda, qué
decide. Nosotros no.

Que parte de la observabilidad la aporte el agente anfitrión no es un hueco en
nuestro diseño: es el reparto correcto. Cada uno observa su lado. El emprendedor, su
agente y su memoria; nosotros, el conocimiento central que servimos. Ninguno mira
dentro del otro.

---

## El principio ya está; la instrumentación es horizonte

Una parte de la observabilidad no espera a ninguna herramienta: es un principio que
ya gobierna el código. **Fallo ruidoso, nunca silencioso.** El sistema está construido
para que un error se note, no para que un fallo se disfrace de éxito. La degradación
silenciosa —conocimiento que se recupera vacío sin lanzar error— se trata como un
defecto a cazar, no como algo tolerable. Ese principio es observabilidad en su forma
más básica y más importante: que el sistema no mienta sobre su propio estado.

La instrumentación completa —el registro estructurado, con una herramienta como
Langfuse (autoalojable, para no ceder los datos a un tercero), del que salga el
análisis de coste, latencia y calidad por consulta— está **diseñada como dirección,
no implementada**. Es uno de los vectores de evolución del proyecto: pasar del
principio (fallo ruidoso, ya vigente) a la instrumentación (medición sistemática,
horizonte).

Dicho con la honestidad del recorrido: hoy el sistema falla de forma ruidosa y es
depurable; mañana, instrumentado, será medible. Lo primero está; lo segundo tiene
dirección.
