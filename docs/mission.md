# Misión del MVP

Diseñar, implementar y evaluar un minijuego 2D de escondite en el que un NPC aprenda a seleccionar estrategias de ocultación mediante aprendizaje por refuerzo, comparando su rendimiento con una política aleatoria mediante experimentos reproducibles.

## Alcance aprobado: fase 1 del TFM

- El NPC aprende a seleccionar un escondite entre varias opciones.
- El jugador inspecciona escondites lanzando chontaduros. Si inspecciona el escondite ocupado, descubre al NPC y gana.
- La inspección es determinista. No se implementan probabilidades de descubrimiento por proximidad o visibilidad.
- El lanzamiento de chontaduros es una interacción conceptual, sin necesidad de físicas de proyectiles.
- No hay navegación inteligente del NPC ni cambio dinámico de escondite durante la partida.
- Se compara la política aprendida con una política aleatoria mediante experimentos reproducibles.
- La arquitectura debe ser ampliable y mantener separadas simulación, entrenamiento y evaluación.

## Límites y decisiones pendientes

La navegación hasta el escondite pertenece a la fase 2. La decisión de permanecer oculto o cambiar de escondite pertenece a la fase 3. Ninguna de estas capacidades forma parte del MVP.

Las reglas aún no definidas, los contratos del entorno, las recompensas, la tecnología y el protocolo experimental deberán concretarse en especificaciones aprobadas antes de implementar el comportamiento afectado. No inferir requisitos a partir de estas omisiones.
