# Hermes-Colega

Un colega de IA que aprende de tu emprendimiento, recuerda por dónde vas y
adapta su consejo a tu startup — con conocimiento validado, y con lo que ya
sabe de ti.

> No hay nada más práctico que una buena teoría —pero la teoría hay que
> validarla en tu entorno: tu emprendimiento.

Emprender en la era de la IA es un juego distinto, y no por usar IA como
herramienta ocasional, sino por incorporar un colega que aprende de tu
emprendimiento y crece con él. Hermes-Colega junta dos tipos de
conocimiento — el validado sobre cómo se construye una startup, y el de tu
startup en concreto — para que cada recomendación esté fundada en ambos.

Cada instancia de Hermes es privada: vive en el entorno del propio
emprendedor, con su memoria físicamente separada de la de cualquier otra
startup. El conocimiento validado, en cambio, vive en un servicio central
compartido que no guarda estado de nadie. Esa separación no es un detalle
de implementación — es la garantía de confidencialidad del sistema.

## Empezar

- **[hercolega.com](https://hercolega.com)** — la web del proyecto, con el
  recorrido completo paso a paso.
- **[startup-advisor](https://startup-advisor-sand.vercel.app)** — conoce
  tu situación: responde unas preguntas y obtén un informe de en qué punto
  está tu emprendimiento.
- **[startup-next](https://startup-next-ui.vercel.app)** — descubre tu
  próxima acción: dale tu situación y te dirá cuál es la actividad más
  relevante para avanzar ahora.

## Leer el detalle

El razonamiento completo del proyecto, con evidencia y decisiones
documentadas, vive en [`/docs`](docs):

- [**Visión**](docs/vision.md) — el principio rector, el mantra, y la
  frontera entre Hermes y startup-next.
- [**El mecanismo OKF-grafo**](docs/mecanismo-okf-grafo.md) — cómo se
  recupera conocimiento cuando el dominio está lo bastante definido como
  para modelarlo como grafo.
- [**La expansión a seis especialistas**](docs/expansion-especialistas.md)
  — cómo startup-next cubre hoy el ciclo de vida completo de una startup.
- [**El estado, jamás en el centro**](docs/ontology-engine.md) — por qué
  el servicio central no guarda nada de ninguna startup, con su
  [detalle técnico](docs/ontology-engine-diseno.md).
- [**La verificación del ciclo completo**](docs/ciclo-completo.md) — cómo
  se demostró que la memoria de Hermes cambia el consejo.
- [**Observabilidad**](docs/observabilidad.md) — cómo se observa un
  sistema agéntico distribuido sin traicionar la confidencialidad que lo
  define.

## Los repositorios de código

El proyecto está repartido en repositorios independientes, cada uno con su
propio ciclo de despliegue. Por ahora son privados (la documentación de
`/docs` es lo público de esta primera etapa); los enlaces se activan aquí
cuando se abran:

| Repositorio | Rol |
|---|---|
| `startup-advisor` | Diagnóstico inicial: entrevista a un fundador y genera su informe de situación |
| `startup-next` | Servicio central de conocimiento: orquestador, especialistas de dominio, sin estado |
| `startup-next-ui` | Frontend de startup-next |
| `hermes-startup-next` | Hermes: memoria privada por startup, contexto histórico, cliente de startup-next |

Este repositorio (`hercolega`) es aditivo: es el punto de entrada y el
hogar de la documentación pública del proyecto. No absorbe ni reemplaza a
los repositorios de código — cada uno conserva su propio ciclo de vida.

## Estructura de este repositorio

```
/
├── web/    → el código de hercolega.com (estático, sin build)
└── docs/   → documentación pública del proyecto
```
