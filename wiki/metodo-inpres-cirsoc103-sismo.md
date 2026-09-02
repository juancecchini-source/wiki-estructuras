---
name: metodo-inpres-cirsoc103-sismo
description: Metodología de carga sísmica según INPRES-CIRSOC 103 Parte I, incluye combinación de acciones (Art. 3.7), fórmula de Ev (Art. 3.5.2). Método dinámico modal-espectral evaluado y DESCARTADO para este proyecto (28/08) — se vuelve al método estático. Ver sección "Cierre del método dinámico" y Pendientes al pie.
status: ACTIVO
last_updated: 2026-09-02
---

# Carga sísmica — INPRES-CIRSOC 103 Parte I

Ver `normativa-vigente.md` — zonificación clásica de 5 zonas sigue vigente pese al mapa PSHA nuevo (no incorporado al reglamento todavía).

## Secuencia de parámetros

1. **Zona sísmica**: Anexo A del reglamento lista zona por departamento/localidad directamente — no hace falta interpolar. Consultar también www.inpres.gob.ar para cruzar.
2. **Método estático vs. dinámico**: admitido directo (sin necesidad de chequear regularidad de Tabla 2.5) si altura < 9m o ≤3 niveles. Para estructuras más altas, chequear Tabla 2.5. La regularidad (Tablas 2.3/2.4) igual importa después para la excentricidad accidental, aunque no bloquee la elección del método.
3. **Grupo/factor de riesgo γr**: Grupo Ao=1,5 (esencial/emergencia), Grupo A=1,3 (uso público >300m² y >100 personas — incluye estadios/gimnasios), Grupo B=1,0 (estándar), Grupo C=0,8.
4. **Clasificación del sitio**: SA-SF según velocidad de onda de corte/SPT. Sin estudio de suelo real, adoptar SD como hipótesis conservadora razonable — NUNCA reemplaza un estudio geotécnico real antes del cálculo definitivo.
5. **Ca, Cv** (Tabla 3.1, según zona+sitio) → T2=Cv/(2,5·Ca), T1=0,2×T2, T3 (Tabla 3.2), as (aceleración efectiva de zona).
6. **Período aproximado Ta** = Cr×H^x (Tabla 6.2). Para sistemas mixtos sin categoría específica en la tabla, usar "Otros sistemas estructurales" (Cr=0,0488, x=0,75).
7. **Coeficiente sísmico C**: si Ta≤T2 → ec.6.3 = 2,5×Ca×γr/R. Comparar SIEMPRE contra el piso mínimo (ec.6.5 para zonas 3-4, ec.6.6 para zonas 0-1-2: C≥0,11×Ca×γr) — no es una elección entre fórmulas, son dos condiciones independientes, tomar la mayor.
8. **R (factor de reducción)**: Tabla 5.1 por sistema estructural. Para precálculo, adoptar el nivel NO especial/convencional (más conservador, no compromete a detallado dúctil todavía). **Estructura mixta (5.1.1)**: tomar el R MÍNIMO entre los sistemas que actúan en paralelo (ej. pórtico arriostrado acero R=3 + pórtico hormigón ductilidad limitada R=3,5 → usar R=3).
9. **W (peso sísmico)**: NO estimar a mano si se puede evitar — modelar primero con viento+nieve+gravitatorias (que no dependen del sismo), dejar que el software dimensione secciones reales, y usar el peso propio real resultante. Sumar sobrecarga con factor f1 (Tabla 3.3, según probabilidad de ocupación — con tribunas/asambleas usar probabilidad intermedia f1=0,50, no la reducida) y nieve con f2=0,20 si el techo evacúa nieve.

## Simultaneidad con viento

NO se combinan viento y sismo (Art. 3.7.4, confirmado con el texto del reglamento) — son casos independientes a envolver, no sumar.

## Combinación de acciones (Art. 3.7, confirmado con el texto del reglamento)

- **[3.16]** 1,20D ± 1,00E + f1×L + f2×S
- **[3.17]** 0,9D ± 1,00E
- **[3.18]** E = EH + EV (definición — NO es la fórmula de magnitud de Ev, esa está en otra parte, pendiente de ubicar)
- **[3.19]** (estructura de acero, componentes sensibles a sobrerresistencia) 1,20D ± Ω0×EH + 0,5L + 0,2S
- **[3.20]** 0,9D ± Ω0×EH

