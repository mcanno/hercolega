# Dónde piensa cada cosa

> Por qué Hermes necesita un modelo de lenguaje propio, qué le pide exactamente, y por qué eso refuerza la confidencialidad en lugar de comprometerla.

## Una pregunta que parece incómoda

Hermes-Colega es local-first: cada instancia corre en el entorno del emprendedor, con su memoria privada, y el conocimiento profundo vive en un servicio central compartido. Pero Hermes, para funcionar, necesita "pensar" —entender lo que el fundador le cuenta, componer su memoria—. Y pensar, en un sistema de IA, significa usar un modelo de lenguaje.

Eso plantea una pregunta que parece incómoda: si Hermes usa un modelo de lenguaje, ¿no están los datos privados del fundador pasando por ese modelo? Y si el modelo es de un proveedor externo, ¿no se rompe entonces la promesa de confidencialidad?

La respuesta está en distinguir **qué tipo de pensar** hace cada parte del sistema. Porque no todo el pensamiento de Hermes-Colega ocurre en el mismo sitio, ni pesa lo mismo.

## Dos clases de inteligencia, en dos lugares

Hay dos tareas muy distintas que un sistema como este realiza, y conviene no confundirlas:

**La inteligencia especializada** —generar la recomendación estratégica, aplicar la metodología, razonar sobre los siete poderes o el desarrollo de clientes— es la parte pesada. Requiere conocimiento profundo de dominio y capacidad de razonamiento sofisticada. **Esta inteligencia vive en el servicio central**, en los especialistas. No es Hermes quien la ejerce.

**La inteligencia doméstica** —las dos tareas que Hermes hace en casa— es ligera. Son solo dos:

- **Entender al fundador.** Cuando el fundador cuenta, en lenguaje natural, "hice las entrevistas que me recomendaste pero no logré validar la propuesta de valor", alguien tiene que comprender ese texto y decidir a qué recomendación pendiente se refiere y qué resultado reporta —si la validó, la falseó, o avanzó sin validar—. Es una tarea de comprensión y clasificación: emparejar un texto con una de varias opciones conocidas.

- **Consolidar la memoria.** Antes de consultar al central, Hermes compone un resumen coherente de su historial —qué se ha intentado, qué ha funcionado, qué no— para pasarlo como contexto. Es una tarea de síntesis: reducir fragmentos dispersos a un resumen ordenado.

Estas dos tareas necesitan un modelo de lenguaje, sí. Pero son tareas *domésticas*: clasificar y resumir, no generar estrategia. Y esa diferencia lo cambia todo.

## Por qué el reparto refuerza la confidencialidad

Aquí está la consecuencia que resuelve la pregunta incómoda del principio.

Como la inteligencia pesada vive en el central y solo la ligera vive en Hermes, **el modelo de lenguaje que Hermes necesita no tiene que ser grande ni potente.** Clasificar un texto contra unas opciones, o resumir un historial, lo hace bien un modelo modesto —incluso uno pequeño corriendo en la propia máquina del emprendedor—.

Y eso hace que el local-first deje de ser una aspiración y se vuelva realista. El emprendedor **elige el modelo que usa Hermes**, y puede elegir uno local: un modelo que corre en su propio ordenador, al que los datos del fundador nunca salen de la máquina. No necesita contratar una API cara ni mandar su información a ningún proveedor, porque la tarea que ese modelo tiene que hacer —entender y resumir— está al alcance de un modelo pequeño.

El reparto, entonces, es doblemente virtuoso:

- El **conocimiento profundo** viaja desde el central hacia Hermes —pero el central no ve los datos de ninguna startup, porque no guarda estado y solo responde consultas de conocimiento—.
- La **comprensión de los datos privados** ocurre en local, en el modelo que el emprendedor controla —y como es una tarea ligera, ese modelo puede ser local de verdad—.

Los datos sensibles nunca *necesitan* salir de la máquina del emprendedor para que Hermes funcione. Que salgan o no es, a partir de ahí, una decisión suya.

## Una decisión que, como todas, es del fundador

Hermes no impone qué modelo usar. Ofrece una conexión estándar, y el emprendedor enchufa el que prefiera. Esa libertad es también una decisión de confidencialidad que se le devuelve:

- Quien priorice la **máxima confidencialidad** conecta un modelo local: nada de lo que Hermes procesa sale de su ordenador.
- Quien priorice la **máxima capacidad** conecta la API de un proveedor potente, sabiendo que los datos que Hermes envíe a ese modelo —el texto del fundador, la memoria— pasarán por ese tercero.

Ni una opción ni la otra se imponen. El sistema no resuelve esa tensión por el emprendedor: se la entrega, con la información para decidir. Es coherente con el principio que gobierna todo el proyecto —la autoridad, y con ella la responsabilidad, es del fundador—. La confidencialidad no es una característica que el sistema garantice unilateralmente; es una decisión que el sistema hace *posible* y deja en manos de quien tiene derecho a tomarla.

## En una frase

El conocimiento pesado se consulta al central, que no recuerda a nadie. La comprensión de lo privado se hace en local, con un modelo que el emprendedor elige y que, por ser ligera la tarea, puede quedarse enteramente en su máquina. Hermes piensa en casa lo que debe quedarse en casa, y pregunta fuera solo lo que no lleva datos dentro.
