# SPEC-001 — Entorno de escondite

- **Estado:** BORRADOR; no aprobado como especificación completa.
- **Versión:** v0.6.
- **Fecha:** 2026-11-08.
- **Ámbito:** fase 1 del TFM MVP.
- **Fuentes:** [visión](../docs/vision.md), [misión](../docs/mission.md), instrucciones de [AGENTS.md](../AGENTS.md) y [SPEC-002 v0.1 — Contexto y buscador automatizado](SPEC-002-context-and-searcher.md).
- **Origen de las decisiones:** decisiones aprobadas por el usuario para v0.3 y revisión funcional de v0.4 y decisiones A1, A2 y A3 aprobadas para v0.5; sincronización documental v0.6 autorizada con las decisiones expresas recogidas en SPEC-002 v0.1. Ambos documentos conservan su estado BORRADOR.

## Objetivo y alcance

Definir el episodio de un minijuego 2D en el que un NPC selecciona un escondite y un buscador dispone de dos inspecciones para descubrirlo. El entorno permitirá estudiar la selección de escondites mediante aprendizaje por refuerzo y compararla con una política aleatoria, manteniendo separadas simulación, entrenamiento y evaluación.

Esta especificación documenta decisiones aprobadas junto con propuestas y contratos incompletos. El estado BORRADOR no revoca las decisiones ya aprobadas ni convierte las propuestas en requisitos autorizados. No implementar los aspectos pendientes por inferencia.

## Decisiones aprobadas

- Hay cinco escondites identificables y las acciones de selección del NPC son `E1`, `E2`, `E3`, `E4` y `E5`.
- El NPC elige un único escondite al inicio y no se desplaza ni cambia de escondite durante el episodio.
- El buscador dispone de dos chontaduros por partida. Cada uno inspecciona un escondite y no se puede inspeccionar dos veces el mismo.
- La inspección es determinista: inspeccionar el escondite ocupado descubre al NPC y da la victoria al buscador.
- Agotar las dos inspecciones sin descubrir al NPC da la victoria al NPC.
- La generación del contexto y la pista compartida están definidas en SPEC-002, RF-012 y RF-013. Las zonas de atención `NORTH`, `CENTER` y `SOUTH` están aprobadas; su codificación definitiva como observación del agente sigue pendiente para SPEC-003. La zona de atención no es la posición real del NPC.
- Se separan el modo de juego humano y el modo automatizado de entrenamiento/evaluación.
- El buscador automatizado utiliza la matriz de pesos y la política probabilística fija definidas en SPEC-002, RF-014 a RF-018. La libertad del humano, la separación visual, la reproducibilidad y los registros se especifican allí en RF-019 a RF-022; la política del buscador no aprende.
- Unity y C# están aprobados como motor y lenguaje para la presentación del videojuego 2D. La lógica del entorno permanece separada de la representación visual. Tecnología de entrenamiento, algoritmo de RL e integración con Unity siguen pendientes.
- Las operaciones inválidas se rechazan con un resultado explícito, distinto de una inspección válida fallida, sin consumir chontaduros, generar recompensas ni modificar el estado del episodio. No se fija una API ni tipos concretos.
- Cada inspección válida consume exactamente un chontaduro y añade su objetivo al conjunto lógico de inspeccionados, tanto si acierta como si falla.
- Las manchas aparecen exclusivamente tras inspecciones válidas fallidas y permanecen durante la partida y en el resultado final hasta iniciar una nueva partida. Un acierto muestra su resultado, sin mancha de fallo sobre el escondite ocupado.

- A1: inspeccionar antes de una selección válida del NPC se rechaza explícitamente, sin consumir chontaduros, registrar inspecciones ni modificar el episodio.
- A2: una primera selección fuera de `E1` a `E5` se rechaza sin recompensas ni cambios de estado; el episodio sigue pendiente de selección y admite una elección válida posterior.
- A3: iniciar otra partida antes de finalizar la actual se rechaza sin efectos secundarios. Tras el final, el reinicio restablece los dos chontaduros, inspeccionados, manchas, resultado y estado terminal; permite un nuevo contexto y una nueva selección según RF-010.

