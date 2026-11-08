# SPEC-002 — Contexto y buscador automatizado

- **Estado:** BORRADOR; no aprobado como especificación completa.
- **Versión:** v0.2.
- **Fecha:** 2026-11-08.
- **Ámbito:** fase 1 del TFM MVP.
- **Fuentes:** [AGENTS.md](../AGENTS.md), [visión](../docs/vision.md), [misión](../docs/mission.md) y [SPEC-001 v0.6](SPEC-001-hiding-environment.md).
- **Origen de las decisiones:** instrucciones explícitas del usuario para SPEC-002 v0.1, incluida la matriz de pesos aprobada. El estado BORRADOR no convierte decisiones todavía abiertas en requisitos autorizados ni revoca las decisiones expresas aquí recogidas.

## Propósito y alcance

Definir la generación del contexto y la política probabilística fija del buscador automatizado utilizada durante el entrenamiento y la evaluación del NPC. El contexto modifica las probabilidades de elegir los escondites que se inspeccionan; el descubrimiento al inspeccionar el escondite ocupado sigue siendo determinista según SPEC-001.

Se especifican la pista compartida, la configuración de referencia, el muestreo ponderado sin repetición y los requisitos conceptuales de reproducibilidad. No se implementa código ni se fijan clases, interfaces, APIs, bibliotecas o formatos de almacenamiento.

## Dependencias y relación con SPEC-001

SPEC-001 es la fuente de las reglas del episodio: escondites RF-001, selección fija del NPC RF-002, presupuesto RF-003, prohibición de reinspección RF-004, victorias RF-005 y RF-006, contexto RF-007, modos RF-008, rechazos RF-009, reinicio RF-010 y presentación RF-011. Estos identificadores se refieren a SPEC-001; no se redefinen aquí.

Esta especificación concreta los pendientes de contexto y buscador de RF-007, RF-008 y RF-010 y de los puntos 1 y 2 de Decisiones pendientes de SPEC-001. Aporta contenido para sus CA-07, CA-08 y la generación contextual de CA-12. No modifica ese archivo ni declara aprobado su borrador completo.

**Consistencia documental:** SPEC-001 BORRADOR v0.6 incorpora por referencia las decisiones de SPEC-002 sobre las tres zonas, su generación, la pista compartida, la matriz de pesos y la política fija del buscador automatizado, y actualiza su trazabilidad cruzada. La sincronización documental está realizada y no modifica las reglas del episodio. Los nombres del campo definitivo de observación y su codificación siguen pendientes.

No hay contradicción funcional con la inspección determinista: solo se sortea el objetivo de inspección. Tampoco se establece una correspondencia espacial entre las tres zonas y los cinco escondites: la matriz define pesos de atención, no posiciones físicas.

## Conceptos y definiciones

- **Zona de atención o contexto:** exactamente uno de `NORTH` (Norte), `CENTER` (Centro) o `SOUTH` (Sur), constante durante el episodio. No representa la posición real del NPC.
- **Peso:** valor positivo asociado a un par contexto–escondite; no es directamente una probabilidad.
- **Disponibles:** escondites existentes aún no inspeccionados en el episodio, conforme al estado lógico de SPEC-001.
- **Política fija probabilística:** regla de selección aleatoria con pesos que no se aprenden ni cambian durante entrenamiento o evaluación. Fija no significa que repita siempre la misma acción.
- **Configuración equivalente para repetición:** mismas semillas, pesos, condiciones iniciales, secuencia de episodios y acciones del NPC, y mismas versiones y reglas de generación y consumo de aleatoriedad. Una semilla aislada no garantiza equivalencia entre tecnologías distintas.

## Requisitos

Los RF-012 a RF-022 continúan la numeración existente sin modificar ni reutilizar RF-001 a RF-011. Los criterios CA y pruebas PP son referencias locales de SPEC-002; deben citarse con su especificación para distinguirlos de los de SPEC-001.

