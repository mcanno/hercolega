# Observabilidad en Hermes-Colega

> Cómo se observa un sistema agéntico distribuido sin traicionar la confidencialidad que lo define.

## Por qué este documento existe

Un agente de IA no falla como el software tradicional. El software clásico se rompe de forma ruidosa: lanza una excepción, devuelve un error, se cae. Un agente falla en silencio. Devuelve un código 200, entrega una respuesta plausible, y sin embargo ha activado el especialista equivocado, ha razonado sobre un contexto vacío, o ha fundado su recomendación en algo que no dijo la fuente. Nada en la superficie delata el fallo. El fundador recibe un consejo con el mismo aspecto de siempre, y lo sigue.

Hermes-Colega se sostiene sobre un principio de honestidad epistémica: el sistema distingue lo validado de lo emergente, marca lo que no sabe, no finge certeza. Ese principio, hasta aquí, gobierna el *contenido* del consejo. La observabilidad es el mismo principio aplicado al *funcionamiento del agente*. Un sistema que predica honestidad sobre lo que sabe debe poder demostrar honestidad sobre cómo opera. Sin observabilidad, la honestidad de Hermes es una afirmación; con ella, es verificable.

## Qué debe observar un sistema agéntico

La observabilidad de agentes no es la monitorización de un servicio web. No basta con latencia, tasa de error y uso de CPU. Un agente toma decisiones en árbol —elige un camino, invoca herramientas, encadena subpasos— y cada nudo de ese árbol es un punto donde puede desviarse sin síntoma visible. Un sistema de observabilidad adecuado debe ofrecer, como mínimo, cinco funciones:

1. **Rastreo del árbol de ejecución.** Reconstruir la traza completa de una decisión: qué se preguntó, qué especialista se activó, qué conocimiento se recuperó, cómo se compuso la respuesta, qué validaciones pasó. No un registro plano de eventos, sino el árbol con su estructura.

2. **Análisis de trazas.** Detectar patrones de fallo a través de muchas ejecuciones: un especialista que falla más de lo normal, un mecanismo de recuperación que devuelve resultados vacíos, una latencia que se degrada.

3. **Evaluación de la calidad de la respuesta.** Juzgar si una recomendación es relevante, si está fundada en las fuentes que cita, si no ha alucinado. La técnica habitual es usar un modelo de lenguaje como evaluador ("LLM como juez"), aplicado sobre las trazas.

4. **Versionado de las instrucciones.** Los *prompts* que gobiernan el comportamiento del agente evolucionan. Saber qué versión produjo qué comportamiento es imprescindible para mejorar sin regresiones.

5. **Monitorización con alertas.** En operación real, detectar y avisar cuando algo se desvía —no descubrirlo semanas después.

Estas funciones definen el *qué*. La parte difícil, en un sistema como este, es el *dónde* y el *para quién*.

## El problema propio de Hermes: observar sin ver

La arquitectura de Hermes-Colega es distribuida por diseño. No hay un Hermes central que atienda a muchas startups; hay N instancias de Hermes, una por emprendimiento, cada una en el entorno del emprendedor —en Docker o en local—, con su memoria física y privada. Esa separación no es una decisión de implementación: es la garantía de confidencialidad. Dos empresas no pueden filtrarse datos porque sus Hermes ni siquiera están en la misma máquina.

Esa misma arquitectura convierte la observabilidad en un problema no trivial. En un SaaS centralizado, observar es fácil: todo pasa por tu servidor, así que lo ves todo. Aquí, el comportamiento de cada agente ocurre en una máquina que no controlas y cuyos datos no debes ver. La pregunta es incómoda y central:

**¿Cómo se mejora el sistema agéntico —que exige aprender de cómo se comporta en producción— sin observar los datos privados de ninguna startup?**

La respuesta parte de una distinción que suena obvia pero que lo cambia todo: **no todo lo que produce un agente tiene el mismo grado de confidencialidad.**

- Los **datos de contenido** —la memoria de la startup, las conversaciones del fundador, sus experimentos, qué validó y qué falseó— son sagrados. Nunca salen del Hermes del emprendedor.
- Los **datos de comportamiento** —qué especialista se activó, cuántos pasos dio el árbol, si una recuperación vino vacía, la latencia, si el validador reintentó— describen *cómo funcionó el agente*, no *sobre qué trabajó*. Y esto puede describirse sin exponer el contenido.

La observabilidad que se necesita para depurar y mejorar el agente vive, casi por completo, en la segunda categoría. Para saber que "el especialista de escalado se activó, recuperó cuatro conceptos, el validador aprobó al segundo intento, en tres segundos", no hace falta saber de qué startup era ni qué decía su memoria. El *cómo funcionó* es separable del *sobre qué trabajó*. Esa separabilidad es lo que hace posible todo lo que sigue.