La variación del riesgo se refiere a qué escondites son inspeccionados, no a la probabilidad de descubrir al NPC una vez inspeccionado el escondite ocupado. La política probabilística y su matriz se definen exclusivamente en SPEC-002; no se duplican aquí sus fórmulas ni pesos. No imponen una conducta al jugador humano.

## Propuestas pendientes de aprobación

| Elemento | Propuesta | Estado |
| --- | --- | --- |
| Observación inicial | Campo `attention_zone` para representar la zona de atención ya aprobada en SPEC-002 | Nombre, codificación y contrato definitivo de observación pendientes para SPEC-003; las tres zonas no están pendientes |
| Recompensa del NPC | `+1` si no es descubierto; `-1` si es descubierto | Pendiente de aprobación; momento de entrega y posibles recompensas intermedias sin definir |

La zona de atención, su generación y la disponibilidad de la pista están definidas en SPEC-002, RF-012 y RF-013. Su representación mediante el campo `attention_zone` sigue siendo una propuesta para SPEC-003, que es una referencia futura y no una especificación aprobada por este documento. La matriz de SPEC-002 define pesos de inspección, no posiciones físicas ni una correspondencia espacial entre zonas y escondites.

## Requisitos funcionales

Los RF siguientes recogen el núcleo aprobado. Los detalles pendientes se indican expresamente y bloquean únicamente la concreción del comportamiento afectado.

| ID | Requisito | Aspectos pendientes |
| --- | --- | --- |
| RF-001 | El entorno ofrece exactamente cinco escondites identificables mediante `E1` a `E5`. | Disposición y presentación visual. |
| RF-002 | Al inicio del episodio el NPC selecciona exactamente una acción entre `E1` y `E5`; el escondite elegido permanece invariable hasta finalizar. Una primera selección inexistente se rechaza según RF-009; el episodio permanece pendiente de selección y admite una selección válida posterior. Una segunda selección tras una elección válida se rechaza según RF-009. | Contrato de invocación. |
| RF-003 | Cada episodio concede al buscador dos chontaduros. Cada inspección válida consume exactamente uno, acierte o falle, y añade el objetivo al conjunto lógico de escondites inspeccionados. | Presentación de la interacción, salvo lo definido en RF-011. |
| RF-004 | Un escondite ya inspeccionado no puede volver a inspeccionarse en el mismo episodio. Los intentos repetidos se rechazan según RF-009, independientemente de las marcas visuales. | Ninguno para la prohibición de reinspección. |
| RF-005 | Si una inspección corresponde al escondite ocupado, el NPC es descubierto, el buscador gana y el episodio finaliza en esa inspección. | Ninguno para la regla de descubrimiento. |
| RF-006 | Si las dos inspecciones se completan sin descubrir al NPC, el NPC gana y el episodio finaliza. | Ninguno para esta condición de victoria. |
| RF-007 | Antes de seleccionar escondite, el NPC recibe una pista contextual; el jugador humano recibe la misma pista. La generación y disponibilidad del contexto, la matriz de pesos y su relación con el riesgo automatizado están definidas en SPEC-002, RF-012 a RF-018. El humano conserva su libertad según SPEC-002, RF-019. | Codificación definitiva como observación del agente para SPEC-003 y presentación visual concreta. No se exige que el humano siga una conducta concreta. |
| RF-008 | El entorno distingue el modo humano del automatizado. En el primero inspecciona el jugador; en entrenamiento/evaluación inspecciona un buscador de política fija. Ambos respetan RF-001 a RF-007, RF-009 y RF-010; RF-011 define la presentación humana. La pista, matriz y política fija, la libertad humana, la separación de responsabilidades y la reproducibilidad están definidas en SPEC-002, RF-012 a RF-022. | Contratos de integración y protocolo experimental; la política fija ya está definida. |
| RF-009 | Rechazar inspecciones antes de una selección válida del NPC, una primera selección fuera de `E1` a `E5`, el inicio de otra partida mientras la actual no haya finalizado, inspecciones repetidas o de escondites inexistentes, una segunda selección tras una elección válida del NPC y cualquier operación de juego posterior al final salvo iniciar una nueva partida. El resultado explícito de rechazo se distingue de un fallo válido. Las acciones inválidas no consumen chontaduros, no generan recompensas ni modifican el estado del episodio. | Sin API ni tipos concretos; otros casos inválidos no definidos siguen pendientes. |
| RF-010 | Tras finalizar la partida anterior, iniciar una nueva restablece dos chontaduros, vacía los inspeccionados, elimina el resultado anterior y desactiva el estado terminal. Elimina las marcas visuales, descarta la selección anterior del NPC y permite una nueva. Genera el nuevo contexto y entrega la pista según SPEC-002, RF-012 y RF-013, con reproducibilidad y registro conforme a sus RF-021 y RF-022. | Detalles técnicos de integración y registro; la generación del contexto ya está definida. |
| RF-011 | En modo humano, mostrar una mancha exclusivamente por cada inspección válida de un escondite vacío; conservarla durante la partida y en la pantalla de resultado hasta iniciar una nueva. Mostrar el acierto sin mancha de fallo sobre el escondite ocupado. Unity y C# presentan estos resultados sin determinar el estado lógico ni condicionar la simulación a animaciones o físicas. | Diseño visual concreto e integración. |

