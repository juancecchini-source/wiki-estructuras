---
name: metodo-cirsoc102-viento
description: Metodología para determinar y aplicar carga de viento según CIRSOC 102-25, con RFEM como herramienta.
status: ESTABLE
last_updated: 2026-09-01
---

# Carga de viento — CIRSOC 102-25

Ver `normativa-vigente.md` para confirmar que sigue siendo la edición vigente.

## Norma en RFEM

RFEM no tiene "CIRSOC 102" nativo. Usar **ASCE 7-10** como base (es la norma sobre la que está construida la 102-25). Validar el qh resultante contra un cálculo manual con las tablas del CIRSOC antes de confiar en el software — diferencias >5% entre ediciones ASCE (7-10 vs 7-16 vs 7-22) casi siempre indican un dato de entrada mal cargado, no una diferencia editorial real. El generador de combinaciones puede forzarte a otra edición (ej. 2016) sin problema: el factor de carga de viento en combos es 1,0 en todas las ediciones desde 7-10 en adelante, así que no hay inconsistencia real.

## Secuencia de parámetros (en este orden)

1. **Categoría de Riesgo**: uso de "reunión pública" con ocupación >300 personas → III. Define qué mapa de velocidad usar (I=300 años, II=700, III/IV=1700).
2. **V (velocidad básica)**: Fig. 1.5-1D del reglamento trae tabla directa por ciudad principal. Usar la ciudad más cercana como referencia si el proyecto no está en una ciudad tabulada.
3. **Categoría de Exposición**: C es el default (terreno abierto, obstrucciones dispersas <10m). D solo con agua/planicie sin obstrucciones a barlovento >1500m.
4. **Kd** = 0,85 (edificios convencionales).
5. **GCpi** (clasificación de cerramiento): comparar SUMA de aberturas por pared (no cada una por separado) contra los umbrales:
   - Cerrado: Ao ≤ 0,4m² o 1%Ag (el menor) → GCpi=±0,18
   - Abierto: cada pared ≥80% abierta
   - Parcialmente cerrado: Ao > 1,10×Aoi (suma del resto de la envolvente) → GCpi=±0,55
   - Parcialmente abierto (catch-all, ni cerrado ni parc.cerrado ni abierto) → GCpi=±0,18 (¡mismo valor que cerrado!)
6. **G** = 0,85 para estructuras rígidas (frecuencia ≥1Hz — casi siempre aplica en naves bajas).
7. **Ke** (factor de altitud): reduce levemente qz a mayor altitud. Efecto chico hasta ~1500msnm.
8. **Kzt** (efecto topográfico, solo si hay loma/escarpa/colina cerca): Kzt=(1+K1·K2·K3)². K1 según forma de terreno+H/Lh, K2 según distancia x/Lh (usar tabla de µ: barlovento µ=1,5 siempre; sotavento µ=1,5 loma/colina, µ=4 escarpa), K3 según altura de evaluación z/Lh. Tabla tabulada disponible con K1,K2,K3 directos por interpolación — más simple que la fórmula cerrada.

## Voladizos de cubierta (Art. 5.11.4 / equivalente 2.4.4)

Cp=0,8 actuando en la cara INFERIOR del voladizo a barlovento, **sumado** (no reemplazo) a la presión ya calculada para la cara superior en esa misma zona. Solo aplica en la dirección de viento donde ese lado del voladizo queda a barlovento — en Case 3 (diagonal), sumar al 75% igual que el resto de esa combinación.

## Techos escalonados (techo bajo adosado a pared más alta)

ASCE 7 tiene provisión específica (Fig. stepped roofs, C&C) — la presión en la intersección con el muro es en realidad MENOR que en el resto del techo bajo, no mayor. Si RFEM no diferencia esa zona y aplica presión uniforme, es conservador (a favor de la seguridad), no un error a corregir.

## Modelado en RFEM — wizard correcto

