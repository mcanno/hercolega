# El estado, jamás en el centro

> Por qué el servicio central de conocimiento no guarda nada de ninguna startup —y por qué esa decisión es el cimiento de toda la arquitectura.

## Una decisión que parece técnica y es fundacional

En algún momento del desarrollo, el servicio central de razonamiento del sistema —el que aplica el conocimiento estructurado para deducir prerrequisitos y relaciones— guardaba estado: contenía datos concretos de las startups que lo consultaban. Era lo natural: un servicio que razona sobre casos parece que debería recordar los casos sobre los que razona.

Se decidió retirar todo ese estado. El servicio central quedó reducido a **conocimiento puro sin memoria de nadie**: sabe cómo se relacionan los conceptos de una metodología, pero no sabe nada de ninguna empresa concreta, y no guarda rastro de las consultas que atiende. Esa decisión, que en el momento pudo parecer una limpieza técnica, resultó ser el cimiento sobre el que se sostiene toda la arquitectura distribuida del proyecto. Sin ella, el resto no habría sido posible.

## La distinción entre conocimiento y estado

Para ver por qué, hay que separar dos cosas que un sistema de este tipo maneja y que es tentador mezclar:

- El **conocimiento** es general y compartible: cómo se construye una startup, qué implica cada fase, cómo se relacionan los conceptos de una metodología. No pertenece a nadie en particular; sirve a todos por igual. Es lo que hace valioso a un especialista.

- El **estado** es particular y privado: en qué punto está *esta* startup, qué ha probado, qué ha validado, qué ha falseado. Pertenece a una empresa concreta, y es lo más sensible que tiene —su situación real, su ventaja, su intimidad estratégica—.

La confusión entre ambos es el error de diseño que la retirada del estado vino a corregir. Un servicio central que mezcla conocimiento (compartido) con estado (privado de cada empresa) es un servicio que acumula, en un solo lugar, los datos sensibles de todas las startups que lo usan. Ese punto central se convierte en el mayor riesgo de confidencialidad del sistema: un único sitio donde, si algo falla, se filtra la información de todos.

## La regla que quedó establecida

La decisión cristalizó en una regla simple y tajante, que gobierna toda la arquitectura:

**El conocimiento es compartido y vive en el centro. El estado es privado y jamás vive en el centro.**

El servicio central contiene conocimiento y solo conocimiento. Todo el estado —la memoria de cada startup— vive exclusivamente en su Hermes, distribuido, en el entorno del emprendedor. El centro razona sobre lo que se le pregunta y olvida; no recuerda a quién atendió ni qué le dijo.

## Por qué esto habilita todo lo demás

Esta regla es la que hace posible la arquitectura distribuida del proyecto —N instancias de Hermes, una por empresa, con estado físicamente separado, más un servicio central compartido y sin estado—. Si el centro guardara estado, la separación física de los Hermes sería una ilusión: los datos volverían a juntarse en el punto central en cuanto cada Hermes lo consultara. La confidencialidad por construcción, que es la promesa central del sistema, exige que el punto compartido no retenga nada.

Dicho de otro modo: **la separación física de las instancias solo garantiza la confidencialidad si el punto donde todas convergen no acumula lo que ellas separan.** Retirar el estado del centro fue quitar el único lugar donde los datos de empresas distintas podrían haberse encontrado.

Hay una elegancia en cómo esta decisión encaja hacia atrás. Se tomó, en su momento, por una razón local —simplificar un servicio, reducir su superficie de riesgo—. Solo después, al definirse la arquitectura distribuida completa, se vio que había sido su prerrequisito silencioso. Las decisiones de diseño más sólidas suelen tener esa cualidad: resuelven un problema inmediato y, sin proponérselo del todo, abren la puerta a algo mayor.

## El centro como servicio de consulta

Lo que queda, tras la retirada del estado, es un servicio central que se comporta como una biblioteca de consulta, no como un archivo de expedientes. Se le pregunta —"dado este concepto, ¿qué lo precede, con qué se relaciona?"— y responde desde su conocimiento estructurado, sin registrar quién preguntó ni sobre qué caso. Cada consulta es independiente y sin huella. El conocimiento fluye hacia quien lo pide; nada del que pide se queda en el centro.

Esa asimetría —el conocimiento sale, el estado no entra— es la forma que toma, en la infraestructura, el principio de confidencialidad del proyecto. El centro sabe mucho y no recuerda a nadie. Cada Hermes recuerda todo de su startup y consulta al centro cuando necesita profundidad que no tiene. Ninguno de los dos invade el terreno del otro.

---

Para el detalle técnico completo de esta decisión —el inventario de endpoints, los dos consumidores que dependían del estado retirado, y cómo se migró sin romper el servicio compartido— ver [ontology-engine-diseno.md](ontology-engine-diseno.md).