## Precondiciones

El inicio inicial crea el primer episodio. Para iniciar otro, el anterior debe haber finalizado; la espera de selección también forma parte de la partida en curso. Los rechazos se rigen por RF-009 y no alteran el estado ni generan recompensas.

- Para comenzar un episodio deben estar disponibles los cinco escondites, el modo de ejecución y la generación y entrega de una pista contextual compartida conforme a SPEC-002, RF-012 y RF-013.
- Antes de la elección del NPC debe haberse proporcionado la pista al NPC. En modo humano, debe proporcionarse la misma pista al jugador antes de sus inspecciones.
- Antes de la primera inspección debe existir exactamente un escondite seleccionado por el NPC.
- Para cada inspección válida, el episodio sigue activo, queda al menos un chontaduro y el objetivo es un escondite existente que no ha sido inspeccionado.
- El modo automatizado utiliza la configuración de referencia y la política fija de SPEC-002, RF-014 a RF-018, con la separación y reproducibilidad de sus RF-020 a RF-022. La definición de la política ya no es un pendiente; los contratos de integración y el protocolo experimental siguen pendientes antes de las ejecuciones que los requieran.

RF-009 define el rechazo de los casos inválidos enumerados. Los demás casos, abandonos y errores de ejecución permanecen pendientes; no inferir su tratamiento.

## Flujo del episodio

1. Crear la primera partida o, solo tras finalizar la anterior, iniciar otra según RF-010: cinco escondites, dos chontaduros, ningún inspeccionado, sin resultado anterior, estado no terminal y sin selección previa del NPC. Eliminar todas las marcas visuales anteriores.
2. Generar el contexto y suministrar la pista según RF-007 y SPEC-002, RF-012 y RF-013; mantenerlo constante durante el episodio. La codificación definitiva de la observación no se fija aquí.
3. El NPC selecciona una vez `E1`, `E2`, `E3`, `E4` o `E5`. Una selección inexistente se rechaza sin recompensas ni cambios de estado; el episodio permanece pendiente de selección y permite una selección válida posterior. La selección válida permanece fija.
4. El buscador humano elige libremente un objetivo válido; el automatizado lo selecciona mediante la matriz y política fija de SPEC-002, RF-014 a RF-019. Solicitar la inspección al entorno. Validar la operación antes de modificar el episodio. Rechazar los casos de RF-009 con un resultado explícito y sin efectos secundarios; el rechazo no es una inspección fallida válida.
5. Para una inspección válida, consumir exactamente un chontaduro y añadir el objetivo al conjunto lógico de inspeccionados antes de resolver el resultado.
6. Comparar el objetivo con el escondite del NPC. Si coinciden, registrar descubrimiento, victoria del buscador y estado terminal inmediatamente. En modo humano, mostrar el acierto sin añadir una mancha de fallo sobre el escondite ocupado.
7. Si no coinciden, en modo humano añadir la mancha de fallo. Si queda un chontaduro, continuar desde el paso 4; si no queda, registrar victoria del NPC y estado terminal.
8. Conservar las manchas de los fallos también en la pantalla de resultado final. Rechazar toda operación de juego posterior al final, salvo iniciar una nueva partida desde el paso 1.

