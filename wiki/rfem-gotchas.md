---
name: rfem-gotchas
description: Comportamientos no obvios de RFEM 6 descubiertos en la práctica. Revisar antes de arrancar un modelo nuevo.
status: ACTIVO
last_updated: 2026-08-31
---

# RFEM — gotchas y comportamientos no obvios

## Wind Load Wizard

- Tipo **"Duopitch/Flat/Monopitch roof"** = solo cubiertas EXENTAS sin pared (marquesinas). Para un edificio real con paredes, usar **"Vertical Walls with Roof"**.
- Nodos base I,J,K,L: deben estar en el piso real (z=0) de las paredes, alineados verticalmente bajo los nodos de techo correspondientes (A bajo I, etc.). Nunca en una altura intermedia.
- Con voladizo de cubierta: usar nodo superior de PARED como referencia, no el borde exterior del techo volado.
- Edificio escalonado (dos volúmenes de distinta altura adosados): el wizard NO lo resuelve en una sola corrida. Correr una corrida por volumen, y excluir manualmente de "Loaded" la pared compartida entre ambos (no recibe viento directo).
- Tabla "Generated on Members/Surfaces/Lines No.", columna "Loaded": si queda vacía, la carga NO se distribuye a los elementos — se queda como bloque de superficie sin repartir, sin aviso de error. Hay que seleccionar ahí explícitamente qué miembros reciben la carga.
- No confundir "Without Load Parallel to" (excluye un miembro puntual de recibir carga aunque esté en el plano) con la selección de "Loaded" (sin la cual no hay redistribución en absoluto) — son pasos distintos, hace falta hacer ambos.
- El wizard NO genera automáticamente: presión de voladizo de cubierta (Art. 5.11.4, Cp=0,8 en cara inferior) ni zonas especiales de techo escalonado — cargar manual.

## Snow Load Wizard

- Solo reconoce geometría rectangular plana, monopitch o duopitch simple. NO genera arrastre (Cap.7) ni deslizamiento (Cap.9) en techos escalonados — manual.

## Free Rectangular Load (cargas manuales tipo arrastre, no balanceada custom, etc.)

- A diferencia del Snow/Wind Wizard (que reparte directo a Members), **Free Rectangular Load exige una Surface real asignada** ("Assigned to Surfaces No."). Con techos modelados solo con correas/cabios (sin Surface), crear una Surface nueva con **Stiffness Type** que NO aporte rigidez real, solo para transferencia de carga — buscar en el desplegable una opción tipo "Load transfer surface" (habilita el checkbox "Apply loads using the load transfer surface", que aparece en gris con Stiffness Type "Standard").
- Distribución "Linear in X/Y": el **signo de la coordenada** del segundo corner point define hacia qué lado crece/decrece la rampa — si el triángulo sale espejado hacia el lado contrario a la superficie real, invertir el signo de esa coordenada (no alcanza con elegir bien la distribución). Verificar en **vista 3D/isométrica**, no en planta — en vista de planta (top view) no se nota la altura del triángulo y puede parecer que la carga "no se ve" o se ve como una simple línea.

## Load Transfer Surface — restringir qué miembros reciben la carga

Por defecto, una Surface de reparto (Stiffness Type "Load transfer surface") tiende a repartir la carga a TODOS los miembros dentro de su huella, no solo a las correas — problema cuando conviven correas y cabreadas/cabios en el mismo paño (la carga real de chapa apoya en correas primero, no directo en la cabreada). Se soluciona en las propiedades de la Surface, pestaña **"Load Transfer"** → campo **"Remove influence from members"** → seleccionar ahí los miembros que NO deben recibir carga (cabreadas, cabios, etc.), dejando que solo las correas la reciban. Puede haber una forma más directa de seleccionar positivamente "solo estos miembros" en vez de excluir el resto — pendiente de confirmar si existe.

## Visualización

- Clic derecho sobre una carga generada → **"Display separately"**: cambia de vista de bloque de superficie a carga real por elemento. Usar para diagnosticar si una redistribución (a correas, por ejemplo) funcionó.

## Casillas de opciones de wizards