### Ev (componente vertical) — confirmado, Art. 3.5.2 (verificado en el texto del reglamento, pág. 49/106)

**[3.10]** Ev = (Ca/2) × γr × D — mismo Ca (Tabla 3.1) y γr (grupo de riesgo) que ya se usan en el resto del cálculo, no hace falta ningún dato nuevo.

Como Ev es proporcional a D, al reemplazar E=EH+EV en [3.16]/[3.17] el término se pliega en el coeficiente de D (mismo formato que ASCE 7):
- [3.16] → (1,20 + Ca/2×γr)×D ± EH + f1×L + f2×S
- [3.17] → (0,9 − Ca/2×γr)×D ± EH (el signo del término Ev acompaña al de EH; usar la combinación de signos que da el caso más desfavorable en cada verificación)

### Art. 6.3 (voladizos/balcones/aleros) — confirmado (pág. 70/106)

Aparte del Ev general de 3.5.2, los componentes sensibles a vibración vertical (6.3.a: voladizos, balcones, aleros) se verifican con dos fuerzas verticales adicionales, que NO se superponen con Ev:
- **[6.15]** Fv = Ca × γr × Wi (hacia abajo)
- **[6.16]** Fvup = −Ca × Wi (hacia arriba — sin γr, signo negativo explícito en el reglamento)

**Wi NO es solo el peso propio del voladizo** — el reglamento define Wi de forma general en el Art. "Acción sísmica horizontal" (antes de 3.5, ec. **[3.15]**): **Wi = Di + f1×Li + f2×Si**, la misma estructura que el W global (usada también en 4.3/6.2: W=ΣWi), aplicada localmente al componente/punto i. Para el voladizo: Di = peso propio de chapa+vigas/reticulado que continúan en el vuelo; f2×Si = nieve tributaria del voladizo (NO se puede omitir); f1×Li = sobrecarga de uso tributaria (probablemente ≈0, misma lógica que el W global, pendiente confirmar si aplica Lr acá).

## W (peso sísmico) — estado actual

W = D + f1×L + f2×S. Para este proyecto puntual:
- **D**: resuelto (autopeso real LC1 + peso de cerramiento LC30, ya modelados).
- **f2×S**: **resuelto y armado en RFEM** (02/09). Se usaron los **casos balanceados directos** (LC23 nave + LC29 alero) al 0,20, **no** la envolvente RC1 que se había propuesto conceptualmente. Es la elección correcta y no una simplificación: para masa sísmica interesa la nieve **esperable** durante el sismo, no el máximo de acumulación/arrastre/deslizamiento, que es un caso de dimensionado y no un estado de carga probable.

### Las dos combinaciones de peso sísmico (medidas 02/09)

Son el insumo de todo lo demás — de acá salen W y V₀, así que quedan registradas acá y no hay que volver a medirlas:

| CO | Nombre en RFEM | Composición | Σ reacciones Z |
|---|---|---|---|
| **CO8** | `CC_W_sismico` | 1,05·LC1 + 1,05·LC30 + 0,20·LC23 + 0,20·LC29 | **1.787,18 kN** |
| **CO9** | `CC_D_sismico` | 1,05·LC1 + 1,05·LC30 | **1.494,13 kN** |

Desglose que se deduce de esos dos valores (no hace falta medirlo aparte):

| Componente | kN |
|---|---|
| LC1 — autopeso de la estructura (sin mayorar) | 1.226,02 |
| LC30 — cerramiento (sin mayorar) | 196,96 |
| LC23 + LC29 — nieve balanceada total (sin factor) | 1.465,25 |

**Estas mediciones corresponden al modelo con el cerramiento ya cambiado a panel y con las rótulas nuevas de correas y columnas de frontis ya aplicadas** (confirmado por Juan 02/09). Que el modelo haya cerrado equilibrio con esas rótulas confirma de paso que no quedó ningún mecanismo.

### Coeficiente sísmico C — derivación verificada contra el reglamento (02/09)