Tras una selección válida, una segunda selección del NPC se rechaza en cualquier punto del mismo episodio. Antes de esa selección válida, toda inspección se rechaza sin registrar inspecciones. Iniciar otra partida mientras la actual no haya finalizado también se rechaza, incluida la espera de selección del NPC. Las acciones inválidas no generan recompensas; la recompensa de los resultados válidos sigue pendiente y la propuesta no forma parte del contrato aprobado. Ninguna transición lógica depende de animaciones ni de físicas de proyectiles.

## Condiciones de finalización e invariantes

| Condición | Resultado |
| --- | --- |
| El NPC es descubierto en la primera o segunda inspección | Victoria del buscador; no se realizan más inspecciones en ese episodio |
| Se completan dos inspecciones distintas sin descubrir al NPC | Victoria del NPC |

Desde la selección válida del NPC hasta finalizar el episodio hay un único escondite ocupado, invariable; antes de esa selección, el nuevo episodio no tiene un escondite elegido. Se realizan como máximo dos inspecciones válidas sobre escondites distintos. No hay sorteo de descubrimiento por proximidad o visibilidad. No se define empate ni finalización por tiempo; abandonos, errores y truncamientos técnicos siguen pendientes.

El presupuesto restante es dos menos el número de inspecciones válidas. Un rechazo no cambia presupuesto, inspeccionados, selección del NPC, contexto, resultado ni estado terminal, y no genera recompensas. Después del final solo iniciar una nueva partida permite restablecer el episodio según RF-010.

## Criterios de aceptación y pruebas previstas

Todos los casos son **pruebas previstas, no implementadas ni ejecutadas**. Los identificadores CA y PP son referencias locales de SPEC-001.