- **"Lock for new objects"**: destildada mientras se sigue modelando (la carga se extiende sola a miembros nuevos al regenerar). Tildar solo cuando el modelo esté cerrado, para "congelar" el resultado.
- **"Consider member eccentricity"**: destildada por defecto — tildarla EXCLUYE del reparto a miembros con excentricidad fuera de plano (comportamiento contraintuitivo). Solo tildar si hay excentricidades deliberadas que deban quedar afuera.
- **"Consider section distribution"**: solo relevante con secciones ahusadas (tapered) — sin efecto en secciones prismáticas.

## Add-ons (Edit Model → Add-ons)

- Un add-on puede aparecer con punto **verde ("Purchased")** pero el checkbox destildado — significa que la licencia está pero no está activado para ESE modelo. Hay que tildarlo ahí antes de que la funcionalidad esté disponible.
- Modal Analysis / Response Spectrum Analysis: necesarios para método dinámico sísmico. No corrsr sin modelo de masa completo (correas, chapas, todo cargado) — con masa incompleta, el período fundamental sale sesgado.

## Normas de referencia (ASCE dentro de RFEM)

- RFEM no tiene CIRSOC nativo para viento/nieve — usar la edición ASCE más cercana a la base real del CIRSOC vigente (ver `normativa-vigente.md`) y validar manualmente contra las tablas argentinas.
- El generador de combinaciones de carga puede no ofrecer la misma edición ASCE que el generador de cargas — no es un problema real si el factor de carga de viento (1,0) es igual en ambas ediciones, lo cual aplica desde ASCE 7-10 en adelante.
- Diferencias de qh >5% entre ediciones ASCE casi siempre son un dato mal cargado (unidades, campo por defecto en otra unidad), no una diferencia editorial genuina.

## Action Combinations (generador automático) — desactivado a propósito en este proyecto

El generador automático de **Action Combinations** (pestaña del mismo nombre en Load Cases & Combinations) arma TODAS las combinaciones posibles a partir de las Actions asignadas (D, W, S, Lr, E...) y **no dejaba borrar selectivamente para volver a generar solo algunas** — quedó desactivado (28/08) para poder armar las combinaciones de sismo a mano (necesario por el plegado de Ev en el factor de D y el chequeo aislado del voladizo, reglas que el generador genérico no conoce).

**Para usarlo sin perder las combos manuales de sismo**: en la Action Combination (ej. AC1) → tab **Assignment** → de la lista "To Assign" (A1 Dead, A2 Wind, A3 Snow, A4 Roof live, A5/A6 Earthquake) seleccionar con `>>` **solo D, W, S, Lr** — dejar afuera las Earthquake load (A5/A6). Así genera nomás las combinaciones de gravedad+viento (ASCE 7 §2.3.1 básicas) sin tocar ni duplicar las de sismo, que quedan armadas a mano en la pestaña Load Combinations.

**Ojo: esto NO alcanza solo — el generador igual mete sismo.** ASCE 7 §2.3.1 (la norma detrás de "Section 2.3 LRFD") incluye NATIVAMENTE 2 de sus 7 fórmulas básicas con término E (1,2D+1,0E+L+0,2S y 0,9D+1,0E) — el generador las arma igual aunque no hayas tildado Earthquake en el Assignment de una AC puntual, porque es la lista completa de la norma. Si además tenés varias Load Cases de sismo agrupadas en una sola Action (ej. LC34-38 bajo "Earthquake load"), el generador las trata como **simultáneas por defecto** y las suma todas juntas en una sola combinación — un desastre (llegamos a 629 combinaciones, 279 mal armadas sumando direcciones opuestas y hasta Fv+Fvup juntas).

**Solución real — Load Case Relations (Exclusive Load Cases)**: Design Situations → DS correspondiente → tildar **"Consider inclusive/exclusive load cases"** → editar la relación (ícono para crear/editar "Relationship Between Load Cases") → tab **Exclusive Load Cases**. Ahí NO se puede poner el mismo rango en "Selected Load Cases" y "Do Not Combine with Load Cases" (tira error: *"The load case is included in the selected load cases"*) — hay que armarlo **par por par**, una fila por cada carga excluida de las demás:

| Selected Load Cases | Do Not Combine with Load Cases |
|---|---|
| LC34 | LC35-38 |
| LC35 | LC34, LC36-38 |
| LC36 | LC34-35, LC37-38 |
| LC37 | LC34-36, LC38 |
| LC38 | LC34-37 |