**C = 0,13**, calculado con **[6.3]** (no despejado de V₀/W). Texto del reglamento, Art. 6.2.2:

- **[6.3]** `C = 2,5 · Ca · γr / R` — para **T ≤ T2**
- **[6.4]** `C = Sa · γr / R` — para T > T2
- **[6.6]** `C = 0,11 · Ca · γr` — mínimo para zonas sísmicas 0, 1 y 2

Para este proyecto: **Ta = 0,225 s ≤ T2 = 0,6 s** → gobierna [6.3] (tramo de aceleración constante, no hace falta entrar al espectro).

`C = 2,5 × 0,12 × 1,3 / 3 = 0,13`

El mínimo de [6.6] da `0,11 × 0,12 × 1,3 = 0,017` — no gobierna, queda muy por debajo.

**No despejar C de V₀/W.** C es un coeficiente reglamentario que depende de Ca, γr, R y T — no de W. Despejarlo de un V₀ de otra época propaga cualquier error que tenga ese V₀ (fue exactamente lo que pasó el 02/09: dio 0,2148, un 65% de más).

### V₀ corregido de 364,83 a 232,3 kN — causa confirmada (02/09)

**El modelo tenía cargado 0,64 kN/m² de cerramiento en vez de 0,06** — un error de unidades de factor 10, confirmado por Juan. Todo lo calculado hasta el 01/09 corrió con ~1.000 kN de peso propio inexistente.

| | kN |
|---|---|
| Cerramiento erróneo: 1.876 m² × **0,64** | 1.200,6 |
| W con ese error = 1,05·(1.226,02 + 1.200,6) + 293,05 | **2.841,0** |
| V₀ que eso produce = 0,13 × 2.841,0 | **369,3** |
| V₀ que estaba documentado y cargado en el modelo | 364,83 |

La diferencia residual de 1,2% se explica porque el LC1 de entonces era ~1.193 kN contra los 1.226 de hoy (2,7%), compatible con los ajustes de sección hechos entre el 28/08 y el 01/09.

**Valor correcto:**

`V₀ = C × W = 0,13 × 1.787,18 = **232,3 kN**` — un **36% menos** que el que está cargado.

**Consecuencias, separadas por si importan o no:**

- **Hay que regenerar LC34 (E dir. X) y LC35 (E dir. Y)** con el V₀ nuevo. Hasta que eso pase, el modelo está 57% sobrecargado en sismo.
- **La distorsión horizontal mejora mucho** (corte −36%): compensa de sobra el +5,2% de masa del panel y la pérdida de rigidez de las rótulas nuevas. Deja de ser un pendiente en riesgo.
- **Las columnas casi no cambian**: están gobernadas por flexión de viento. El P de la columna del pórtico baja de −177,96 a ~−128 kN, pero la compresión pesa 0,03 sobre un η de 0,83.
- **Sí cambian correas y reticulado**, que son los elementos gobernados por gravitatorias — es donde puede haber sobredimensionado real, y por eso importa para una cotización.
- **La deriva ELS de 26,0 mm NO mejora**: es bajo DS2 con viento, y sacar peso propio no reduce el desplazamiento lateral por viento en un modelo geométricamente lineal. La causa sigue siendo la sección de H°A°.

**Lección de proceso**: el archivo de proyecto decía 0,06 kN/m² mientras el modelo tenía 0,64. La documentación y el modelo estaban divergidos, y el error sobrevivió porque **nadie chequeó nunca la carga total contra la superficie**. El chequeo que lo habría detectado en 30 segundos: `Σ reacciones Z del load case ÷ carga unitaria = superficie implícita`, y comparar contra la superficie real. Hacerlo cada vez que se carga o se cambia una Surface Load.

### Efecto del cambio de chapa a panel sobre W (02/09)

El cerramiento pasó de **chapa 0,6mm (0,06 kN/m²)** a **panel Maxiroof PUR 50mm (0,105 kN/m²)**. Superficie de cerramiento implícita: 196,96 / 0,105 = **1.876 m²**.

| | W (kN) |
|---|---|
| Con chapa (estado del 28/08) | 1.698,55 |
| **Con panel (estado actual)** | **1.787,18** |

