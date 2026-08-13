# La expansión a siete especialistas de dominio

> Cómo startup-next pasó de dos especialistas de dominio a los siete que
> cubren hoy el ciclo de vida completo de una startup, y qué frontera
> distingue a cada uno.

## La taxonomía final

| Especialista | Cubre |
|---|---|
| `ideacion` | Ideación y modelo de negocio (Business Model Canvas) en su forma inicial |
| `mvp` | Prototipado, construcción del producto mínimo viable, economías de escala del producto |
| `pmf` | Encaje producto-mercado, desarrollo de clientes, Jobs To Be Done |
| `operaciones` | Cómo se organiza y ejecuta el trabajo interno, adopción de IA en la operación |
| `escalado` | Crecimiento defendible una vez hay encaje producto-mercado: motor de crecimiento, barreras competitivas |
| `plataformas` | Negocios de plataforma: dos o más lados de mercado, efectos de red, arranque en frío |
| `gobernanza` | Estructura legal/estatutaria y blindaje de la misión frente a la extracción de valor cortoplacista: gobernanza fiduciaria, clases de acciones, estructuras de holding |

Dos roles de la taxonomía original (`financiacion`, `administracion`)
desaparecieron por completo — no caen a "sin especialista" en tiempo de
ejecución, se volvieron estructuralmente irrepresentables: el orquestador
que decide a qué especialista enrutar cada tarea no puede emitirlos nunca
más, porque ya no existen como opción posible.

## Una única fuente de verdad para "qué especialista está implementado hoy"

Cada especialista nuevo requiere tocar más de un punto del sistema: el
prompt del orquestador debe saber que existe, el enrutador interno debe
saber a qué función despachar la tarea, y la respuesta al fundador debe
reflejar cuál atendió el caso. Mantener esa lista de especialistas
implementados sincronizada a mano en varios lugares independientes ya
había causado un error real una vez (un punto del sistema reconocía un rol
nuevo mientras otro seguía asumiendo el rol anterior por defecto). La
corrección fue estructural, no puntual: una única fuente de verdad —un
conjunto explícito de qué roles están realmente implementados— que el
resto del sistema consulta, en vez de que cada punto mantenga su propia
copia de la lista.

## El prompt del orquestador: fronteras explícitas, no implícitas

La frontera entre especialistas vecinos no queda a criterio del modelo en
cada caso: está escrita, versionada, una frase por especialista:

> - **ideacion**: el problema, el cliente o el segmento todavía no están
>   validados, o la tarea es diseñar/ajustar el modelo de negocio en su
>   forma inicial — antes de construir nada.
> - **mvp**: construir y probar una primera versión real del producto
>   (prototipado, experimentos, métricas) — no todavía escalar el negocio.
> - **pmf**: ya existe un producto y clientes reales; la tarea es validar
>   o mejorar el encaje producto-mercado — no construir el producto por
>   primera vez (eso es mvp) ni escalar (eso es escalado).
> - **operaciones**: cómo se organiza y ejecuta el trabajo interno una vez
>   el negocio funciona — no la estrategia de crecimiento externo
>   (escalado) ni la validación de mercado (pmf).
> - **escalado**: crecer de forma defendible una vez hay encaje
>   producto-mercado — no la operación interna del día a día
>   (operaciones).
> - **plataformas**: el negocio en sí es una plataforma (dos o más lados
>   de mercado, efectos de red, arranque en frío). Es transversal: si la
>   tarea trata específicamente la dinámica de plataforma, se elige este
>   especialista aunque la startup también esté en fase de ideación o
>   escalado; si no, se clasifica por fase como de costumbre aunque el
>   negocio sea una plataforma.
> - **gobernanza**: la tarea trata la estructura legal/estatutaria de la
>   empresa o cómo blindar su misión frente a la extracción de valor
>   cortoplacista (composición y deber fiduciario del directorio, clases
>   de acciones y derechos de voto, estructuras de holding o fideicomiso,
>   protección ante adquisiciones hostiles o presión de inversores) — no
>   el diseño del modelo de negocio en sí (eso es ideacion) ni el motor de
>   crecimiento del negocio (eso es escalado), aunque la tarea surja en
>   cualquier fase de la startup.

## Las fuentes de conocimiento, y cómo se repartieron

Cada especialista se fundamenta en una o más fuentes de conocimiento
reexpresado. Algunas fuentes son transversales a varios especialistas a la
vez:

- ***7 Powers*** (Hamilton Helmer) es la fuente más transversal: cada uno
  de sus siete "poderes" (barreras competitivas) se repartió a un
  especialista según a qué fase de la startup aplica mejor — economías de
  escala a `mvp`, efectos de red a `plataformas`, contraposicionamiento y
  costes de cambio y creación de marca y recurso acorralado a `escalado`,
  y poder del proceso a `operaciones`. Ningún capítulo del libro quedó sin
  asignar.
- ***Jobs to be Done*** (Anthony Ulwick) alimenta a `pmf`.
- ***Platform Scale*** (Sangeet Paul Choudary) alimenta a `plataformas`,
  elegida sobre otras referencias del mismo campo por ser más operativa y
  accionable (diseño de la interacción central, arranque en frío, bucles
  de crecimiento) frente a un enfoque más panorámico.
- Varios artículos sobre organizaciones nativas de IA alimentan a
  `operaciones` — el terreno más nuevo y menos consolidado de los siete,
  con el conocimiento marcado explícitamente como emergente, no
  verificado con el mismo rigor que el resto.
- ***Incorruptible*** (Eric Ries) alimenta a `gobernanza`, con status
  `Verified` — a diferencia de `operaciones`, es una fuente publicada con
  ISBN real, no una categoría todavía en formación.

Donde una fuente ya existente en el sistema cubría parcialmente a un
especialista nuevo, se reetiquetó de forma aditiva (nunca se le quita
cobertura a un especialista existente para dársela a uno nuevo) — el
mismo contenido puede servir a más de un especialista a la vez.

## Orden de construcción: uno a la vez, con criterio explícito

Los especialistas nuevos no se construyeron todos de una vez ni en
paralelo. El orden se decidió por tres criterios: qué fuente ya estaba
disponible frente a cuál requería trabajo de preparación externa, qué
frontera con especialistas ya construidos era más nítida (para no
contaminar la medición de los siguientes con errores de enrutamiento
tempranos), y qué especialistas podían compartir el trabajo de incorporar
una misma fuente nueva. Cada especialista se verificó de punta a punta
contra producción antes de empezar el siguiente.

## Verificación

El cambio al prompt del orquestador es compartido por los siete roles —
cualquier tarea, incluidas las que atienden los especialistas más
antiguos, pasa por el prompt actualizado. Por eso la verificación de cada
especialista nuevo nunca fue aislada: exige reconfirmar, cada vez, que los
especialistas ya existentes siguen enrutando y respondiendo igual que
antes. Para cada especialista nuevo, el mínimo exigido fue un caso real o
realista contra producción que cayera inequívocamente en su frontera, con
confirmación de que el especialista correcto atendió la tarea y de que las
fuentes citadas en la recomendación final vienen realmente del conocimiento
recién incorporado — no alcanza con contar que el reetiquetado quedó bien
en la base de datos, hace falta confirmar que el especialista efectivamente
lo recupera y lo cita.