Con esto, el generador nunca combina dos de esas cargas en la misma combinación — quedan como alternativas mutuamente excluyentes, no sumables. Asignar la relación a la Design Situation correspondiente (campo "Assignment to Design Situations" en el mismo diálogo).

**Aun así, las combinaciones de sismo que arma el generador quedan con 1,20D/0,90D genérico, sin el plegado de Ev específico de INPRES-CIRSOC** (nuestro 1,278/0,822) — son válidas en su forma pero menos precisas, y en el lado de 0,9D son *menos* conservadoras que las nuestras para chequeos de arrancamiento (0,90D+E tiene más D disponible que 0,822D+E). Mejor borrarlas después de generar y quedarse solo con las combinaciones de sismo armadas a mano.

## Materiales

- Perfiles W: material ASTM A992 (o A572 Gr.50 si A992 no aparece en la biblioteca — mismo Fy/Fu).

## Análisis Modal / Dinámico — gotchas descubiertos en la sesión del 27/08

- **"Node No." vs "FE Mesh Node No." son numeraciones DISTINTAS**. Un error de cálculo que dice "FE mesh node No. X" NO es el nodo estructural X que vos dibujaste — es un punto interno de la malla. Para ubicarlo: **Edit → Find via Number**, cambiar el tipo de objeto de "Node" a **"FE Mesh Node"** antes de buscar. Un apoyo/restricción puesto en el "Node" equivocado no tiene ningún efecto aunque el número coincida.
- **"The stiffness matrix is singular... around axis Y" (rotación, no traslación) en nodos donde solo llegan miembros Truss**: es esperable y no es error de modelado — un Truss (only N) no tiene rigidez rotacional, entonces el giro en ese nodo queda matemáticamente indeterminado (sin masa ni rigidez asociada). Fix de raíz (mejor que apoyos puntuales nodo por nodo): en **Modal Analysis Settings (MOS1) → Mass Matrix Settings → "About axis"**, destildar las 3 rotaciones (X,Y,Z) — así la masa rotacional queda excluida globalmente y el problema no vuelve a aparecer en ningún cruce de arriostramiento.
- **Correas con rótula en ambos extremos (simplemente apoyadas) generan el mismo tipo de error en el nodo del pórtico**: al liberar los 3 giros de la correa, el lado "correa" del nodo queda con una rotación desacoplada sin masa/rigidez — mismo fix de Mass Matrix Settings de arriba.
- **Corrección (28/08) — liberar los 3 giros de la correa NO es la práctica estándar**: se investigó en fuentes (SCI/newsteelconstruction.com, steelconstruction.info, AISI Design Guide, papers de conexión correa-chapa) y la conexión real correa-cabio (clip/ménsula) SÍ aporta restricción torsional al pórtico — es justamente el mecanismo por el que las correas arriostran el ala comprimida del cabio contra pandeo lateral-torsional. Liberar las 3 le saca esa restricción al modelo y generaba matriz singular en el nodo del pórtico (además de la rotación de masa desacoplada de arriba) — no era un gap real de arriostramiento, era un error de modelado. **Fix**: liberar SOLO el momento de flexión de eje mayor de la correa (My o Mz según la orientación del eje local — confirmar en cada proyecto), dejar el otro eje de flexión y la torsión rígidos. Mantiene la hipótesis "correa simplemente apoyada" a gravedad sin sacarle al pórtico la restricción torsional real. Con esta corrección, el apoyo puntual parche (ver `metodo-inpres-cirsoc103-sismo.md`) deja de ser necesario.
- **Cable vs. Truss para arriostramiento de acero**: el tipo **Cable** saca su rigidez dinámica de la pretensión (pensado para cables reales con catenaria — pasarelas colgantes, cubiertas tensadas). Sin pretensión bien definida, la rigidez linealizada para modal puede quedar casi en cero, generando nodos singulares en cadena a lo largo de la malla interna del cable (cada vez que se "arregla" uno aparece otro cerca). Para diagonales rectas de acero (X-bracing), usar **Truss (only N)** + no linealidad **"Failure if compression"** (ojo: "Failure if tension" es la opción CONTRARIA, deja el miembro trabajando solo a compresión — fácil de confundir por el nombre).
- **Renombrar un nodo con ID fuera de rango** (ej. un nodo corrupto con número >1.000.000, típico de un bug de numeración tras copiar/generar superficies): **Tools → Renumber → Individually** NO funciona si al pasar el mouse sobre el nodo aparece un ícono de "prohibido" — señal de que RFEM lo reconoce como parte de un grupo de nodos duplicados/coincidentes. Mejor: crear un nodo nuevo cualquiera, asignarle a mano las coordenadas exactas del nodo corrupto (leídas de la tabla de Nodos), y correr **Tools → Model Check → Identical Nodes** — los va a agrupar y fusionar solo, quedándose con el número válido.
- **Participación de masa modal que no sube al agregar modos** (se estanca aunque subas de 15 a 50): típico de edificios con **diafragma flexible** (techo de chapa sobre reticulado, sin superficie estructural real) — cada pórtico vibra casi independiente, entonces la masa queda repartida en decenas de modos chicos en vez de 2-3 modos globales grandes. Cambiar el tipo de matriz de masa (Consistent↔Diagonal) NO resuelve esto — el problema es de conectividad, no de formulación numérica. Fix: **Rigid Coupling tipo "Diaphragm"** (Special Objects) en los nodos de apoyo de cabreada de cada zona de techo (un diafragma por cada plano de techo distinto — no hace falta que sea horizontal, puede seguir la pendiente real si los nodos elegidos son coplanares). Esto acopla desplazamientos en el plano + rotación perpendicular, dejando libre lo demás. Ojo: agregar el diafragma reordena los modos (el modo que antes cargaba la masa en una dirección puede pasar a un número de modo más alto) — hay que volver a revisar con más modos después de agregarlo, no asumir que con los mismos modos de antes alcanza.
- Al definir el diafragma, RFEM pide 3 nodos para fijar el plano (**"Link plane by 3 nodes"**) — puede autocompletar con un nodo que NO es parte de la selección; verificar que esté a la altura/coordenada correcta antes de confirmar, si no el plano queda inclinado sin querer.
- Al tipear una lista de nodos a mano (ej. en "Nodes Assignment"), un guión colgante sin número de cierre (ej. `478-` sin nada después, típico de una selección gráfica interrumpida) tira el error "ingresar enteros o rangos entre 1 y 1000000" aunque el resto de la lista esté bien. Revisar el final del campo, o presionar Esc antes de cerrar el diálogo de selección gráfica.

