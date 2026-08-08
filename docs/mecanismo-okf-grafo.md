# El mecanismo OKF-grafo: recuperación de conocimiento determinista

> Diseño técnico de cómo un especialista de startup-next recupera conocimiento
> cuando el dominio está lo bastante definido como para modelarlo como grafo,
> en vez de buscarlo por semejanza semántica (RAG). Ver [docs/vision.md](vision.md)
> para el marco conceptual (las "dos naturalezas de conocimiento").

## Objetivo

Un mecanismo genérico y reutilizable de recuperación de conocimiento por
grafo: navegación de nodos OKF (Open Knowledge Format) vía sus relaciones
declaradas, no similitud vectorial. El primer especialista construido sobre
este mecanismo fue `escalado`, pero el mecanismo en sí no es específico de
ese dominio — debe ser barato sumar después otros especialistas OKF
aportando solo sus ficheros de conocimiento y registrando el especialista,
sin rediseñar el mecanismo.

Convive con el RAG vectorial que usan los especialistas de dominio más
difuso (`ideacion`, `mvp`, `pmf`): son dos mecanismos de recuperación
paralelos dentro del mismo sistema, cada uno usado donde corresponde según
la naturaleza del conocimiento.

## El formato de origen: qué es un concepto OKF

Cada concepto de conocimiento reexpresado se guarda como un fichero Markdown
con cabecera YAML. Un ejemplo real (simplificado):

```
id: okf_contraposicionamiento
type: Concept
title: "Contraposicionamiento (Counter-positioning)"
version: "0.2"
status: Verified
sources:
  - resource: "urn:isbn:9780998116303"
    title: "7 Poderes: Los Fundamentos de la Estrategia Empresarial"
    authors:
      - "Hamilton W. Helmer"
    source_type: "Book"
tags:
  - estrategia
  - ventaja_competitiva
relations:
  prerequisites: []
  related_concepts:
    - okf_economias_de_escala
    - okf_costos_de_cambio
---

# Contraposicionamiento (Counter-positioning)

## 1. Resumen Ejecutivo
...
## 2. Definición y Principios Clave
...
## 3. Componentes y Estructura
...
## 4. Casos de Aplicación / Implicaciones Prácticas
...
```

