---
name: rfem-gotchas
description: Comportamientos no obvios de RFEM 6 descubiertos en la práctica. Revisar antes de arrancar un modelo nuevo.
status: ACTIVO
last_updated: 2026-08-21
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