## Steel Design — "Calculate All" dispara optimización iterativa de secciones (lenta)

Con el add-on **Steel Design** tildado en "Selected for Calculation", Calculate All no es solo el análisis estático — corre en secuencia: Initializing → Member result sections → **Member internal forces and deformations** (esto ES el análisis estático, con desplazamientos ya disponibles acá) → **Optimization** → Design of members → Processing results.

La Optimization es un **loop iterativo**: busca la sección más chica que cumple (dentro de la serie de perfiles), y como cambiar la sección cambia la rigidez del modelo, recalcula el análisis completo de nuevo — con proyectos de muchos miembros y Result Combinations grandes (este proyecto: 18 casos de viento + 8 de nieve + varias de sismo) puede llegar a cientos de "steps" (ej. 1/162) y tardar bastante más que el análisis estático solo.

**Si solo hace falta ver desplazamientos/deriva con las secciones actuales (no reoptimizar), no hace falta tildar Steel Design** — con Static Analysis sola alcanza, mucho más rápido, porque se salta Optimization + Design of members enteros.

**Para acelerar cuando sí hace falta optimizar**:
- Global Settings (Steel Design) → **Configurations to Calculate**: destildar lo que no haga falta en esa corrida (ver nota abajo sobre qué opciones aparecen realmente).
- **Método de diseño**: Enumeration (calcula cada combinación individual de la RC — exacto pero lento, sobre todo con cross-sections tipo "General" que corren SHAPE-THIN por cada combinación) vs. **Envelope** (usa solo el sobre máx/mín de la RC — mucho más rápido, algo más conservador) vs. Mixed (híbrido con umbral configurable).
- Design Situations → destildar "To Design" en las que no hagan falta para esa corrida puntual.