| ID | Requisito |
| --- | --- |
| RF-012 | Seleccionar al inicio de cada episodio exactamente un contexto de las tres zonas definidas, de forma independiente entre episodios y con probabilidad uniforme de 1/3 por zona. Mantenerlo constante hasta finalizar. La generación admite semilla configurable. No imponer cuotas por bloques de episodios. |
| RF-013 | Entregar la zona de atención al NPC antes de su selección y mostrar la misma pista al humano durante la partida interactiva, antes de sus inspecciones. El buscador automatizado utiliza ese contexto. La pista no revela el escondite del NPC ni las inspecciones futuras; no existe opción de ocultarla en el MVP. |
| RF-014 | Utilizar la matriz de referencia de esta especificación como configuración inicial del MVP. Los pesos son positivos y parametrizables, sin configuraciones alternativas aquí definidas ni mecanismos de ajuste dinámico. Registrar la configuración utilizada y mantenerla fija durante entrenamiento y evaluación. |
| RF-015 | Para la primera inspección automatizada, muestrear entre los cinco escondites usando el peso de cada uno dividido entre la suma de la fila del contexto. Aplicar las precondiciones y validaciones del entorno de SPEC-001. |
| RF-016 | Tras una primera inspección válida fallida, excluir su objetivo y seleccionar la segunda mediante los pesos originales de los cuatro restantes, renormalizados exclusivamente por su suma. No repetir escondites ni modificar los pesos para compensar el fallo. |
| RF-017 | Respetar las transiciones y victorias de SPEC-001: detener inmediatamente las inspecciones al descubrir al NPC, sin segunda inspección tras un acierto inicial; tras la segunda inspección finalizar con la victoria correspondiente. Nunca realizar más de dos inspecciones válidas ni continuar en estado terminal. |
| RF-018 | La política del buscador selecciona usando únicamente el contexto, los disponibles y su aleatoriedad conforme a los pesos fijos. No recibe ni consulta el escondite real del NPC. El entorno resuelve cada inspección y determina si procede continuar. El buscador no aprende durante entrenamiento ni evaluación. |
| RF-019 | En modo humano, permitir elegir libremente cualquier escondite válido según SPEC-001, independientemente de su peso o de la pista. La matriz no restringe ni sortea las elecciones humanas. |
| RF-020 | Separar responsabilidades según la sección siguiente. La selección probabilística y el estado lógico deben funcionar sin representación visual, animaciones ni físicas; Unity muestra pistas y resultados sin ser dependencia de esa lógica. |
| RF-021 | Permitir repetir contextos y decisiones del buscador con semillas configurables y configuración equivalente. Separar conceptualmente su aleatoriedad de la del NPC, de modo que el consumo aleatorio del NPC no determine los sorteos del contexto o del buscador. No imponer todavía cómo se organizan los generadores o las semillas. |
| RF-022 | Registrar semillas y configuración de pesos utilizada, contexto de cada episodio, inspecciones efectivamente realizadas y resultado, con las versiones necesarias para la repetición. No registrar una segunda inspección como realizada si hubo acierto inicial. No se define aquí persistencia ni formato de registros. |

### Separación de responsabilidades

| Responsabilidad | Contrato conceptual |
| --- | --- |
| Entorno de simulación | Mantiene estado, disponibles, presupuesto y condiciones terminales; valida y resuelve las inspecciones conforme a SPEC-001. Conoce el escondite ocupado para resolver el resultado, sin exponerlo al buscador. |
| Generador de contexto | Selecciona la zona al inicio conforme a RF-012; no decide el escondite del NPC ni los objetivos de inspección. |
| Política del buscador automatizado | Selecciona objetivos con los pesos del contexto y los disponibles; no decide victorias ni modifica el estado del entorno directamente. |
| Política del NPC | Selecciona dónde esconderse. Su aprendizaje se especificará posteriormente en SPEC-003, aún no creada ni aprobada por este documento. |
| Presentación Unity | Muestra la pista y los resultados conforme a SPEC-001, sin determinar las decisiones probabilísticas ni las transiciones lógicas. |

## Política probabilística y fórmulas

### Matriz de referencia del MVP

| Contexto | E1 | E2 | E3 | E4 | E5 |
| --- | ---: | ---: | ---: | ---: | ---: |
| NORTH | 6 | 5 | 2 | 2 | 3 |
| CENTER | 2 | 3 | 5 | 4 | 6 |
| SOUTH | 2 | 2 | 6 | 5 | 3 |

Las sumas de las filas son, respectivamente, 18, 20 y 18. La parametrización no autoriza cambios durante una ejecución ni añade variantes de referencia al MVP.

### Generación del contexto

Para cada episodio t y cualquier zona c:

\[
P(C_t=c)=1/3.
\]

Los contextos se sortean independientemente entre episodios. Pueden repetirse, incluido tras reiniciar una partida finalizada; no se exige un ejemplar de cada zona por cada tres episodios. La independencia es una propiedad del modelo de muestreo, compatible con repetir una secuencia pseudoaleatoria mediante semilla.