**+5,2%**. El W anterior se reconstruyó escalando LC30 hacia atrás (1.876 m² × 0,06 = 112,55 kN), no se midió — vale mientras el autopeso de la estructura (LC1) no haya cambiado entre el 28/08 y hoy.

**T no sale del modelo**: se usa Ta de la fórmula empírica con el tope de [6.7]. Consecuencia práctica: mientras no cambie la geometría, **C queda fijo en 0,13 y V₀ escala lineal con W**.

**Decisión — el 1,05 sobre las cargas permanentes (confirmado por Juan 02/09)**: es un **mayorante del 5% sobre el peso propio** para cubrir lo que el modelo de barras no dibuja — chapas de nudo, bulones, soldadura. Aplicado consistente en CO8 y CO9. No es un factor reglamentario ni parte de la ecuación [3.15]: es un criterio de proyecto, y por eso queda escrito acá.

**El período T no sale del modelo** — se calcula con la fórmula empírica y el tope de [6.7]. Consecuencia práctica importante: **si cambia la masa pero no la geometría, C no cambia y V₀ escala lineal con W** (`V₀_nuevo = 0,2041 × W_nuevo`), sin necesidad de rehacer nada del capítulo. Solo hay que recalcular T si cambia la altura o la tipología.
- **Mampostería de cerramiento (0 a 3m) — no modelada, y está bien no modelarla para carga gravitatoria** (apoya en el suelo, no cuelga de la estructura). **Para W el criterio es otro y depende de la junta**: si existe junta de separación sísmica real entre la mampostería y las columnas, su masa inercial va a su propia fundación y **queda fuera de W**; si está en contacto, hay que incluirla **y** deja de ser solo un tema de masa (el relleno rigidiza el pórtico, atrae carga y puede generar columna corta). **Criterio adoptado (Juan, 02/09): se asume junta, con el mismo estatus que la junta de las gradas** — o sea, la mampostería queda **fuera de W**, condicionado a que la junta se confirme. Los dos pendientes (junta de mampostería y junta de tribunas) van juntos: si uno cae, hay que revisar los dos, porque sostienen la misma hipótesis de masas que no entran al sistema resistente. Nota adicional: aun con junta, la mampostería necesita vínculo lateral para su estabilidad fuera del plano, y esa fuerza de anclaje **sí** es una acción local sobre la estructura (misma lógica de Wi = Di + f1·Li + f2·Si aplicada al componente, ec. [3.15]).
- **f1×L**: probablemente **≈0 para este proyecto** — las tribunas están apoyadas en el suelo con fundación propia, independientes de columnas/vigas/pórticos (no le transmiten masa sísmica al sistema resistente que se está diseñando), y todo el edificio es a nivel de suelo (sin entrepisos apoyados en la estructura). **Condición para que esto sea válido**: debe existir junta de separación sísmica real entre tribuna y edificio (evitar golpeteo/pounding) — confirmar que está contemplada en el proyecto.

## Corte basal y distribución en altura (Art. 6.2, confirmado contra el texto del reglamento)

- **[6.1]** V₀ = C × W
- **[6.2]** W = ΣWi (i=1...n niveles)
- **[6.7]** T (a usar) ≤ Cu×Ta (Tabla 6.1, Cu según as — interpolable)
- **[6.11]** Fk = (Wk×hk×V₀) / Σ(Wi×hi) — distribución **lineal** en altura (a diferencia de ASCE 7, NO hay exponente k ni interpolación por período)
- **[6.12]/[6.13]** — caso especial (90% en niveles intermedios + 10% extra concentrado en el último nivel) SOLO si el período T sin el límite [6.7] supera **2×T2**. Chequear siempre este umbral antes de asumir que aplica [6.11] directa.

Sin entrepisos pero con dos alturas de techo (nave vs. alero-vestuarios), tratar como **n=2 niveles de masa** para [6.11], cada uno con su Wk y su altura representativa (centro de gravedad del techo respectivo — aproximación razonable: promedio eave-cumbrera). Wk de cada zona se obtiene igual que W global: sumar reacciones de `CC_W_sismico` filtrando solo los nodos de esa zona.