- **"Vertical Walls with Roof"** (no "Duopitch/Flat roof" aislado — ese tipo es solo para cubiertas exentas sin pared, tipo marquesina).
- Nodos base I,J,K,L: en la base REAL (piso, z=0) de las paredes — nunca en una altura intermedia. Deben quedar alineados verticalmente bajo los nodos de techo A-F correspondientes.
- Con voladizo de cubierta: usar los nodos superiores de PARED (no el borde exterior del techo volado) al definir la geometría.
- **Edificio escalonado (dos alturas de nave distintas, adosadas)**: el wizard de un solo bloque NO lo resuelve. Correr 2 corridas separadas (una por volumen), excluyendo manualmente de "Loaded" la pared compartida entre ambos volúmenes (no recibe viento directo, queda tapada).
- Tabla "Generated on Members/Surfaces/Lines No.": si "Loaded" (miembros) queda vacío, NO se genera carga sobre elementos — se queda como bloque de superficie sin repartir. Hay que seleccionar ahí explícitamente las correas/columnas que deben recibir la carga.
- "Display separately" (clic derecho sobre la carga generada): cambia la vista de bloque de superficie a carga real por elemento — usar para diagnosticar si la redistribución funcionó.
- Casilla "Lock for new objects": destildada mientras se sigue modelando (permite que la carga se extienda sola a miembros nuevos al regenerar). Tildarla recién cuando el modelo esté cerrado.
- "Consider member eccentricity" / "Consider section distribution": destildadas salvo excentricidades deliberadas o secciones ahusadas.

## Casos de carga (4 casos del reglamento)

- Case 1 (perpendicular a cada cara, 0°/90°): gobierna casi siempre el dimensionado de barras individuales.
- Case 3 (0° y 90° simultáneos al 75%): obligatorio (no exceptuable salvo geometría regular específica), gobierna columnas de esquina y arriostramiento/anclajes.
- Case 2/4 (con torsión agregada): exceptuables en edificios de geometría regular — RFEM puede omitirlos solo si detecta que aplica la excepción.
- Generar ambos signos de Cpi (+ y −) en cada caso — la mitad "negativa" no se genera sola, hay que pedirla.

## Correas y chapa

- Correas: modelar como barras (transferencia de carga real + arriostramiento lateral contra pandeo de cordón/dintel). Simplemente apoyadas para precálculo (más simple, resultado conservador, evita cargas parciales del Cap.5 de nieve). Continuas = más eficiente en material pero exige más cálculo — decisión para etapa de cálculo definitivo.
- Chapa: solo como carga (peso propio + generación de superficie para wizards), nunca como elemento estructural resistente, salvo diseño explícito de diafragma de chapa (stressed-skin), que es método especializado aparte.
- Correa apoyada sobre ala de perfil: para precálculo, modelar coincidiendo con el eje del dintel es la práctica estándar (no un atajo cuestionable). La excentricidad real se puede agregar después con la función "member eccentricity" de RFEM cuando el detalle de conexión esté definido.

## Direcciones 0°/90° vs. 180°/270° — no siempre hace falta el caso "espejado"

El **Envelope Procedure** de ASCE 7 (Cap. 28, el que usa el Wind Load Wizard) está construido para que, en edificios razonablemente simétricos respecto a un eje, **no haga falta generar el caso de dirección opuesta (180°/270°) por separado** — el efecto de dirección ya queda contemplado dentro del propio patrón de coeficientes de cada caso (Case 1/2/3/4, con W1/W2 y ±GCpi). Confirmado el 31/08 en una consulta puntual: no es una limitación del wizard, es el procedimiento en sí (a diferencia del Directional Procedure, donde sí se verifican direcciones por separado).

**Condición para que esto sea válido**: que el edificio sea razonablemente simétrico respecto al eje en cuestión (ej. para viento paralelo a cumbrera, que los dos frentes/cabeceras sean parecidos entre sí). Si hay una asimetría real (una cabecera con portón grande y la otra sin abertura, por ejemplo), esta simplificación no se puede dar por válida sin revisarla.