### Selección de inspecciones

Sea H = {E1, E2, E3, E4, E5}, w(c,i) el peso positivo de i en c y W(c) la suma de los cinco pesos. Para la primera inspección I₁:

\[
P(I_1=i\mid C=c)=\frac{w(c,i)}{W(c)}.
\]

Tras fallar en j, la segunda inspección I₂ se realiza entre H sin j:

\[
P(I_2=i\mid I_1=j,\text{fallo},C=c)=
\begin{cases}
\dfrac{w(c,i)}{W(c)-w(c,j)}, & i\ne j,\\
0, & i=j.
\end{cases}
\]

En ambos sorteos las probabilidades disponibles son positivas y suman uno. El segundo sorteo conserva la proporción entre los pesos de cualesquiera dos escondites restantes. No se realiza si la primera inspección descubre al NPC.

### Probabilidad total de descubrimiento

Para un NPC escondido en i y un contexto fijo c, los casos «acierto inicial en i» y «primer fallo en j distinto de i seguido de acierto en i» son disjuntos:

\[
P(\text{descubrir}\mid \text{NPC}=i,C=c)=
\frac{w(c,i)}{W(c)}+
\sum_{j\in H,\,j\ne i}
\frac{w(c,j)}{W(c)}\,
\frac{w(c,i)}{W(c)-w(c,j)}.
\]

No debe confundirse esta probabilidad con la de la primera inspección. La probabilidad de que el NPC gane, condicionada al mismo escondite y contexto, es su complemento. Son propiedades teóricas de la política, no resultados experimentales de aprendizaje.

### Ejemplo NORTH

Con pesos (6, 5, 2, 2, 3), la primera inspección elige E1 con probabilidad 6/18 = 1/3. Si falla en E1, los disponibles tienen pesos (5, 2, 2, 3), suma 12; la probabilidad de inspeccionar E2 en segundo lugar es 5/12.

Si el NPC está en E1, la probabilidad total de descubrirlo es:

\[
\frac{6}{18}+
\frac{5}{18}\frac{6}{13}+
2\frac{2}{18}\frac{6}{16}+
\frac{3}{18}\frac{6}{15}
=\frac{159}{260}\approx0{,}61153846.
\]

El caso anterior de fallo en E1 y este cálculo de NPC en E1 son escenarios distintos. Las sumas, fracciones y aproximación del ejemplo se comprobaron mediante aritmética racional al redactar esta versión; no constituyen pruebas ejecutadas de una implementación.

## Criterios de aceptación

| Criterio local | Resultado observable |
| --- | --- |
| CA-01 | Solo se generan las tres zonas, con probabilidad teórica 1/3 cada una e independencia entre episodios, sin cuotas por bloques; el contexto no cambia dentro del episodio. |
| CA-02 | NPC y humano reciben la misma pista en los momentos definidos; el buscador usa ese contexto. La pista no contiene el escondite ocupado ni objetivos futuros y no puede ocultarse mediante una opción del MVP. |
| CA-03 | Cada contexto selecciona exactamente su fila de referencia; sus pesos permanecen positivos e invariables durante entrenamiento y evaluación y su configuración queda identificada. |
| CA-04 | La primera selección tiene las probabilidades de la fórmula, cuya suma es uno en cada contexto. |
| CA-05 | Tras un fallo inicial, el objetivo fallido tiene probabilidad cero; los cuatro pesos restantes conservan su valor original y sus probabilidades suman uno tras renormalizar. |
| CA-06 | Los episodios automatizados no repiten objetivos, terminan en el primer acierto y nunca superan dos inspecciones; las victorias y efectos lógicos coinciden con SPEC-001. |
| CA-07 | Para contexto, disponibles y estado aleatorio equivalentes, variar únicamente la posición oculta del NPC no cambia el objetivo seleccionado por el buscador. La posición sí puede cambiar el resultado resuelto por el entorno y la existencia de una segunda inspección. |
| CA-08 | El humano puede elegir cualquiera de los objetivos válidos en cada contexto, sin restricciones adicionales por pesos; sigue sujeto a presupuesto, rechazos y terminalidad de SPEC-001. |
| CA-09 | La generación, selección y resolución lógica funcionan sin gráficos y producen los mismos resultados al conectar o desconectar la presentación con las mismas entradas y aleatoriedad. |
| CA-10 | La repetición bajo configuración equivalente reproduce contextos, inspecciones realizadas y resultados. Consumir aleatoriedad del NPC sin cambiar sus acciones no altera los sorteos del contexto ni del buscador. |
| CA-11 | Los registros permiten identificar semillas, pesos, versiones y, por episodio, contexto, inspecciones realizadas y resultado; no incluyen inspecciones no realizadas como si lo fueran. |