**Reparto de Fk entre los pórticos de una misma zona**: por **masa tributaria** (ancho de paño de cada pórtico / ancho total de la zona, con ajuste de medio paño en los pórticos de punta), NO por rigidez relativa. Es la distribución correcta para diafragma flexible (cada pórtico resiste su propia masa, sin redistribución vía diafragma) y tiene la ventaja práctica de que **no depende de las secciones** — no hay que rehacer el reparto cada vez que se ajustan perfiles, a diferencia de un reparto por rigidez (que sí lo exigiría, y fue parte de lo que hizo laborioso el estático la primera vez).

## Método dinámico (Cap. 7, Procedimiento Modal Espectral) — evaluado y DESCARTADO para este proyecto (ver "Cierre del método dinámico" más abajo). Queda documentado como referencia para un proyecto futuro con diafragma rígido/semirrígido real.

Decisión tomada 27/08: migrar del estático al dinámico

Motivo: el reparto manual del método estático (por zona nave/alero, por altura, por pórtico, por dirección) se volvió muy laborioso y se rehace por completo cada vez que cambian las secciones. Con RF-DYNAM Pro (add-ons "Modal Analysis" + "Response Spectrum Analysis", ya activados en Edit Model→Add-ons) la masa se deriva de las combinaciones de carga y se recalcula sola.

**El cálculo estático NO se descarta** — el reglamento lo exige como piso de comparación:

- **[7.2.3]** Modos a considerar: hasta que la masa acumulada sea ≥90% en cada dirección analizada.
- **[7.1]** Cm = Sam × γr / R — ordenada espectral por modo (misma lógica de C que ya usamos, aplicada modo por modo).
- **[7.2.4]** Superposición modal: **CQC**. Si los períodos de los modos a superponer difieren >10% entre sí, se admite SRSS.
- **[7.2.5] Solicitaciones mínimas**: si el corte basal dinámico (Voe) resulta **menor al 85%** del corte basal estático V₀ (según 6.2 — ya calculado, V₀=364,83 kN), escalar las solicitaciones dinámicas por 0,85×V₀/Voe.
- **[7.2.6]** Torsión accidental en el método dinámico: se modela corriendo el centro de masa la distancia ec_ak (misma Tabla 6.3), no como momento torsor aparte.
- **[7.1.2]**: diafragmas NO rígidos (nuestro caso, flexible) requieren grados de libertad adicionales para los movimientos relativos entre masas — se resuelve solo al usar "masas desde combinación de carga" (reparte masa nodo por nodo según la carga real, no 2 masas lumped por zona).

### Setup en RFEM

1. Add-ons activos: Modal Analysis + Response Spectrum Analysis (Edit Model→Add-ons).
2. Load Case LC32, tipo **Modal Analysis** → masas importadas desde `CC_W_sismico` (1,05×LC1+1,05×LC30+0,20×LC23+0,20×LC29) vía "Import masses only from load case/load combination".
3. Load Case LC33, tipo **Response Spectrum Analysis** → importa modal desde LC32. Espectro **RS1 "According to Standard - INPRES-CIRSOC 103 | 2013-07"** (sí está soportado nativamente, confirmado — normas activadas en Edit Model→Base Data→Standards I, sección "Design | Standard Group" → "Dynamic analysis"). Asignado solo a direcciones **X e Y** (NO a Z — la vertical se resuelve aparte con Ev, ec. 3.10, no por espectro dinámico, ver Art. 7.1.1).
4. Parámetros de RS1 que el default de RFEM trae MAL y hay que corregir a mano: **Seismic group** (default A0 → cambiar a **A**), **Spectral type** (default 1/SA-SC → cambiar a **2**/SD), **Overall reduction factor R** (default 1,5 → cambiar a **3,0**, no se autocalcula). Al corregir Group y Type, Ca/Cv/γr se recalculan solos (verificado: dan 0,120/0,180/1,300, coinciden exactamente con lo calculado a mano). Na=1, Nv=1,2, ζ=5%, fa=1 vienen bien por default, no tocar.
5. SPS1 (Spectral Analysis Settings de LC33): combination rule **CQC** (default trae SRSS, cambiar). **Damping for CQC Rule: D=0,05 (5%) — quedó en 0,000 por default, PENDIENTE confirmar que se corrigió.**