| Criterio | RF | Resultado observable | Prueba prevista |
| --- | --- | --- | --- |
| CA-01 | RF-001 | Existen exactamente cinco identificadores distintos: `E1` a `E5`. | PP-01: comprobar el conjunto de escondites. |
| CA-02 | RF-002 | Cada una de las cinco acciones válidas selecciona un único escondite que no cambia durante el episodio. | PP-02: recorrer las cinco selecciones y comprobar su invariancia tras las inspecciones. |
| CA-03 | RF-003 | Una partida empieza con dos chontaduros; cada inspección válida consume exactamente uno y añade el objetivo a los inspeccionados, tanto en aciertos como en fallos. | PP-03: comprobar presupuesto y conjunto lógico al inicio, tras un acierto inicial (queda uno), tras un fallo (queda uno) y tras fallo seguido de acierto o de otro fallo (quedan cero). |
| CA-04 | RF-004, RF-009 | Ningún episodio contiene dos inspecciones del mismo escondite; repetirlo produce rechazo explícito sin efectos secundarios. | PP-04: tras un fallo válido, repetir el objetivo y verificar rechazo distinto del fallo válido, sin consumo, recompensa ni modificación del estado. Comprobar el bloqueo también sin representación gráfica. |
| CA-05 | RF-005 | Coincidir con el escondite ocupado descubre siempre al NPC y termina la partida con victoria del buscador. | PP-05: cubrir cada escondite como acierto en la primera y en la segunda inspección; comprobar que no continúa el episodio. |
| CA-06 | RF-006 | Dos inspecciones distintas que fallan terminan con victoria del NPC. | PP-06: cubrir las elecciones del NPC y los pares ordenados de inspecciones válidas que no lo contienen. |
| CA-07 | RF-007 | La generación del contexto, la pista compartida y la variación del riesgo mediante la matriz y política fija cumplen SPEC-002, RF-012 a RF-018; el humano conserva la libertad de su RF-019. | PP-07: comprobar igualdad y disponibilidad de la pista antes de las decisiones e integrar las verificaciones de contexto, matriz y política de SPEC-002, CA-01 a CA-08 / PP-01 a PP-08. La codificación definitiva de observación sigue pendiente para SPEC-003. |
| CA-08 | RF-008, RF-009, RF-010 | Los modos humano y automatizado comparten transiciones lógicas, presupuesto, inspeccionados, resultado, rechazo de inválidas y reinicio. El automatizado aplica la matriz y política fija de SPEC-002, RF-014 a RF-018, sin requerir gráficos; contexto, pista, libertad humana, separación y reproducibilidad se rigen por sus RF-012 a RF-022. | PP-08: usar secuencias equivalentes para comparar esos estados en ambos modos, sin esperar animaciones ni simular proyectiles. Verificar la integración con SPEC-002, CA-01 a CA-11 / PP-01 a PP-11; su política ya está definida. Los contratos técnicos de integración siguen pendientes. |
| CA-09 | RF-004, RF-008, RF-011 | En modo humano, la mancha aparece exclusivamente tras una inspección válida fallida. El acierto muestra su resultado sin mancha de fallo sobre el ocupado. Las manchas permanecen durante la partida y en la pantalla final y desaparecen únicamente al iniciar otra partida. La prohibición de reinspección depende del estado lógico, separado de Unity. | PP-09: cubrir (a) fallo seguido de acierto: una mancha en el vacío y ninguna en el ocupado; (b) dos fallos consecutivos: dos manchas. Comprobar persistencia durante el juego y en ambos resultados finales, rechazo de reinspección sin nuevas manchas y limpieza de todas las manchas solo al iniciar otra partida. Verificar el bloqueo sin representación gráfica y sin depender de animaciones ni físicas. |
| CA-10 | RF-002, RF-009 | Los casos inválidos enumerados se rechazan explícitamente sin efectos secundarios ni recompensas; un rechazo no se confunde con una inspección válida fallida. | PP-10: probar objetivo repetido, objetivo inexistente y segunda selección del NPC, antes y después de una inspección cuando corresponda. Comparar presupuesto, inspeccionados, selección, contexto, resultado y estado terminal antes y después; comprobar ausencia de recompensa y de cambios visuales. |
| CA-11 | RF-005, RF-006, RF-009 | Tras cualquiera de las victorias se rechazan las operaciones de juego salvo iniciar una nueva partida. | PP-11: tras acierto inicial, fallo seguido de acierto y dos fallos, intentar inspeccionar y seleccionar escondite; verificar rechazo explícito y ausencia de cambios de estado, consumo, recompensas o cambios visuales. |
| CA-12 | RF-002, RF-010, RF-011 | Tras finalizar la anterior, iniciar otra partida restablece completamente el episodio, elimina marcas y permite elegir nuevamente escondite; el contexto y la pista se rigen por SPEC-002, RF-012 y RF-013, con reproducibilidad y registro según sus RF-021 y RF-022. | PP-12: desde cada resultado final, iniciar otra partida y comprobar presupuesto de dos, conjunto vacío, ausencia de resultado y selección previos, estado no terminal y ninguna marca. Seleccionar un escondite y comprobar que vuelve a ser posible inspeccionar los objetivos de la partida anterior. Verificar la nueva generación y entrega de la pista según SPEC-002, CA-01, CA-02, CA-10 y CA-11 / PP-01, PP-02, PP-10 y PP-11; no exigir que su valor difiera del anterior. |
| CA-13 | RF-002, RF-003, RF-009 | Inspeccionar antes de una selección válida produce rechazo explícito distinto de un fallo válido, sin consumo, registro de inspecciones ni cambios de estado. | PP-13: intentar inspeccionar antes de seleccionar y después de una selección inválida. Comparar presupuesto, inspeccionados, selección, contexto, resultado, estado terminal y marcas antes y después; comprobar rechazo y ausencia de recompensas. Elegir válidamente y comprobar que se permite inspeccionar. |
| CA-14 | RF-001, RF-002, RF-009 | Una primera selección inexistente se rechaza explícitamente sin efectos secundarios ni recompensas; permanece pendiente de selección y permite una elección válida posterior. | PP-14: intentar y repetir una selección inexistente; comparar todo el estado enumerado en PP-13 y comprobar rechazo sin recompensas. Después probar cada selección válida en episodios independientes y comprobar que permite inspeccionar. |
| CA-15 | RF-009, RF-010, RF-011 | Iniciar otra partida mientras la actual no haya finalizado se rechaza explícitamente sin efectos secundarios ni recompensas. | PP-15: intentar reiniciar durante la espera de selección, tras la selección válida y tras un primer fallo; comparar todo el estado de PP-13, incluida la mancha, y comprobar rechazo sin recompensas. Finalizar y verificar el reinicio válido mediante PP-12. |