Un hallazgo real que importa para entender el mecanismo: `relations.prerequisites`
en estos ficheros es un DAG *conceptual/compositivo* ("para entender X hace
falta entender Y primero"), no un orden metodológico temporal ("hacer X
antes que Y en la vida real de una startup"). Son dos tipos de precedencia
distintos y no deben confundirse — el mecanismo de grafo de un especialista
trabaja solo con el primero.

## Pertenencia concepto → especialista: un tag aditivo

Un mismo grafo de conocimiento (por ejemplo, los 7 poderes de Hamilton
Helmer) puede alimentar a más de un especialista sin duplicar ficheros: cada
concepto lleva un campo `especialistas: string[]` aditivo. Un concepto puede
servir a varios especialistas a la vez, y la pertenencia se declara
explícita en vez de inferirse de la carpeta o del nombre de archivo — la
carpeta de almacenamiento se organiza por *fuente* (un directorio por libro
o documento de origen), no por especialista, precisamente porque un
fichero puede servir a más de uno.

Un concepto sin ningún especialista asignado todavía puede convivir en el
mismo grafo, presente pero inerte, sin que ningún especialista lo consuma
— eso permite importar una fuente completa e íntegra aunque solo una parte
de ella tenga especialista implementado hoy.

## Carga: singleton en memoria

El grafo completo se carga una sola vez por proceso y se cachea en memoria
— mismo patrón que ya usa el motor de reglas metodológicas del sistema para
su propio grafo de prerrequisitos genéricos. No hay razón para releer o
reparsear los ficheros en cada consulta: son archivos locales, y el grafo
no cambia dentro de la vida de un proceso.

Al cargar, se valida la integridad del grafo completo: toda referencia
declarada en `relations` debe resolver a un `id` de concepto realmente
cargado. Si no, la carga falla con un mensaje claro — mejor un fallo
ruidoso al arrancar que una referencia rota descubierta en silencio durante
una consulta real.

## Algoritmo de recuperación: subgrafo inducido + BFS acotado desde un ancla

Cuatro pasos, en orden:

**1. Subgrafo inducido por especialista.** Antes de cualquier navegación,
se filtra el grafo completo a los nodos cuyo campo `especialistas` incluye
el rol del especialista actual. Esto resuelve de raíz el riesgo de fuga
entre especialistas: aunque un concepto de un dominio tenga una relación
declarada hacia un concepto de otro dominio en la fuente original, la
recuperación de un especialista nunca cruza a conceptos que no le
pertenecen, porque el filtro ocurre antes de navegar cualquier arista.

**2. Selección de ancla, por LLM, sobre el conjunto cerrado del
especialista.** El conjunto de conceptos posibles por especialista es
chico (del orden de una decena) — no hace falta búsqueda semántica para
elegir entre pocas opciones. Una llamada de modelo pequeña y acotada recibe
el título y el resumen ejecutivo de cada concepto del subgrafo más la
tarea que se está atendiendo, y devuelve uno o dos conceptos como ancla —
nunca inventados, siempre de los provistos.

**3. Recorrido en anchura (BFS) acotado desde la(s) ancla(s), dentro del
subgrafo ya filtrado.** Las dos relaciones del grafo (`prerequisites` y
`related_concepts`) se tratan como no dirigidas para este propósito — no
importa la dirección declarada, solo qué está conceptualmente cerca. El
recorrido está acotado por una profundidad máxima (por defecto, dos
saltos) y un tamaño máximo de nodos recuperados (por defecto, seis): si el
recorrido encuentra más nodos que el máximo, se descartan primero los más
lejanos, nunca el ancla.

**4. Serialización para el prompt.** Por cada concepto del subgrafo
resultante se inyecta su identificador, título, estado de validación, y
tres de las cuatro secciones de su cuerpo (resumen ejecutivo, definición y
principios, casos de aplicación) — se omite la sección de "componentes y
estructura" por defecto, porque en la práctica son diagramas de caracteres
pensados para lectura humana, de bajo valor accionable para un modelo y
con un coste de tokens no trivial.

Una nota honesta sobre este mecanismo: con subgrafos chicos (media docena
de nodos), en la práctica el recorrido acotado devuelve casi siempre el
subgrafo entero sin importar el ancla elegida, porque todo queda a
distancia corta de cualquier nodo. La poda por tamaño se diseñó igual, con
la expectativa de que sí se ejercite de verdad en grafos más grandes o más
dispersos — y así ocurrió en la práctica: los especialistas construidos
después, sobre fuentes con más nodos y una topología distinta (un único
nodo actuando de hub central en vez de un grafo casi completo), sí
exigieron la poda para mantener acotado el contexto que recibe el modelo.

## Formato de cita: linaje desde la fuente, no un identificador interno

Cada concepto ya trae en su cabecera el título y los autores de la fuente
de la que se reexpresó. La cita que llega al fundador se construye desde
ahí — título de la fuente, autor, y título del concepto como equivalente a
"capítulo" — nunca un identificador técnico interno:

```
7 Poderes: Los Fundamentos de la Estrategia Empresarial — Hamilton W. Helmer — Contraposicionamiento (Counter-positioning)
```

Si el estado de validación del concepto no es "verificado", la cita se
antepone con un marcador fijo:

```
[Conocimiento emergente, no validado] <linaje>
```

Esta es la aplicación directa de la cuarta cláusula del principio rector
del proyecto (ver [docs/vision.md](vision.md)): revelar el grado de
certeza de cada dato, no solo su contenido.

## Convivencia con el RAG vectorial

Los especialistas construidos sobre RAG vectorial no se tocan al construir
este mecanismo — son dos caminos de recuperación paralelos e
independientes, seleccionados según el especialista que atienda una tarea
dada, nunca mezclados dentro de un mismo ciclo. Un especialista de grafo
nunca dispara una búsqueda vectorial, y viceversa. La verificación de
fuentes (que ninguna cita esté inventada) opera igual para ambos: es una
comprobación determinista de pertenencia de conjunto contra lo realmente
recuperado, sin necesidad de una llamada de modelo adicional.

## Plan de verificación (aplicado al piloto y a cada especialista nuevo)

El estándar de verificación es el mismo para cualquier especialista nuevo
construido sobre este mecanismo:

- Al menos un caso real o realista contra producción, confirmando que el
  especialista correcto atiende la tarea y que las fuentes citadas en la
  recomendación final traen linaje real, no vacío.
- Regresión obligatoria de los especialistas ya existentes, porque el
  prompt del orquestador que decide a qué especialista enrutar una tarea es
  compartido por todos.
- Verificación offline de la integridad del grafo y del recorrido acotado
  contra los propios ficheros de conocimiento reales, no contra un grafo
  sintético de prueba.
- Limpieza de cualquier dato de prueba generado durante la verificación,
  sin tocar nunca datos reales de un fundador.

Sobre si conviene migrar los especialistas que hoy usan RAG vectorial hacia
este mecanismo: no es una decisión tomada de antemano. La comparación entre
ambos (tasa de fallo, calidad de las citas, coste de construcción) se mide
con cada especialista nuevo y queda como evidencia para decidir caso por
caso, no como una migración planificada.