## Cierre del método dinámico (28/08) — descartado para este proyecto

Se migró del método estático al dinámico el 27/08 (ver sección de arriba) porque el reparto manual se volvió muy laborioso. En el camino aparecieron varios problemas de modelado en RFEM que había que resolver antes de poder correr el modal — quedaron documentados como gotchas generales en `rfem-gotchas.md` (sección "Análisis Modal/Dinámico") porque no son específicos de sismo: tipo Cable vs. Truss en arriostramientos, rótulas de correas generando nodos singulares, Mass Matrix Settings, nodo corrupto con ID fuera de rango, diafragmas rígidos para participación de masa.

**Bloqueador**: la participación de masa modal nunca llegó cerca del 90% exigido (ec. 7.2.3). Se probaron dos configuraciones:

1. **Sin diafragma** (solo pórticos + correas + puntales reales conectando pórticos entre sí): con 15-50 modos se estancó en ~35%(X)/42%(Y). Se repitió el 28/08 con 80 modos (método "Automatic, to reach effective modal mass factors", techo 200 modos) y dio prácticamente el mismo resultado: **34,87%(X) / 42,02%(Y) / 11,31%(rotZ)** — plateau real, no un problema de cantidad de modos.
2. **Con diafragma rígido** (2 Rigid Coupling tipo Diaphragm — Special Objects: uno en los nodos de apoyo de cabreada de la nave, eave 5,85m, plano horizontal; otro en el alero, plano inclinado 4,7m→3,47m): con 10 modos la participación bajó a 14,33%(X) / **0,00%(Y)** — peor que sin diafragma. Diagnóstico: los diafragmas reordenaron los modos y el que cargaba masa en Y quedó fuera de los primeros 10.

**Por qué se descarta y no se sigue insistiendo con más modos**: el Rigid Coupling tipo Diaphragm de RFEM está pensado para elementos de alta rigidez en el plano (losas de hormigón, entrepisos) — no para chapa sobre reticulado sin topping, que la literatura idealiza como diafragma **flexible** (AISC, "Impact of Diaphragm Behavior on the Seismic Design of Low-Rise Steel Buildings", 2008: sistemas de steel deck sin topping con pórticos arriostrados se idealizan como diafragma flexible). Forzar un diafragma rígido acopla artificialmente pórticos que en la realidad casi no se ayudan entre sí — no es una corrección del modelo, es alejarlo del comportamiento real. Y sin diafragma, con solo el acople puntual de los puntales reales, la masa sísmica queda repartida en un número enorme de modos locales casi idénticos entre sí (cada pórtico vibrando casi independiente) — juntar el 90% exigiría un orden de magnitud más de modos que los probados, sin viabilidad de tiempo de cálculo ni ganancia real de información de diseño frente al estático.

**Decisión**: se descarta el método dinámico modal-espectral para este proyecto. Se vuelve al **método estático** (Cap. 6), admitido directo por el reglamento para esta altura (<9m, sin necesidad de chequear regularidad de Tabla 2.5) y más representativo del comportamiento real de un diafragma flexible con pórticos casi independientes (reparto por área de influencia, sin redistribución artificial entre pórticos). La sección "Método dinámico" de arriba queda como referencia para un proyecto futuro con diafragma rígido/semirrígido real (losa, entrepiso).

## Distorsión horizontal de piso (Art. 6.4.2) — verificado en X (28/08), PASA

Pórtico interior (nodos 66/68/69/71/74, Y=-24), de aislado (LC34, sin mezclar con gravedad — importante: de es el desplazamiento del caso sísmico puro, NO de una combinación con D+S, que arrastra un desplazamiento lateral propio grande por la asimetría nave+alero en voladizo).

de,68=13,5mm → du=Cd·de/γr=3×13,5/1,3=31,15mm → θsk=du/hsk=31,15/5850=**0,00532** — límite Tabla 6.4 Grupo A Condición D = 0,01. **Pasa con margen ~1,9x.**