## Una arquitectura de observabilidad en tres capas

### Capa 1 — Observabilidad local, completa, privada

Cada instancia de Hermes incluye su propio sistema de observabilidad, integrado en el contenedor, siempre activo. Es completo: tiene acceso a todo, incluido el contenido, porque es la herramienta con la que el propio emprendedor —o quien le dé soporte— depura *su* agente. Aquí se ve la traza entera de una recomendación, con su contexto real, para entender a fondo un caso concreto.

Esta capa nunca emite nada al exterior. Es del emprendedor, para el emprendedor, en su entorno. Que forme parte integral de la instalación no es accesorio: un colega que no se puede observar no es un colega en el que se pueda confiar. La observabilidad local es parte de lo que hace a Hermes digno de confianza, igual que su memoria o su criterio.

Como el sistema de observabilidad elegido es de código abierto y autoalojable, esta capa no depende de ningún servicio externo ni envía trazas a la nube de un tercero. Vive donde vive el agente.

### Capa 2 — Telemetría agregada, opcional, anonimizada en origen

Las instancias cuyos dueños lo autoricen —y solo esas— emiten hacia el mantenedor del sistema un flujo de **eventos de comportamiento**, con tres propiedades que lo hacen honesto:

- **Anonimizado en origen.** La anonimización ocurre *dentro* del Hermes del emprendedor, antes de que nada salga de su máquina. No se emite un dato identificable que un servidor central anonimiza después. Lo que cruza la frontera de la máquina ya viene despojado de identidad. La diferencia es decisiva: con anonimización en destino, el emprendedor tiene que *confiar* en que el mantenedor cumple; con anonimización en origen, sobre código abierto que corre en su propia máquina, puede *verificar* que se cumple. Es la misma lógica que la separación física de las instancias: confidencialidad por construcción, no por promesa.

- **Solo metadata de comportamiento, jamás contenido.** La línea es nítida a propósito, porque una línea difusa se erosiona con el tiempo y una nítida se sostiene:

  | Puede salir | Nunca sale |
  |---|---|
  | Especialista activado | Texto de la consulta del fundador |
  | Mecanismo usado (RAG / OKF) | Texto de las recomendaciones |
  | IDs de conceptos recuperados *(ver nota)* | Contenido de la memoria |
  | Si el ancla se seleccionó y si fue coherente | Identificador de la startup |
  | Reintentos del validador | Cualquier dato que reidentifique |
  | Estado terminal (aprobado / rechazado) | |
  | Latencias | |
  | Conceptos recuperados vacíos (fallos silenciosos) | |
  | Fase o dimensión de la consulta | |

- **Sin hilo de sesión.** Esta es la pieza más fina del diseño, y merece explicarse. El riesgo de reidentificación no viene de saber que "alguien recuperó el concepto de contraposicionamiento" —eso es inocuo—. Viene de poder reconstruir el *patrón temporal* de una startup concreta: "esta instancia consultó contraposicionamiento, luego costes de cambio, luego financiación, luego gobernanza". Esa *secuencia* dibuja la situación estratégica de una empresa identificable. Por eso los eventos se emiten **desacoplados**: llegan como hechos sueltos, agregados con los de muchas otras sesiones, sin ningún identificador que permita coser dos eventos como pertenecientes a la misma instancia. El mantenedor ve el bosque —qué conceptos se recuperan bien o mal en todo el sistema— pero no puede seguir el rastro de ningún árbol individual. Los IDs de concepto salen, y con ellos la utilidad de depuración; el hilo que los convertiría en un perfil, no.

  Este desacople tiene un coste consciente: en agregado se pierde la capacidad de depurar *trayectorias completas* de principio a fin. Pero esa depuración es precisamente para lo que sirve la Capa 1, que tiene todo el contexto de un caso. Cada capa hace lo suyo: la local, para entender un caso a fondo; la agregada, para ver el sistema. La división es limpia.

Esta capa está **desactivada por defecto**. Un Hermes recién instalado no emite nada. El emprendedor decide activarla, de forma granular —qué categorías de evento comparte— y revocable en cualquier momento. La autoridad sobre los datos, como la autoridad sobre las decisiones, es del fundador.

### Capa 3 — Observabilidad del sistema

Con el flujo agregado y anónimo de la Capa 2, el mantenedor del sistema observa el comportamiento del sistema agéntico *completo*: si la selección de ancla acierta en promedio, qué especialista concentra más fallos, si la poda del grafo deja fuera conceptos que luego se echan en falta, si aparece un patrón de recuperaciones vacías —esos fallos silenciosos detectados ahora en agregado, no a mano—. El sistema mejora para todas las instancias sin que su mantenedor haya visto los datos de ninguna.

