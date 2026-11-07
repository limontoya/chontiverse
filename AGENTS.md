# Instrucciones del proyecto

## Propósito y alcance

Chontiverse es un proyecto de videojuego con identidad cultural caleña y un TFM centrado en aprendizaje por refuerzo.

Antes de proponer funcionalidades o modificar comportamiento, consultar `docs/vision.md`, `docs/mission.md`, las especificaciones aplicables en `specs/` y las decisiones existentes en `docs/architecture/`.

El MVP corresponde exclusivamente a la fase 1:
- Un minijuego 2D en el que un NPC aprende a seleccionar un escondite entre varias opciones.
- El jugador inspecciona escondites mediante lanzamientos conceptuales de chontaduros. Si inspecciona el escondite ocupado, descubre al NPC y gana.
- La inspección es determinista: no introducir probabilidades de descubrimiento por proximidad o visibilidad.
- No se requieren físicas de proyectiles. No añadirlas sin aprobación de alcance.
- No hay navegación inteligente del NPC ni cambio dinámico de escondite.
- Se compara la política aprendida con una política aleatoria mediante experimentos reproducibles.
- La arquitectura es ampliable, sin implementar anticipadamente capacidades de fases posteriores.

La navegación del NPC pertenece a la fase 2. Aprender cuándo permanecer oculto o cambiar de escondite pertenece a la fase 3. La hoja de ruta no autoriza su implementación.

Toda ampliación del MVP requiere confirmación explícita del usuario y una justificación de su necesidad académica.

## Fuentes de verdad: spec-as-source

Las especificaciones aprobadas son la fuente principal del comportamiento esperado. La implementación y las pruebas deben ajustarse a ellas y respetar la visión, la misión y las decisiones de arquitectura aprobadas.

Distinguir documentos propuestos de documentos aprobados. No tratar una propuesta como requisito autorizado.

No interpretar documentos vacíos, omisiones o decisiones pendientes como autorización para inventar requisitos. Ante contradicciones, señalar las fuentes afectadas y solicitar la decisión necesaria antes de implementar el comportamiento en conflicto.

No alterar una especificación para justificar a posteriori una desviación del código. Proponer primero el cambio de comportamiento y obtener su aprobación antes de implementarlo.

## Flujo Spec-Driven Development

1. Identificar la historia, requisito o pregunta experimental que motiva el trabajo.
2. Revisar visión, misión, especificaciones y decisiones de arquitectura aplicables.
3. Proponer o actualizar la especificación antes de implementar cambios de comportamiento; obtener aprobación de las decisiones pendientes.
4. Definir criterios de aceptación observables y su método de verificación.
5. Implementar únicamente el alcance especificado y autorizado.
6. Ejecutar las verificaciones pertinentes y registrar sus resultados.
7. Actualizar la trazabilidad y comunicar cobertura, limitaciones y decisiones pendientes.

Las correcciones que restauren un comportamiento ya aprobado no requieren redefinir el requisito. No ampliar el alcance durante una corrección.

## Especificaciones y trazabilidad

Usar identificadores estables y únicos dentro de cada categoría:

| Prefijo | Significado |
| --- | --- |
| US | Historia de usuario |
| RF | Requisito funcional |
| RNF | Requisito no funcional |
| RE | Requisito experimental |
| SPEC | Especificación |
| ADR | Decisión de arquitectura |
| EXP | Experimento |

Usar el formato `US-001`, `RF-001`, `RNF-001`, `RE-001`, `SPEC-001`, `ADR-001` y `EXP-001`. No renumerar identificadores existentes ni reutilizar los retirados.

Cada especificación debe indicar:
- Identificador, estado y objetivo.
- Historias y requisitos relacionados, según corresponda.
- Alcance, exclusiones y decisiones pendientes.
- Comportamiento esperado y contratos relevantes.
- Criterios de aceptación y método de verificación.
- Referencias a decisiones de arquitectura, implementación, pruebas y experimentos que la materializan o verifican, cuando existan.

Mantener trazabilidad entre historias, requisitos, especificaciones, implementación y pruebas. Vincular los requisitos experimentales con sus experimentos y resultados. Las pruebas deben identificar el requisito o criterio que verifican.

No duplicar requisitos entre documentos: enlazar su fuente. No crear historias artificiales para decisiones puramente técnicas o experimentales; registrar las relaciones que correspondan.

## Arquitectura y dependencias

Respetar las decisiones documentadas en `docs/architecture/`. Registrar mediante ADR las decisiones técnicas relevantes, sus motivos y consecuencias.

Separar responsabilidades mediante contratos explícitos:
- La simulación define y ejecuta las reglas del entorno.
- Las políticas seleccionan acciones conforme al mismo contrato del entorno.
- El entrenamiento actualiza la política aprendida.
- La evaluación mide políticas sin actualizarlas y utiliza un protocolo común de comparación.

Mantener comparables la política aprendida y la aleatoria bajo las mismas reglas y condiciones de evaluación. Evitar que el entorno dependa de un algoritmo concreto de entrenamiento.

Favorecer la solución más sencilla que satisfaga las especificaciones. No añadir abstracciones, servicios o dependencias innecesarios. Justificar cualquier dependencia nueva, sus alternativas y su coste de mantenimiento antes de incorporarla.

## Reproducibilidad y verificación

Especificar el protocolo experimental antes de ejecutar la evaluación destinada a sustentar conclusiones: métricas, condiciones, semillas, número de ejecuciones y episodios, presupuesto y criterio de comparación. No inventar sus valores si están pendientes.

Registrar en cada EXP los requisitos y especificaciones relacionados, la configuración, semillas, versiones de código y dependencias, procedimiento de ejecución y resultados necesarios para reproducirlo.

Separar entrenamiento y evaluación. No actualizar la política durante la evaluación. Documentar la política aleatoria, los escenarios y la separación entre condiciones de entrenamiento y evaluación.

Reportar la variabilidad entre ejecuciones según el protocolo aprobado. No presentar una ejecución aislada como evidencia suficiente de mejora ni confundir recompensa de entrenamiento con métrica de evaluación.

Verificar las reglas deterministas de inspección y los contratos aprobados de observaciones, acciones, recompensas y finalización. Ejecutar pruebas proporcionadas al cambio y vincularlas con sus criterios de aceptación.

## Disciplina de trabajo

Respetar las restricciones de cada solicitud. No modificar archivos cuando se solicite únicamente inspección ni editar archivos fuera del alcance autorizado.

No implementar código ni añadir dependencias en tareas exclusivamente documentales. No sobrescribir cambios ajenos.

No realizar commits ni push sin autorización explícita.

Al entregar trabajo, indicar qué cambió, qué requisitos se cubrieron, qué verificaciones se ejecutaron y qué queda pendiente. No afirmar que una prueba se ha ejecutado si no se ha realizado.