LC36 (dir. −X) verificado como exactamente el negativo de LC34 en ambos nodos — confirma que no hay error de carga en esa dirección.

**Y — verificado (28/08), PASA con menos margen que X.** A diferencia de X (pórtico continuo, un solo tramo), en Y el sistema cambia de tipo a los 3m (H°A° abajo, arriostramiento de acero arriba) — se chequea por tramo, en el pórtico de punta (Y=-43,2, donde está el arriostramiento real):

- Tramo 1 (0→3m, H°A°): θsk=0,00146 (derecha) / 0,00169 (izquierda).
- Tramo 2 (3→5,85m, acero): θsk=0,00713 (derecha) / **0,00761 (izquierda, gobierna)**.

Límite 0,01 (Grupo A, Cond. D) — pasa con margen ~1,3x (más ajustado que el 1,9x de X; revisar de nuevo si cambian secciones de diagonales).

**Hallazgo aparte (no es falla de deriva, es informativo)**: el nodo de cumbrera del mismo pórtico (114) da de=46,7mm en Y (du amplificado ≈107,7mm) — mucho más que los eaves (~11mm), porque está a mitad de tramo sin arriostramiento directo. No es un punto que entre en el chequeo de Art. 6.4 (compara una misma ubicación en planta entre alturas, no dos puntos a la misma altura), pero confirma otra vez el comportamiento de diafragma flexible — dato a tener en cuenta para la conexión de correa de cumbrera o si se evalúa arriostramiento de techo adicional a mitad de largo.

**Pendiente**: el pórtico de punta en X (probablemente no gobierna dado el margen amplio del interior, pero no confirmado formalmente).

## Efecto P-Delta (Art. 8.4.4) — verificado (28/08), NO APLICA

CE=(Pk·de·γr)/(Vk·hsk) — Cd se cancela algebraicamente (aparece en du y de nuevo en el denominador), no hace falta para este cálculo. Pórtico de punta, dirección Y (el caso más exigido de deriva):

- Vk=52,279 kN — reacción de apoyo en Y de todo el pórtico aislado (columnas+arriostramiento), bajo LC35 puro. Mucho mayor que la carga local aplicada ahí (20,27 kN) — confirma que este pórtico recibe carga transferida desde los interiores vía correas/puntales, coherente con todo lo demás.
- Pk≈156 kN (tributaria nave+alero de este pórtico, estimada).
- Tramo 2 (acero, de=8,8mm, hsk=2850mm): **CE=0,0120**.
- Tramo 1 (H°A°, de=1,9mm, hsk=3000mm): CE≈0,0025.

Ambos muy por debajo del umbral 0,10 — **P-Delta no aplica a este proyecto**, ni con margen de error grande en la estimación de Pk.

## Pendientes por cerrar (no dar por completo el cálculo sin esto)

- **Arriostramiento anti-torsión del pórtico (nodo correa-pórtico) — era un error de modelado, no un gap real (corregido 28/08)**: se había liberado los 3 giros de las correas (My+Mz+torsión) asumiendo que la conexión real (clip/ménsula) no transmite ningún momento. Se investigó en fuentes (SCI, AISC/steelconstruction.info, AISI Design Guide, papers de rigidez rotacional correa-chapa) y la conexión real SÍ aporta restricción torsional — es el mecanismo por el que la correa arriostra el ala comprimida del cabio contra pandeo lateral-torsional. Liberar las 3 le sacaba esa restricción al modelo, generando matriz singular en el nodo del pórtico (nodo 581, y análogo en 131) — no era falta de arriostramiento real. Se había puesto un apoyo puntual restringiendo solo φy como parche numérico; **fix correcto**: liberar SOLO el momento de flexión de eje mayor de la correa (My en este proyecto — confirmado con la orientación del eje local), dejar Mz y torsión rígidos. El apoyo puntual parche queda innecesario una vez aplicado esto — sacarlo para no duplicar restricciones.