## Los principios que lo hacen honesto

Un mecanismo de telemetría puede ser una herramienta legítima de mejora o una vigilancia con buena cara. La diferencia está en los detalles, y aquí los detalles se rigen por los mismos principios que gobiernan el resto del proyecto:

- **Anonimización en origen, verificable.** Ya expuesto: el emprendedor no confía, verifica, sobre código que corre en su máquina.

- **Transparencia sobre lo que se emite.** Cada instancia mantiene un registro local visible de la telemetría saliente. El emprendedor puede ver, en cualquier momento, exactamente qué eventos se han enviado. Nada oculto. Es la cuarta cláusula del principio rector del proyecto —transparencia sin agendas ocultas— aplicada a la telemetría.

- **Consentimiento real, granular y revocable.** No un "acepto todo" enterrado en un instalador. Una decisión explícita, por categorías, que se puede retirar.

- **El contenido, nunca —ni siquiera anonimizado.** La tentación futura será emitir "el texto de la recomendación, pero sin el nombre de la empresa". No se hará. El texto de una recomendación puede contener detalles que reidentifican por sí solos: el segmento, el producto, la situación. La línea defendible es solo metadata estructural, jamás texto libre.

## Elección de herramienta

Los criterios de elección no son los de una empresa cualquiera: los impone la arquitectura de este proyecto.

1. **Autoalojable y de código abierto.** No por ideología, sino por necesidad estructural. Si cada Hermes corre en el entorno del emprendedor con datos privados, un sistema de observabilidad comercial y cerrado que enviara las trazas a la nube de un tercero contradiría de raíz el principio de que los datos de cada startup no salen de su entorno. Los datos de observabilidad de un Hermes son tan confidenciales como su memoria: tienen que poder quedarse en local. Este criterio descarta, para el núcleo, las plataformas comerciales sin versión autoalojable.

2. **Licencia permisiva.** "Código abierto" no es todo igual. Entre los candidatos autoalojables, las licencias van desde las más restrictivas (Elastic 2.0) hasta las más permisivas (MIT). Para un sistema pensado para ser distribuido e instalado por muchos emprendedores —y potencialmente provisto por organizaciones de apoyo—, la licencia más permisiva reduce la fricción legal futura.

3. **Compatibilidad con el stack.** El backend agéntico está construido sobre LangGraph. La herramienta debe integrarse sin fricción.

Con esos criterios, la recomendación principal es **Langfuse**: su núcleo completo —rastreo, evaluaciones, gestión de *prompts*— es autoalojable bajo licencia MIT, la más permisiva de las disponibles, y es compatible con cualquier framework, incluido LangGraph. Para un modelo distribuido donde cada instancia puede necesitar su propia observabilidad local, la permisividad de la licencia es la que menos problemas dará.

## Una conexión que trasciende la observabilidad

Vale la pena hacer explícito algo que el diseño anterior insinúa. El problema que resuelve la telemetría agregada —aprender del comportamiento de muchas instancias sin ver los datos de ninguna— es, en su estructura, el mismo problema que plantea la ambición última del proyecto: destilar el conocimiento de frontera de la startup nativa de IA a partir de la experiencia de muchas startups, sin que ninguna exponga lo suyo.

La observabilidad con consentimiento es la versión de infraestructura de esa idea, y la más sencilla de las dos, porque la metadata de comportamiento es intrínsecamente menos sensible que el conocimiento de negocio. Resolver bien la telemetría agregada es, por eso, algo más que una herramienta de depuración: es el primer ensayo real del principio que gobierna la aspiración máxima del sistema —aprender del conjunto sin ver al individuo—. Lo que aquí se prueba con métricas de comportamiento, algún día habrá de aplicarse al conocimiento mismo.

## Estado

El diseño de observabilidad aquí descrito está definido, no implementado. La observabilidad local (Capa 1) es directa: integrar una herramienta autoalojable en el contenedor de Hermes. La telemetría agregada (Capas 2 y 3) tiene una pieza que exige trabajo cuidadoso y honesto: garantizar que la metadata emitida es verdaderamente segura —que ningún campo aparentemente inocuo, ni su combinación, permite reidentificar—. Definir con precisión qué es "metadata segura", y demostrar que el desacople de sesión resiste, es un problema técnico real que este documento plantea pero no cierra. Reconocer dónde termina el diseño y empieza lo que queda por resolver es coherente con el resto del proyecto.
