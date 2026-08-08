# La verificación del ciclo completo

> Cómo se demostró que la memoria de Hermes cambia el consejo —que el ciclo vive, y no es una promesa.

## Qué había que demostrar

Toda la arquitectura de Hermes-Colega existe para sostener una afirmación: que un colega con memoria da mejores consejos que un buscador sin ella, porque **recuerda lo que ya se probó**. Es fácil enunciarlo. Es fácil, incluso, construir todas las piezas que deberían hacerlo posible —la memoria, la captura de resultados, la composición del contexto histórico, la invocación a los especialistas—. Lo difícil, y lo que de verdad importa, es demostrar que esas piezas, encadenadas, producen el efecto prometido: que ante la misma pregunta, tras un resultado, **el consejo cambia**.

Hasta este punto, cada pieza del sistema se había verificado por separado. Los seis especialistas respondían bien de forma aislada. La memoria guardaba resultados. El contexto histórico se componía. Pero nada había ejercitado el *ciclo entero de principio a fin*: el recorrido que va del consejo, a la acción del fundador, al resultado que se cuenta, a la memoria que lo registra, a la siguiente invocación que ya lo tiene en cuenta. Ese ciclo es la tesis. Demostrarlo vivo era la prueba central del proyecto.

## El montaje de la prueba

La verificación se hizo contra el sistema real en producción, no contra un entorno simulado. Se partió de una situación limpia y controlada: una startup de prueba con una recomendación inicial ya registrada en memoria, para poder observar el ciclo desde un estado conocido.

El recorrido de la prueba tuvo tres momentos:

**Primero, una recomendación inicial.** El sistema, con el contexto de la startup, produjo su consejo de partida.

**Después, un resultado del fundador.** Se introdujo —tal como lo haría un fundador contándolo— que la vía recomendada se había intentado y no había funcionado. El mecanismo de cruce identificó a qué recomendación pendiente correspondía ese resultado, y lo registró en la memoria con su estado: una hipótesis falseada.

**Por último, la misma pregunta, otra vez.** Con la memoria ya actualizada, se volvió a invocar al sistema con la tarea original. La pregunta era idéntica; lo único distinto era que la memoria ahora contenía el resultado del intento fallido.

La verificación estrella era comparar la primera recomendación con la segunda.

## El resultado

La diferencia fue clara y coherente.

**La primera recomendación** giraba, en sus distintas variantes, alrededor de una misma idea: diagnosticar primero en qué fase de poder se encontraba la startup antes de actuar. Era un consejo razonable de partida —establecer el diagnóstico antes de elegir la palanca.

El fundador reportó que ese diagnóstico se había intentado y no había servido.

**La segunda recomendación**, ante la misma tarea exacta, **abandonó por completo el enfoque del diagnóstico de fase.** Ninguna de sus variantes lo mencionaba. En su lugar, saltó directamente a las palancas concretas que la situación permitía —costes de cambio, el recurso que la startup ya controlaba, reforzar la posición existente—, que eran precisamente las alternativas que quedaban una vez descartada la primera vía.

El especialista real de producción —no una simulación— había recordado que el primer enfoque no funcionó, y redirigió. Un buscador de documentos, ante la misma pregunta las dos veces, habría devuelto lo mismo, porque no tiene memoria de lo ocurrido entre una consulta y otra. El sistema devolvió algo distinto *por causa del resultado capturado*. El mantra que gobierna la visión del proyecto —la acción genera experiencia, la experiencia funda la siguiente acción— dejó de ser una frase y pasó a ser comportamiento observable.

## Qué es real y qué se simuló

Una verificación solo vale lo que vale su honestidad sobre sus propios límites. Esta tiene una parte real y una parte simulada, y la distinción importa.

**Lo real —y es donde se juega la tesis—** es la invocación al sistema de especialistas en producción. La recomendación que cambió la produjo el especialista real, con su mecanismo real de recuperación de conocimiento, contra el servicio desplegado. Ahí, donde se demuestra que la memoria modula el consejo, no hay simulación.

**Lo simulado** es el canal conversacional: en un uso real, el fundador contaría su resultado en una conversación, y una interfaz recogería ese texto. Como esa interfaz es trabajo posterior, la prueba le pasó el texto del fundador directamente a la función que lo procesa —que es exactamente como esa función está diseñada para recibirlo—. No se falseó el resultado; se sustituyó la interfaz de entrada por una llamada directa. La lógica que cruza el resultado contra la recomendación y lo registra en memoria es la real.

Dicho de otro modo: lo que se simuló fue *cómo llega* el resultado del fundador, no *qué hace el sistema con él*. Y la pregunta que la verificación venía a responder —¿cambia el consejo cuando la memoria registra un resultado?— se responde íntegramente en la parte real.

## Lo que esto cierra, y lo que abre

Con esta verificación, la columna vertebral del proyecto queda demostrada: el ciclo completo funciona de punta a punta, y la memoria hace lo que la tesis prometía. No es un conjunto de piezas que *deberían* encajar; es un ciclo que se observó cerrar.

Queda abierto, como trabajo natural siguiente, hacer real la última pieza simulada —el canal conversacional por el que el fundador cuenta sus resultados— para que el ciclo funcione sin ninguna sustitución. Pero eso es refinar la entrada de un ciclo que ya está probado, no demostrar un ciclo por probar. La diferencia entre "las piezas existen" y "el ciclo vive" ya está saldada.