- **Junta de separación sísmica tribuna-edificio**: confirmar que está prevista en el proyecto (condición para que f1×L≈0 sea válido).
- **Torsión accidental (Tabla 6.3) — NO APLICA, confirmado por texto del reglamento (28/08)**: Art. 8.2.1.2 (Diafragma totalmente flexible) es explícito: *"cada uno de los elementos verticales se diseñará para las acciones correspondientes a su área de influencia y **no se considerarán torsiones**"*. Con diafragma flexible confirmado (ver punto siguiente), la torsión accidental queda formalmente eximida — no hace falta Tabla 6.3 para este proyecto.
- **Rigidez de diafragma (8.2.1) — criterio del reglamento (28/08)**: Art. 8.2.1.2 define diafragma totalmente flexible si "la máxima deflexión horizontal propia excede el doble del promedio de los desplazamientos relativos (del nivel) de los dos elementos verticales que menos se desplazan" — verificación formal con deflexiones todavía pendiente, pero hay evidencia indirecta fuerte a favor: la participación de masa modal estancada incluso a 80-90 modos (chapa sobre reticulado sin diafragma real) y la literatura (AISC 2008, steel deck sin topping con pórticos arriostrados = diafragma flexible por tipología). Mismo artículo habilita la exención de torsión de arriba.
- **Metodología de reparto validada contra el reglamento (28/08)**: Art. 6.2.4.1 confirma que Fk se aplica "en el baricentro de la carga gravitatoria Wk ubicada en el nivel k" (exactamente el hk-CG que sacamos de RFEM). Art. 6.2.4 remite el reparto entre elementos resistentes al Capítulo 8, no a una fórmula manual del Cap. 6. Art. 8.2.1.2 exige reparto por área de influencia para diafragma flexible (lo que ya veníamos haciendo, per pórtico). Art. 8.1.1 confirma que el análisis elástico lineal estándar (dejar que RFEM reparta internamente entre correas/arriostramiento/pórticos de H°A° vía su rigidez real) es el método aceptado — no hace falta un Master Node/Rigid Link para forzar el reparto a mano, sería agregar una rigidez artificial no prevista por el reglamento ni por la estructura real.
- **Fundaciones (Cap.9)**: arriostramiento de bases — pendiente para etapa de detallado.
- **Dirección -Y no se modeló como combinación separada — decisión deliberada, no un gap (confirmado y CERRADO 31/08)**: solo existen LC34(+X)/LC36(-X)/LC35(+Y) — no hay una LC ni una combinación para -Y, a propósito. Los tensores/diagonales de arriostramiento son los mismos en todo el techo/paredes (mismo Member Representative, ver `rfem-gotchas.md`), así que el máximo esfuerzo que salga de +Y ya cubre a su "espejo" bajo -Y, porque terminan con la misma sección. **Confirmado por Juan (31/08): el arriostramiento está en los DOS extremos de la nave (simétrico)** — con eso, tanto el tensor como el chequeo de vuelco de bases [3.17] quedan cubiertos con +Y solo, sin necesidad de -Y explícito. No queda nada pendiente en este punto.
- **Bug de tipeo encontrado y CORREGIDO (31/08)**: CO11 (3.16 -X) y CO12 (3.16 +Y) tenían el término sísmico (LC36 y LC35 respectivamente) con factor 0,2 en vez de 1,0 — subestimaba el sismo 5x en esas dos combinaciones puntuales. Confirmado contra el texto real del reglamento (línea 2381 del PDF): *"1,20 D ± 1,00 E ± f1 L ± f2 S [3.16]"* — f2 multiplica solo a S (nieve), E siempre lleva coeficiente 1,00. Corregido en RFEM y verificado en el Excel exportado post-fix: CO10/CO11/CO12 ahora con factor 1,0 en el término sísmico, correctas las tres.

## RFEM

RFEM SÍ tiene INPRES-CIRSOC 103 soportado nativamente (a diferencia de viento/nieve) — vía add-on **RF-DYNAM Pro** (Modal Analysis + Response Spectrum Analysis). Chequear que el add-on esté tildado en Edit Model→Add-ons (puede aparecer con punto verde "Purchased" pero destildado igual — hay que activarlo ahí antes de usarlo). Para el método estático no hace falta ese add-on — se puede aplicar como carga estática equivalente distribuida en altura, sin modelo de masa detallado.