La observación y las recompensas propuestas necesitarán criterios y pruebas adicionales si se aprueban. No se fijan aquí métricas de rendimiento ni umbrales de mejora.

## Separación y trazabilidad experimental

- La simulación mantiene las reglas y resultados del episodio independientemente de la política del NPC y del origen humano o automatizado de las inspecciones.
- Para CA-09, la simulación conserva el conjunto lógico de todos los escondites inspeccionados, incluido el acierto. Unity y C# representan con manchas únicamente los fallos válidos, no todos los miembros de ese conjunto. La presencia o ausencia de gráficos no determina si un escondite puede inspeccionarse. Simulación y pruebas automatizadas no dependen de manchas, animaciones ni físicas de proyectiles.
- El entrenamiento actualiza la política del NPC; la evaluación la ejecuta sin actualizarla. El buscador fijo no debe confundirse con la política aleatoria de referencia del NPC.
- La comparación entre NPC aprendido y NPC aleatorio debe utilizar las mismas reglas y condiciones del protocolo aprobado. La distribución de la selección aleatoria aún no está definida.
- Los futuros EXP deberán enlazar SPEC-001, SPEC-002 y los RE correspondientes, y registrar configuración, semillas, versiones, procedimiento y resultados. Número de ejecuciones, episodios, métricas y criterio de comparación están pendientes.
- Trazabilidad actual: decisiones del usuario → RF-001 a RF-011 → CA-01 a CA-15 → PP-01 a PP-15, mediante las relaciones de la tabla. Se conservan los identificadores existentes. A1 → RF-002, RF-003, RF-009 → CA-13 / PP-13; A2 → RF-001, RF-002, RF-009 → CA-14 / PP-14; A3 → RF-009 a RF-011 → CA-15 / PP-15 y CA-12 / PP-12. Se aplica la ausencia de efectos secundarios de CA-10. No existen todavía referencias asociadas a implementación, pruebas ejecutables o resultados.
- US, RNF, RE, ADR y EXP relacionados: pendientes de creación o vinculación cuando corresponda; este borrador no les asigna identificadores ni inventa su contenido.

### Trazabilidad cruzada con SPEC-002

Los RF-012 a RF-022 y los CA/PP de la columna de destino pertenecen a [SPEC-002 v0.1](SPEC-002-context-and-searcher.md). Sus decisiones expresas se incorporan por referencia; no se aprueba por inferencia el borrador completo ni su propuesta de protocolo estadístico.