**Corrección (31/08, confirmado con captura real del proyecto)**: para el estándar **AISC 360 | 2016** (el que usa este proyecto), "Configurations to Calculate" solo lista **Strength** (obligatorio, no se puede destildar), **Serviceability** y **Seismic** — NO hay opción de "Fire Resistance" en este estándar/estas condiciones (puede ser propio de otras normas, o no aparecer si no hay cargas de incendio definidas en el modelo). No asumir que existe sin comprobarlo en el diálogo real.

Fuentes: FAQ Dlubal ["Optimization of Long Calculation Times for STEEL EC3 and ALUMINUM"](https://www.dlubal.com/en/support-and-learning/support/faq/002290), manual online ["Global Settings | Steel Design"](https://www.dlubal.com/en/downloads-and-information/documents/online-manuals/rfem-6-steel-design/000266).

## Steel Design — método "Envelope" puede dar secciones sobredimensionadas sin motivo físico

Al configurar el método de diseño en Global Settings (Steel Design), **NO usar "Envelope" para tratar de acelerar el cálculo si importa la precisión del dimensionado** (ej. precálculo de cotización, donde una sección más grande de lo necesario distorsiona el costo real):

- **Enumeration**: calcula cada combinación (CO) de la Result Combination una por una — cada chequeo usa esfuerzos (N, Vy, Vz, My, Mz, T) que ocurrieron juntos en una combinación real. Preciso, más lento.
- **Envelope**: arma primero el sobre máx/mín de cada componente por separado, y diseña con eso — **puede combinar esfuerzos que nunca ocurren simultáneamente en la realidad**, dando resultados conservadores (Dlubal documenta un caso propio con ratio 131% vs. la combinación real — [KB 001589](https://www.dlubal.com/en/support-and-learning/support/knowledge-base/001589)). No es inseguro (conservador = del lado seguro), pero sí puede sobredimensionar sin que corresponda a una combinación físicamente posible.
- **Mixed**: usa Enumeration hasta un umbral de variantes configurable, recién arriba de ese umbral cae a Envelope.

Para acelerar el cálculo sin perder precisión: dejar el método en **Enumeration** y en cambio destildar en Global Settings → Configurations to Calculate lo que no haga falta (ver arriba qué opciones aparecen realmente para el estándar de este proyecto) — eso no tiene costo de precisión.

Fuente: [Global Settings | Steel Design](https://www.dlubal.com/en/downloads-and-information/documents/online-manuals/rfem-6-steel-design/000266), [Comparing Stability Analysis... with Enveloping Result Combination (KB 001589)](https://www.dlubal.com/en/support-and-learning/support/knowledge-base/001589).

## Steel Design — reducir alcance del cálculo sin perder precisión: "Storing Results" y "Member Representatives"

Dos ajustes que sí reducen tiempo de cálculo/optimización de Steel Design **sin el costo de precisión de Envelope** (ver gotcha anterior):

**1. Global Settings (Steel Design) → Storing Results → Members**: dropdown con dos opciones (confirmado con captura real del proyecto, estándar AISC 360|2016):
- **By location** (default): guarda el ratio de diseño máximo en cada punto x a lo largo de cada miembro — más detalle, archivo de resultados más pesado.
- **By object**: guarda solo el ratio de diseño máximo del miembro entero. RFEM lo etiqueta directamente en el tooltip como *"The calculation process is fast"*. No cambia qué combinaciones se chequean ni el método de diseño — solo el nivel de detalle que se guarda. La vista gráfica de resultados no se pierde: si hace falta el detalle por ubicación en algún miembro puntual, RFEM lo recalcula al vuelo bajo demanda.
- Fuente: manual "Design Check Details \| Results" y "Global Settings \| Steel Design" (rfem-6-steel-design/000266, /000286).

**2. Member (Set) Representatives — el equivalente real a "agrupar miembros repetidos" (no existe un feature separado llamado "Member Design Groups")**: para estructuras con muchos miembros estructuralmente idénticos (mismo material+sección+tipo+longitud), RFEM puede diseñar **solo el representante con los esfuerzos más desfavorables de cada grupo de miembros idénticos**, en vez de todos uno por uno — reduce el alcance real de la Optimization, no solo el almacenamiento. **Validado en este proyecto (31/08): 708 miembros → 41 representantes.**

- Activar en **Model → Base Data → pestaña "Settings & Options"** → sección "Options" → tildar **"Member Representatives"** (NO "Member Set Representatives", salvo que además tengas objetos diseñados como Member Set — no es el caso acá).
- Se abre la pestaña **"Member Representatives Wizard"** → lista **"Consideration of Member Properties"**. Además de los criterios base obligatorios (Materials/Sections/Line Types/Member Types/Lengths, siempre tildados), **tildar también**: Rotations, Member Orientation | Position, End Modifications, Hinges, Eccentricities, Member Supports, Definable Stiffnesses, Nonlinearities. Dejar sin tildar: Transverse Stiffeners, Member Openings, Result Intermediate Points, Comments (irrelevantes para perfiles de catálogo estándar).
  - **Por qué tildar esos 8 y no confiar en el default**: el mecanismo agrupa y diseña UN SOLO representante por grupo — si dos miembros comparten sección+longitud pero difieren en alguna de estas propiedades (ej. una correa con distinta condición de rótula tras el fix del 28/08, una diagonal Truss "Failure if compression" vs. un miembro lineal, un tensor/apoyo puntual intermedio en una correa sí y en otra no), agruparlos sin ese criterio puede dejar al "hermano" del grupo sin verificar en su condición real — no es solo una cuestión de precisión, es un riesgo de subdiseño silencioso.
  - Los sub-ítems de **Steel Design Properties** y **Concrete Design Properties** (con flecha ">") vienen todos tildados por defecto al expandirlos — incluye **Effective Lengths** y **Design Supports**, los más críticos para pandeo. No hace falta tocarlos.
- En **Steel Design → Objects to Design**: la tabla lista una fila por tipo de objeto (Members, Member Sets, Member Representatives, Member Set Representatives). Vaciar **Members** (destildar "Design All", borrar "Selected Objects") y tildar **"Design All"** en **Member Representatives**.
- **Columna "Not Valid / Deactivated"**: los representantes que agrupan miembros no diseñables por Steel Design (en este proyecto, columnas/vigas de H°A°) aparecen ahí automáticamente excluidos — comportamiento esperado, confirmado en este proyecto (representantes 1-4,18,32,35 = columnas/vigas de hormigón). Verificar con click derecho → "Go To"/"Show" sobre cada uno antes de dar por buena la corrida, no asumir que es benigno sin mirarlo — si apareciera ahí un representante de un miembro de acero, sería señal de que algún criterio de agrupamiento quedó mal.
- Fuentes: ["Representatives" (Steel Design)](https://www.dlubal.com/en/downloads-and-information/documents/online-manuals/rfem-6-steel-design/000114), ["Member (Set) Representatives in RFEM 6" (KB 001726)](https://www.dlubal.com/en/support-and-learning/support/knowledge-base/001726), ["Member Representative Wizard"](https://www.dlubal.com/en/downloads-and-information/documents/online-manuals/rfem-6/000373).

## Steel Design — la Optimization puede NO actualizar automáticamente la sección del modelo (verificar antes de exportar/tomar resultados)

Dlubal documenta explícitamente, para el flujo manual de optimización (click derecho sobre la celda de sección en la tabla de Cross-Sections → "Optimize"): *"the effects of the modified cross-section on stiffnesses and internal forces in the model are not taken into account"* — hace falta un paso separado ("transfer optimized cross-sections to RFEM/RSTAB", disponible en el menú contextual) y **recalcular de nuevo** para que el modelo realmente quede con la sección optimizada.

**No confirmado todavía si esto también aplica al loop automático dentro de "Calculate All"** (el que corre como partial step "Optimization" con "step X/Y", iterando solo) — no encontré una fuente que lo aclare explícitamente para ese caso puntual.

**Antes de exportar (IFC, DXF), generar la Parts List, o dar por buena cualquier verificación (deriva, P-Delta) con las secciones "finales"**: comparar la tabla **Cross-Sections** del modelo contra la tabla **Design Ratios** — si la sección base y la sección con la que se calculó el ratio no coinciden, falta aplicar la optimización antes de confiar en cualquier resultado o exportación posterior.

Fuente: [Cross-Section Optimization for Steel Design in RFEM 6 (FAQ 005130)](https://www.dlubal.com/en/support-and-learning/support/faq/005130), [Section Optimization in Steel Design Add-on (FAQ 005705)](https://www.dlubal.com/en/support-and-learning/support/faq/005705).

## Generador automático de combinaciones + Actions con componentes que deben sumarse (no alternativas) — receta completa (31/08)

Cuando una Action (ej. "S - Snow load") tiene asignadas varias Load Cases que en realidad son **componentes que deben combinarse entre sí según reglas específicas** (no alternativas independientes como sí lo son las 18 LC de viento), el generador automático de Load Combinations trata cada LC de la Action como una alternativa suelta por defecto — genera combinaciones incompletas (falta el término compañero obligatorio) en la gran mayoría de los casos. Detectado exportando las Load Combinations a Excel y comparando patrones con un script — invisible a simple vista en la UI, con cientos de combinaciones de por medio.

**Caso real de este proyecto**: Action A3 "Snow load" con LC23(balanceada nave)/LC24(S1)/LC25(S2)/LC26(arrastre sotavento)/LC27(arrastre barlovento)/LC28(deslizamiento)/LC29(balanceada alero) — de las cuales solo 7 combinaciones son físicamente válidas (ver `metodo-cirsoc104-nieve.md`, coherencia direccional), pero el generador producía ~260 de 350 combinaciones LRFD con el término compañero faltante (ej. nieve solo en la nave, nada en el alero).

**Receta que funcionó — combinar 3 mecanismos**:

1. **Action Type = "Differently"** (Actions → la Action en cuestión → pestaña Main): habilita una columna **"Group"** en la pestaña Assignment (aparece con valores `-` vacíos — hay que completarla a mano, no es obvio que ya está ahí). LC del mismo grupo = alternativas entre sí. LC de grupos distintos = se cruzan libremente (pero **también permite que un grupo entero quede ausente** — esto NO fuerza que todos los grupos participen siempre, ojo con asumir que sí).
2. **Load Case Relations → pestaña "Exclusive Load Cases"** (mismo diálogo "Relationship Between Load Cases" que ya se usaba para sismo): para bloquear pares que combinan direcciones de viento contrarias (ej. LC24 con LC27).
3. **Load Case Relations → pestaña "Inclusive Load Cases"**: columnas "Selected Load Cases" / "Combine only together with Load Cases" — fuerza que el LC seleccionado **solo** pueda aparecer en combinaciones donde también esté presente (al menos uno de) los LC listados en la segunda columna. **Confirmado empíricamente: varias filas para el MISMO "Selected Load Case" se exigen todas a la vez (AND entre filas)** — no se pisan entre sí, cada fila agrega una condición adicional que debe cumplirse. Esto no estaba claro en la documentación de Dlubal, hubo que probarlo generando y revisando el resultado real.

Con Differently+Group resolviendo la estructura general, Exclusive bloqueando los cruces de dirección inválidos, e Inclusive (con varias filas por LC) obligando a que cada componente tenga sus términos compañeros obligatorios, se llegó de **~260 combinaciones incompletas de 350** a **0 combinaciones fuera de las 8 físicamente válidas** (las 7 esperadas + 1 extra válida pero no crítica).

**Metodología para verificar que funcionó**: no confiar en inspección visual de la UI — exportar Load Combinations a Excel, parsear con script (agrupar por el conjunto de LC de la categoría en cuestión, ignorando factor/orden) y comparar contra los patrones físicamente válidos esperados. Se necesitaron 4 iteraciones de prueba-y-error hasta cerrar esto — cada mecanismo de RFEM (Differently, Exclusive, Inclusive) se comportó de forma coherente con lo documentado, pero **la combinación de los tres, y el comportamiento de "varias filas = AND"**, no estaba clara sin probarlo con el resultado real generado.

## Sections table — la fila que "parece" fija puede no ser la que usa el miembro real (31/08)

Al fijar manualmente una sección en Steel Design → Sections → "Use Other Section for Design" (ver gotcha de más arriba sobre correas/puntales), es fácil editar la fila EQUIVOCADA sin darse cuenta. Caso real: se fijó la fila "W 530x82.0" (cuyo nombre base YA coincidía con el valor fijado, por eso parecía correcto), pero el miembro real de las columnas (Member Representative) usaba una fila DISTINTA de la tabla — con sección base **W200x15.0**, mucho más chica — que nunca se tocó. Resultado: ratio de diseño absurdo (η=15,113), que en un primer vistazo pareció un problema de torsión (el chequeo que más fallaba era H3.3, torsión combinada) pero en **Design Check Details** se vio que el torsor real era casi nulo (T=-0,05 kNm) — la tensión normal se disparaba a valores imposibles (fn=-4689 N/mm², 13x el Fy) por usar una sección demasiado chica para los esfuerzos reales, no por un problema de torsión genuino.

**Cómo verificarlo**: abrir **Design Check Details** (doble click sobre la fila del resultado, o el ícono correspondiente en la tabla) — ahí se ve el nombre exacto de la sección que RFEM usó de verdad para ESE miembro puntual (columna "Section Properties"), que puede no coincidir con la fila que se cree haber editado. No confiar en que "la fila que edité es la que corresponde" sin este chequeo cruzado.

## Effective Lengths — sin asignar, RFEM salta el chequeo de estabilidad SIN avisar con un ratio alto (31/08)

Si no se asigna explícitamente un Effective Length (K-factor) a un miembro, RFEM no asume un default silencioso y sigue — tira el error **ER0069: "No effective lengths are assigned to Members No. ... The stability design could not be performed for the members"**, y el ratio de diseño que se ve en la tabla resumen (ej. "Max: 0,297") **excluye por completo el chequeo de pandeo/estabilidad** — puede parecer que la sección está sobrada cuando en realidad ni se evaluó la parte más exigente. Revisar siempre la tabla de **Errors & Warnings** después de correr Design, no solo el ratio máximo.

**Cómo asignarlo**: Navegador → **Types for Steel Design → New "Effective Length"**. Pestaña Main: tildar los 4 modos de pandeo (flexión eje mayor/menor, torsional, lateral-torsional), método Eigenvalue para Mcr, ejes principales. Pestaña **"Nodal Supports & Effective Lengths"**: define K por segmento (ky/u, kz/v, kt, k-LT) — default **1,0 en los cuatro**, y la condición de apoyo default en los extremos es "Fixed in z/v & y/u & torsion" (traslación y torsión restringidas, giro de flexión libre — la condición clásica articulada que corresponde a K=1,0 de Euler).

**Ojo con miembros partidos en varios tramos físicos** (ej. una columna modelada como 3 Members de 0,95m cada uno, arriostrada en un eje cada 0,95m pero no en el otro, que necesita el tramo completo sin arriostrar): el checkbox **"Intermediate nodes"** de esa misma pestaña solo tiene sentido si el Effective Length se asigna a un **Member Set** que agrupe los tramos como una sola entidad continua — asignado a **Members individuales** sueltos, cada uno de 0,95m tiene su propio Start/End y no hay forma de decirle a RFEM que el eje fuerte debe tratarse como un solo tramo de 2,85m. Confirmar si el objeto es Member o Member Set antes de intentar usar nodos intermedios — no da error, simplemente no hace lo que uno espera.

## Generar un PDF de reporte sin LaTeX/pip/LibreOffice — usar el Chromium de Playwright directo (31/08)

Si hace falta un PDF de una memoria de cálculo o similar y el entorno no tiene LaTeX, pip funcional, ni LibreOffice instalados: si Playwright está disponible en el entorno (por los MCP tools), ya trae un binario de Chromium/chrome-headless-shell cacheado en `~/.cache/ms-playwright/` — se puede invocar directo por línea de comandos con el flag `--print-to-pdf`, sin necesidad de instalar nada more ni depender de sudo:

```
"$HOME/.cache/ms-playwright/chromium_headless_shell-XXXX/chrome-headless-shell-linux64/chrome-headless-shell" \
  --headless --disable-gpu --no-sandbox \
  --print-to-pdf=salida.pdf --no-pdf-header-footer \
  "file:///ruta/al/archivo.html"
```

(el número de versión "XXXX" varía — buscarlo con `find ~/.cache/ms-playwright -iname "chrome-headless-shell"`). Mucho más simple que instalar puppeteer/reportlab/LaTeX, y no requiere red ni permisos elevados.
