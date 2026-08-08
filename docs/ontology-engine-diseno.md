# ontology-engine como servicio puramente de consulta

> Documento técnico. Para la versión divulgativa de esta misma decisión,
> ver [docs/ontology-engine.md](ontology-engine.md).

## El problema

El servicio central de razonamiento metodológico (`ontology-engine`) no
tenía autenticación y persistía hechos reales de startups reales en una
base de datos compartida, sin control de acceso — una situación que
contradecía directamente la visión de confidencialidad del proyecto. La
decisión de fondo: el servicio queda reducido a conocimiento puro de
metodología (qué precede a qué, qué se relaciona con qué en el marco Lean
Startup), sin guardar ningún hecho concreto de ninguna startup real.

En términos técnicos, esto es la distinción entre **TBox** (la
terminología y estructura de una metodología: qué es una hipótesis, qué la
precede) y **ABox** (los hechos concretos de un caso real: qué hipótesis
tiene *esta* startup, si ya la validó). El servicio se queda con el TBox y
retira por completo el ABox.

## Inventario de endpoints: qué se mantiene, qué se retira

El criterio es directo: los endpoints que consultan la terminología general
(TBox) se mantienen; los que leen o escriben hechos de una startup
concreta (ABox) se retiran.

**Se mantienen** (TBox, sin datos de ninguna startup):
- Estado del servicio.
- Listado de conceptos de la metodología y sus relaciones.
- Subtipos de un concepto (por ejemplo, los subtipos de "Hipótesis").
- Prerrequisitos de un concepto — el endpoint central: es lo que
  consulta startup-next para informar, sin forzar ni juzgar, qué pasos
  metodológicos suele preceder a la tarea que un fundador está por
  encarar.

**Se retiran** (ABox, hechos de startups reales):
- Lectura del grafo de hechos de una startup concreta.
- Lectura de vecinos de un nodo dentro de ese grafo.
- Validación de una startup concreta contra las reglas metodológicas
  (el "modo enriquecido").
- Escritura de hechos nuevos sobre una startup concreta.

## Un hallazgo que cambió el alcance: dos consumidores, no uno

El primer marco del problema asumía un solo consumidor del ABox: el "modo
enriquecido" de startup-next, activado cuando un fundador aportaba un
informe con una identidad de startup verificable. Ese consumidor ya tenía
solución: la memoria privada de Hermes (ver
[docs/vision.md](vision.md) y [docs/ciclo-completo.md](ciclo-completo.md))
reemplaza esa función sin depender en absoluto de un servicio central —
la verdad sobre una startup concreta vive en su propio Hermes, no en un
servicio compartido.

Pero había un **segundo consumidor**, independiente de Hermes: la propia
aplicación de diagnóstico inicial (`startup-advisor`) llamaba al motor de
razonamiento directamente desde su flujo de entrevista, para escribir
hechos reales durante la conversación con el fundador y validar contra
ellos, incorporando el resultado como consideraciones metodológicas dentro
del informe que el fundador recibe al final de la entrevista. Retirar el
ABox sin decidir qué pasa con este segundo consumidor lo habría dejado
fallando en cada entrevista.

## La decisión: el mismo patrón TBox-only para los dos consumidores

Entre las opciones consideradas —desaparecer sin reemplazo, o reemplazar
solo para el camino mediado por Hermes (que de todas formas ya no lo
necesitaba)— la que se adoptó fue una tercera: aplicar al segundo
consumidor el mismo patrón que startup-next ya usa para su propio modo
sin hechos reales. En vez de escribir hechos de la entrevista y validar
contra ellos, la aplicación de diagnóstico consulta los prerrequisitos
genéricos de los conceptos que la entrevista tocó, y los presenta como
consideraciones metodológicas generales — encuadradas explícitamente como
orden metodológico genérico, no como una evaluación de hechos reales de
esa startup. Es un cambio real de comportamiento (deja de avisar cuando un
fundador *concretamente* se saltó un paso, porque ya no hay hechos reales
contra qué comparar), pero no introduce ningún mecanismo nuevo: reusa uno
que ya estaba en producción y verificado.

## El efecto colateral: el mecanismo de firma de PDF pierde su propósito

Antes de este cambio, un informe podía exportarse como PDF firmado
criptográficamente, y ese PDF permitía a startup-next extraer una
identidad real de startup para activar el modo enriquecido — el único
propósito downstream de esa identidad real era consultar el ABox. Con el
ABox retirado por completo, la extracción de identidad real de un PDF deja
de habilitar nada: el mecanismo de firma se convierte en código sin
propósito. La decisión fue retirarlo por completo, no dejarlo inactivo —
mantener un mecanismo de firma y verificación criptográfica que ya no
protege nada contradice el resto de los criterios del proyecto.

## Plan de migración: los consumidores primero, el servicio después

El orden de migración se pensó para que el servicio compartido nunca
quedara roto a mitad de camino, dado que dos aplicaciones en producción
dependían de él simultáneamente:

1. Vaciar los datos reales de startups del servicio (paso ya ejecutado y
   verificado por separado, antes de tocar ningún código de los
   consumidores) — esto deja a ambas aplicaciones corriendo con
   degradación elegante, sin cambiar una sola línea de código todavía.
2. Actualizar los dos consumidores para que dejen de depender del ABox,
   cada uno con la solución que le corresponde (memoria privada de Hermes
   para startup-next, el patrón TBox-only para la aplicación de
   diagnóstico), aprovechando la ventana segura que dejó el paso anterior.
3. Recién entonces, retirar del propio servicio los endpoints y el código
   de ABox que ya no tienen ningún consumidor.
4. Decidir por separado si las tablas de la base de datos que guardaban
   esos hechos se eliminan del esquema o se dejan vacías.

## Verificación

El mismo estándar que el resto del proyecto: evidencia real contra
producción, no solo que el código compile.

- Los endpoints de TBox siguen respondiendo con normalidad tras el
  despliegue.
- Los endpoints retirados devuelven una respuesta de "no encontrado", no
  un error de servidor sin manejar.
- Una tarea real contra startup-next confirma que ya no existe ninguna
  rama de "modo enriquecido" — el sistema opera en un único modo,
  siempre.
- Una entrevista real (o claramente marcada como de prueba) contra la
  aplicación de diagnóstico se completa sin error visible para el
  fundador, con el informe guardándose correctamente.