**Ojo con esto al leer un LC**: antes de concluir que una distribución de presiones está mal, verificar que el LC que estás mirando sea realmente el caso que su nombre dice (ver sección "Nomenclatura de direcciones" abajo — en este proyecto los nombres estaban invertidos y eso disparó una investigación entera).

**Paredes laterales (paralelas a la dirección del viento) — succión en las DOS, no presiones opuestas**: es un error común esperar que si una pared lateral tiene succión, la del otro lado tenga presión (sentido contrario) — no es así. Windward: presión (Cp≈+0,8). Leeward Y ambas paredes laterales: succión (Cp≈-0,2 a -0,7 según la zona), **mismo signo en las dos paredes laterales**, no antisimétrico — el flujo se separa en las esquinas de barlovento y "tira hacia afuera" de las dos caras paralelas al viento por igual. Con geometría escalonada real (ej. un alero pegado de un solo lado, como en este proyecto), la succión real puede diferir en *magnitud* entre ambas paredes laterales por interferencia aerodinámica — pero el wizard (pensado para prismas rectangulares simples) no captura esa diferencia; es una simplificación aceptada para precálculo, no un error del método.

## Nomenclatura de direcciones — "Direction N" del wizard vs. paralelo/perpendicular a la cumbrera (01/09)

**El error más caro de la sesión del 01/09**: los Load Cases de viento de este proyecto estaban nombrados **al revés** ("paralelo a la cumbrera" era en realidad el caso perpendicular y viceversa), y sobrevivió sin detectarse desde que se crearon los LC. Disparó una investigación larga sobre un supuesto bug del wizard que no existía: lo que estaba mal era el nombre, no la carga.

**Cómo leer correctamente una fila de "Set Wind Perpendicular to"**: la tabla se llama literalmente *"Wind **Perpendicular to**"*, así que "Direction N" = viento **perpendicular a la pared/roof side listada en esa fila** — esa pared es la de barlovento en ese caso. No es "viento paralelo a esa pared".

**Traducción a terminología de cumbrera (techo a dos aguas)**:

| Roof side de la fila | Qué pared es | Viento de ese caso | Nombre correcto | ASCE 7 |
|---|---|---|---|---|
| Incluye el nodo de cumbrera (ej. `B-C-D`) | Cabecera / gable end | Viaja **a lo largo** de la cumbrera | **Paralelo a la cumbrera** | Longitudinal Direction |
| Sin nodo de cumbrera (ej. `A-B`, `D-E`) | Pared larga / de alero | Viaja **atravesando** la cumbrera | **Perpendicular a la cumbrera** | Transverse Direction |

Regla práctica para no equivocarse: **la pared de cabecera es la única cuyo "roof side" tiene 3 nodos** (sube hasta la cumbrera pasando por el nodo del vértice); las paredes largas tienen 2. Si el viento le pega de frente a la de 3 nodos, es el caso **paralelo** a la cumbrera.

**Verificación sin ambigüedad, con datos y no a ojo**: exportar la tabla de Nodos y mirar los dos nodos de cumbrera. Si comparten X y difieren en Y, la cumbrera corre en el eje Y → viento en Y = paralelo; viento en X = perpendicular. Usar la columna **"Global Coordinates" (I/J/K)**, NO la de "Coordinates" (F/G/H): para nodos tipo "On Member" o con sistema de referencia propio, F/G/H son coordenadas **locales** y pueden dar una geometría falsa (en este proyecto parecían indicar un nodo espejado al otro lado del edificio, que no era real).

## Volúmenes adosados — organizar los Load Cases por dirección REAL, no por el número interno de cada wizard

Con varios wizards (uno por volumen, ver "Edificio escalonado" arriba), **el "Direction 1" de un wizard no necesariamente apunta al mismo sentido real que el "Direction 1" de otro** — depende del orden de nodos con que se definió cada bloque. Si se asume que coinciden y se mandan al mismo Load Case, se termina con un LC que suma viento en +X sobre un volumen y viento en −X sobre el otro: un evento que no existe, y donde los efectos se compensan en vez de mostrar el caso crítico.