## Pruebas previstas

Todas las PP siguientes son **pruebas previstas, no implementadas ni ejecutadas**. Las comprobaciones estadísticas son propuestas de verificación, no un protocolo experimental aprobado ni evidencia de mejora del NPC.

| Prueba local | Método previsto |
| --- | --- |
| PP-01 | Revisar la regla de generación uniforme e independiente y ausencia de cuotas. Comprobar dominio y constancia del contexto durante selección, inspecciones y resultado. Proponer una comprobación estadística de frecuencias y dependencia entre episodios según las tolerancias indicadas abajo. |
| PP-02 | Verificar entrega previa al NPC, igualdad y visibilidad de la pista humana durante la partida y uso por el buscador. Revisar que la pista solo comunica la zona y que no existe opción de ocultarla. |
| PP-03 | Comparar las tres filas con la matriz, comprobar positividad y sumas 18, 20, 18; comparar la configuración antes y después de secuencias de entrenamiento y evaluación para descartar aprendizaje o ajuste del buscador. |
| PP-04 | Para cada contexto, contrastar las cinco probabilidades iniciales con cocientes exactos de pesos y comprobar su suma. Complementar con frecuencias de muestreo bajo el protocolo estadístico que se apruebe. |
| PP-05 | Para cada contexto y cada primer objetivo fallido, comprobar exclusión, conservación de los cuatro pesos, denominador correcto, suma uno y proporciones relativas. Verificar en NORTH tras fallar E1 el valor 5/12 para E2. |
| PP-06 | Cubrir cada escondite del NPC y las secuencias de acierto inicial, fallo seguido de acierto y dos fallos distintos. Comprobar presupuesto, inspeccionados y victoria según SPEC-001; ausencia de repeticiones y de solicitudes adicionales tras el final. Contrastar la fórmula total con la enumeración de los caminos disjuntos de primera y segunda inspección. |
| PP-07 | Comparar selecciones con igual contexto, disponibles y estado aleatorio, variando únicamente el escondite oculto. En la segunda decisión comparar estados equivalentes en los que el primer objetivo fue fallido en ambos casos; no exigir igual longitud de episodios si una posición produce acierto inicial. Revisar que la política no recibe la posición oculta. |
| PP-08 | En cada contexto y con presupuestos válidos, permitir al humano elegir cualquiera de los escondites disponibles. Comprobar que no hay sorteo ni bloqueo por pesos y que permanecen los rechazos de SPEC-001. |
| PP-09 | Ejecutar conceptualmente las mismas secuencias con y sin presentación y contrastar contexto, objetivos, presupuesto, inspeccionados, resultado y terminalidad, sin esperar animaciones ni físicas. |
| PP-10 | Repetir una secuencia de episodios con semillas, versiones, pesos y acciones del NPC equivalentes; comparar contextos, objetivos y resultados. Repetir consumiendo aleatoriedad adicional del NPC sin alterar sus acciones y comprobar que no cambia la secuencia del contexto/buscador. No exigir que semillas diferentes produzcan siempre resultados distintos. |
| PP-11 | Revisar la información registrada contra las inspecciones efectivas, con las tres formas de finalización de PP-06. Comprobar semillas, pesos, versiones, contexto y resultado sin imponer un formato o almacenamiento. |

### Propuesta de comprobaciones estadísticas

Priorizar comprobaciones exactas de las reglas y normalizaciones; una muestra finita no demuestra uniformidad ni independencia. No exigir frecuencias observadas idénticas a las probabilidades teóricas.

Como propuesta pendiente de aprobación en el protocolo, para n sorteos independientes de una categoría de probabilidad teórica p, usar una tolerancia absoluta ε(n,α) = √(ln(2/α)/(2n)), derivada de la cota de Hoeffding para indicadores. Esta cota limita por α la probabilidad de superar la tolerancia bajo el modelo esperado. Si se comprueban m categorías o condiciones conjuntamente, proponer α/m por comprobación para controlar el error familiar mediante la cota de la unión. El tamaño n, α, número de comparaciones y semillas de verificación quedan pendientes; no se fija un umbral numérico arbitrario.