| Referencia local de SPEC-001 | Requisitos de SPEC-002 | Verificación prevista en SPEC-002 |
| --- | --- | --- |
| RF-007; CA-07 / PP-07 | RF-012 a RF-019: contexto, pista, matriz, política y libertad humana | CA-01 a CA-08 / PP-01 a PP-08 |
| RF-008; CA-08 / PP-08 | RF-012 a RF-022: modos, contexto, política, separación y reproducibilidad | CA-01 a CA-11 / PP-01 a PP-11 |
| RF-010; CA-12 / PP-12 | RF-012, RF-013, RF-021 y RF-022: contexto al reiniciar, pista, repetición y registro | CA-01, CA-02, CA-10 y CA-11 / PP-01, PP-02, PP-10 y PP-11 |

Las fórmulas, matriz, criterios y pruebas específicos del muestreo se mantienen en su fuente. Las PP locales comprueban su integración con las reglas del episodio. La organización técnica de semillas y registros y el protocolo estadístico completo siguen pendientes según SPEC-002.

## Exclusiones

- Navegación inteligente o desplazamiento del NPC hasta el escondite: fase 2.
- Cambio dinámico de escondite o aprendizaje sobre cuándo permanecer o cambiar: fase 3.
- Probabilidades de descubrimiento por proximidad o visibilidad.
- Físicas de proyectiles para lanzar chontaduros; la interacción es conceptual.
- Aprendizaje del buscador automatizado: en el MVP su política es fija.
- Elección de tecnología de entrenamiento, algoritmo de RL, mecanismo de integración con Unity, bibliotecas o dependencias. Unity y C# sí están aprobados para la presentación 2D; esta aprobación no determina la tecnología del entrenamiento ni la de su integración.

## Decisiones pendientes

1. Definir para SPEC-003 el nombre y la codificación definitiva de la zona de atención como observación del agente, incluido aprobar o revisar el campo propuesto `attention_zone`. Las zonas, su generación y la pista compartida están definidas en SPEC-002, RF-012 y RF-013; no están pendientes. No asumir un mapeo espacial entre cinco escondites y tres zonas.
2. Resuelto en SPEC-002, RF-014 a RF-019: matriz de pesos, relación del contexto con el riesgo, política fija y segunda inspección tras un fallo, sin restringir al humano. Se conserva este punto para trazabilidad; no queda una decisión funcional pendiente sobre esa política.
3. Aprobar o revisar las recompensas `+1` y `-1`, su momento de entrega y la existencia o ausencia de recompensas intermedias.
4. Concretar en SPEC-003 si la observación del agente es suficiente y si existe información adicional autorizada. No asumir historial de partidas. El buscador no accede al escondite ocupado, conforme a SPEC-002, RF-018.
5. Definir otros casos no cubiertos por RF-009, abandonos, errores y truncamientos técnicos. A1, A2 y A3 quedan resueltos en RF-002, RF-009 y RF-010. No están pendientes los rechazos ya aprobados ni el restablecimiento definido en RF-010.
6. Definir la presentación identificable de los escondites y de la pista al humano, sin introducir requisitos de navegación o físicas.
7. Seleccionar tecnología de entrenamiento, algoritmo de RL, mecanismo y contratos de integración con Unity y presupuesto de entrenamiento mediante las especificaciones y ADR pertinentes. Unity y C# ya están aprobados para la presentación; la lógica debe permanecer separada.
8. Definir la política aleatoria de referencia del NPC y el protocolo reproducible: escenarios, semillas, repeticiones, episodios, métricas definitivas y criterio de comparación.

Resolver y aprobar los puntos que afecten a un comportamiento antes de implementarlo. La aprobación de este borrador completo y sus futuras revisiones debe quedar explícita.

Los puntos 1, 3, 4 y 7 bloquean cerrar el contrato del agente y la integración de aprendizaje por refuerzo. La generación del contexto y la política fija del buscador ya están definidas en SPEC-002; el punto 2 está resuelto. El punto 8 bloquea la evaluación comparativa definitiva. Estas limitaciones no convierten en propuestas las reglas funcionales ya aprobadas.