**Metodología correcta**:
1. Para cada wizard y cada dirección habilitada, identificar la pared windward y su **normal saliente** (con las coordenadas reales de sus nodos).
2. Traducir esa normal a un sentido global (+X, −X, +Y, −Y). Una pared es windward cuando el viento sopla **en sentido opuesto a su normal saliente**.
3. Agrupar por sentido global: **un evento real de viento = un Load Case**, alimentado por todos los wizards que aportan carga en ese evento.
4. Recién ahí asignar los LC destino en la pestaña "Load Cases" de cada wizard.

**El wizard no deja tildar dos direcciones opuestas (0° y 180°) en la misma entrada** — asume el Envelope Procedure donde un sentido por eje alcanza (válido solo en edificios simétricos). Con un volumen adosado esa simetría se rompe y hacen falta los dos sentidos: la solución es **duplicar la entrada del wizard** (íconos del panel "List") y dejar una entrada por sentido. En la copia, destildar en su pestaña "Load Cases" las filas de las direcciones que ya genera la entrada original, o esas cargas se duplican.

**Qué recibe y qué no cada volumen**:
- La **cara compartida** entre los dos volúmenes se destilda de "Set Loaded Wall/Roof" **en los dos wizards** (no recibe viento directo en ninguna dirección).
- El volumen que queda **a la sombra** del otro igual tiene que evaluarse en ese sentido: hay que **habilitar esa dirección en su wizard** para que existan sus succiones de sotavento. Destildar la dirección no "lo protege", simplemente hace que ese caso no exista y se pierdan sus succiones.
- La **franja de pared que sobresale** por encima del volumen adosado sí está expuesta y hay que cargarla. No se puede resolver dejando la pared entera tildada en "Loaded" y excluyendo los miembros interiores (ver `rfem-gotchas.md`: esa exclusión **redistribuye** la carga en vez de descartarla, inflando lo que reciben los miembros restantes). Se carga a mano con **Free Rectangular Load** sobre la franja real, con el valor de presión que el propio wizard le había asignado a esa pared, y al 75% en los Load Cases de Case 3.

## Combinaciones con viento — la categoría "Wind" ya es alternativa por defecto

A diferencia de sismo (donde varias Load Cases bajo una Action se suman como simultáneas si no se arma una relación a mano), **para viento no hace falta definir ninguna relación de exclusividad**. El manual de Dlubal lo dice explícitamente en un recuadro Tip:

> *"Load cases of the 'Wind' action category are generally applied as alternatively acting. You do not need to define any mutually exclusive criteria for them."*

El mecanismo es el **Action Type = "Alternatively"** de la Action de viento: *"Only one of the load cases of the action can be effective in the combination. This is the case, for example, with wind from different directions."* Verificado en este proyecto con 31 LC de viento: 0 combinaciones con dos vientos simultáneos.

Lo único que hay que controlar al agregar LC de viento nuevos: que estén **asignados a esa Action** (pestaña Assignment) y con categoría W — si quedan afuera, no reciben el tratamiento alternativo ni entran a las combinaciones.

**Coherencia direccional viento ↔ nieve: NO hace falta forzarla.** Es correcto que un LC de viento en un sentido se combine con arrastre o nieve desbalanceada formados por viento del sentido contrario — ASCE 7 y CIRSOC 104 tratan las configuraciones de nieve como patrones independientes del viento concurrente (el arrastre es residuo de eventos previos, no del viento que actúa en esa combinación). La coherencia que sí importa es la **interna de la nieve** (que la desbalanceada vaya con el arrastre del lado correcto), ver `metodo-cirsoc104-nieve.md`.

Fuentes: [Load Case Relations | RFEM 6](https://www.dlubal.com/en/downloads-and-information/documents/online-manuals/rfem-6/002930), [Actions | RFEM 6](https://www.dlubal.com/en/downloads-and-information/documents/online-manuals/rfem-6/000251), [Wind Loads | Load Wizards | RFEM 6](https://www.dlubal.com/en/downloads-and-information/documents/online-manuals/rfem-6/000265).