Aplicar la propuesta a contextos y sorteos ponderados en condiciones controladas. Para la segunda inspección, fijar contexto y primer objetivo fallido y comparar solo los cuatro disponibles. Para detectar dependencia temporal de contextos, proponer pares de episodios no solapados y contrastar las nueve probabilidades conjuntas teóricas de 1/9, además de revisar el mecanismo de generación; esto no demuestra ausencia de toda dependencia posible. Ninguna prueba estadística aislada basta para validar el contrato.

## Exclusiones

- Algoritmo de RL, observaciones y recompensas definitivas, y entrenamiento del NPC: se abordarán posteriormente en SPEC-003 según corresponda.
- Política aleatoria de referencia del NPC y protocolo experimental completo: se abordarán en EXP-001 o especificaciones posteriores; no quedan aprobados aquí.
- Persistencia de resultados, formatos, estructura de archivos, generadores específicos, clases, interfaces, APIs, bibliotecas e integración concreta: requieren decisiones posteriores, mediante ADR cuando corresponda.
- Navegación, cambio de escondite durante el episodio y multijugador.
- Ajuste dinámico, aprendizaje del buscador y configuraciones alternativas a la matriz de referencia.

Las referencias a SPEC-003 y EXP-001 son destinos futuros, no documentos aprobados ni creados por esta especificación.

## Decisiones pendientes

1. Revisión y aprobación del borrador completo. La sincronización documental de los aspectos correspondientes ya se realizó en SPEC-001 BORRADOR v0.6.
2. Contrato definitivo de observación y su codificación, incluidas posibles informaciones adicionales; definir las recompensas y el aprendizaje en el documento correspondiente. Las zonas y su distribución aquí indicadas sí están decididas.
3. Generadores y organización técnica de la aleatoriedad y semillas que materialicen RF-021; contratos de integración, tecnología de entrenamiento y ADR pertinentes, sin dependencia visual.
4. Formatos y mecanismos de registro y futura persistencia, sin cambiar los datos conceptuales exigidos por RF-022.
5. Protocolo de verificación estadística: tamaño muestral, tolerancias y control de error, semillas y comparaciones; protocolo experimental completo, métricas, repeticiones y política aleatoria del NPC.
6. Diseño visual concreto de la pista y los escondites. La disponibilidad de la pista y la libertad del humano no están pendientes.
7. Casos no cubiertos, abandonos, errores y truncamientos técnicos que SPEC-001 mantiene abiertos, incluido el tratamiento concreto de configuraciones de pesos inválidas; no inferir una API ni resultados de error.

No están pendientes la matriz de referencia, el carácter fijo del buscador, la renormalización sin repetición ni las reglas explícitas de contexto y pista de esta solicitud. No se autoriza implementar los demás aspectos por inferencia.

## Matriz de trazabilidad

Las referencias CA y PP de esta tabla son locales a SPEC-002. Las reglas del episodio se verifican en su fuente y se reutilizan en las comprobaciones de integración, sin duplicar requisitos.

| Requisito | Relación con SPEC-001 | Criterio | Prueba prevista |
| --- | --- | --- | --- |
| RF-012 | RF-007, RF-010; CA-07, CA-12 | CA-01 | PP-01 |
| RF-013 | RF-007, RF-008; CA-07 | CA-02 | PP-02 |
| RF-014 | RF-007, RF-008; CA-08 | CA-03 | PP-03 |
| RF-015 | RF-001, RF-003, RF-008, RF-009 | CA-04 | PP-04 |
| RF-016 | RF-003, RF-004, RF-008 | CA-05, CA-06 | PP-05, PP-06 |
| RF-017 | RF-003 a RF-006, RF-009; CA-03 a CA-06, CA-11 | CA-06 | PP-06 |
| RF-018 | RF-007, RF-008; CA-08 | CA-03, CA-07 | PP-03, PP-07 |
| RF-019 | RF-007, RF-008, RF-009 | CA-08 | PP-08 |
| RF-020 | RF-008, RF-011; CA-08, CA-09 | CA-09 | PP-09 |
| RF-021 | Separación y trazabilidad experimental | CA-10 | PP-10 |
| RF-022 | Separación y trazabilidad experimental | CA-11 | PP-11 |

No hay implementación, pruebas ejecutables ni resultados experimentales vinculados todavía. No se crean historias artificiales ni se asignan US, RNF, RE, ADR o EXP sin contenido definido. Los futuros experimentos deberán vincular esta especificación y los requisitos experimentales que se aprueben.
