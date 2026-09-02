---
name: rfem-buenas-practicas
description: Buenas prácticas generales de RFEM 6 (verificación de modelo, tolerancias, mallado, combinaciones, backups) — no atadas a un proyecto puntual, aplicar desde el arranque de cualquier modelo nuevo.
status: ACTIVO
last_updated: 2026-08-31
---

# RFEM — buenas prácticas generales

Complementa a `rfem-gotchas.md` (comportamientos no obvios/errores ya pisados) — este archivo son prácticas recomendadas por Dlubal para cualquier proyecto, a aplicar de entrada.

## Verificar el modelo ANTES de calcular

Dos herramientas distintas, ambas en el menú **Tools**, para correr antes de "Calculate All" — no son lo mismo:

- **Tools → Model Check**: detecta problemas geométricos.
  - **Identical Nodes**: agrupa nodos con coordenadas iguales (o casi iguales, según tolerancia) para fusionar — típico en modelos importados de CAD, donde una línea corta genera dos nodos casi coincidentes.
  - **Crossing or Not Connected Lines/Members**: encuentra miembros que se cruzan en el espacio pero NO comparten nodo en el cruce (ej. dos diagonales de arriostramiento que se cruzan sin nodo común). **Ojo**: conectar automáticamente ahí NO siempre es lo correcto — la propia documentación de Dlubal advierte *"Only connect members if internal forces and moments can be transferred as well at the crossings"* (dos diagonales de bracing que se cruzan sin gusset real, por ejemplo, NO deben conectarse — hacerlo introduciría continuidad estructural que no existe).
  - **Overlapping Lines/Members**: miembros superpuestos parcial o totalmente — al confirmar, RFEM los divide automáticamente en tramos.
  - Fuente: [Model Check | Calculation | RFEM 6](https://www.dlubal.com/en/downloads-and-information/documents/online-manuals/rfem-6/000437).

- **Tools → Plausibility Check**: detecta problemas de consistencia de datos de entrada (no geometría). Dos niveles: **"Normal"** (campos faltantes en una tabla) y **"With warnings"** (chequeo más detallado, incluye también nodos de coordenadas idénticas). Recomendado correrlo **antes de calcular**, no después. Fuente: [Plausibility Check | Calculation | RFEM 6](https://www.dlubal.com/en/downloads-and-information/documents/online-manuals/rfem-6/000436), [Tips & Tricks: Plausibility Check (KB 000897)](https://www.dlubal.com/en/support-and-learning/support/knowledge-base/000897).

## Verificar la ORIENTACIÓN (rotación de sección) de los elementos, no solo su posición

Ni **Model Check** ni **Plausibility Check** detectan una sección **girada 90°**: geométricamente el modelo es impecable — el miembro está donde va, conectado a los nodos correctos, con la sección correcta. Lo único que está mal es el **ángulo de rotación de la sección alrededor de su eje longitudinal**, y eso RFEM lo toma como un dato de entrada válido. Es un error silencioso: no da warning, calcula sin quejarse, y el resultado es numéricamente correcto para el modelo que se le describió.

**Por qué importa tanto en secciones abiertas**: en un perfil C conformado en frío la relación Iy/Iz es del orden de **7 a 1**. Girar la sección 90° no penaliza un 10 o 20% — divide la rigidez y la resistencia por **casi un orden de magnitud**. En un perfil W la relación es parecida o peor.

**Cómo verificarlo — tres chequeos, del más rápido al más completo:**

1. **Un η absurdo es un síntoma de modelado, no de sección chica.** Si un elemento secundario (correa, puntal, tensor) da η > 3, la explicación casi nunca es "falta sección": nadie elige una sección 5 veces más chica de lo necesario. Antes de agrandar el perfil, sospechar del modelo. Con η > 10 en ELS, directamente asumirlo.
2. **Cruzar el ratio de solicitaciones contra el ratio de inercias.** En el Design Check Details, comparar `Mz/My` de las *Design Internal Forces* contra `Iy/Iz` de las *Section Properties*. Si la carga dominante del elemento está flexionando el **eje débil**, hay que justificar por qué — en una correa de pared, un puntal o una viga, la carga principal debe ir al eje fuerte. Ejemplo real (Añelo, correa de pared C120/50/2,5): `Mz/My = 7,90/0,38 = 20,8` con `Iy/Iz = 6,85` — el viento, que es la carga que gobierna, estaba entrando enteramente por el eje débil.
3. **Mirarlo en pantalla**: activar la visualización del modelo con secciones renderizadas y los **ejes locales** de miembro (Display navigator → Model → Members → *Member Axis Systems* / *Cross-Sections*). Es el chequeo definitivo, y conviene hacerlo por familia de elementos (todas las correas de pared juntas, todas las de cubierta juntas), no miembro por miembro: **el error de rotación es sistemático** — si se generó una correa girada, están giradas todas las de esa familia, porque salen de la misma operación de copia/generación.

**Cuándo correr este chequeo**: junto con Model Check y Plausibility Check, antes de la primera corrida completa; y otra vez apenas aparece un η fuera de escala. Es barato y evita rehacer verificaciones completas.

**Corolario de la misma familia**: no dar por sentado qué eje local es el fuerte. En la columna del pórtico de Añelo, la flexión de eje débil ocurre alrededor del eje local **z** y la de eje mayor alrededor del **y** — pero eso depende de la rotación real de la sección y hay que verificarlo de nuevo en cada tipo de elemento (ver `proyecto-polideportivo-anelo.md`).

## Verificar Load Combinations ANTES de la primera corrida completa (no reactivo, siempre que haya combinatoria compuesta)

Cuando un proyecto tiene alguna Action con componentes que deben combinarse entre sí según reglas específicas (no alternativas simples como el viento) — típicamente nieve con arrastre/deslizamiento/zonas, pero puede aplicarse a cualquier caso similar — **no confiar en que el generador automático de RFEM las combinó bien solo porque no tiró error**. El generador no avisa cuando genera combinaciones incompletas (le falta un término compañero obligatorio) — es indistinguible de una combinación válida mirando la UI, y a la escala de cientos de combinaciones es inviable revisarlas una por una a simple vista.

**Paso a incorporar de entrada, no solo cuando algo se ve raro**:
1. Exportar **Load Combinations** (y **Result Combinations** si aplica) a Excel (Export desde la tabla).
2. Parsear con un script corto (agrupar cada combo por el conjunto de Load Cases de la categoría en cuestión, ignorando factor/orden) y comparar contra los patrones físicamente válidos esperados para ese tipo de carga (definidos en el método correspondiente, ej. `metodo-cirsoc104-nieve.md` para nieve).
3. Repetir después de cualquier regeneración de combinaciones (borrar+generar de nuevo invalida la verificación anterior).

Encontrado en el proyecto Añelo (31/08): esta verificación, hecha recién cuando Juan sospechó algo raro (no de entrada), encontró un bug de tipeo en 2 combinaciones sísmicas (factor 0,2 en vez de 1,0) y un bug sistemático en ~75% de las combinaciones auto-generadas de nieve (término compañero faltante) — ninguno de los dos visible en la UI de RFEM. Ver la receta completa del fix (Action Type Differently + Load Case Relations Inclusive/Exclusive) en `rfem-gotchas.md`.

## Tolerancias del modelo (Settings & Options → Model Tolerances)

- Default de RFEM: **0,5mm** de tolerancia para nodos/líneas/superficies/direcciones — por debajo de esa distancia, dos elementos se consideran el mismo punto.
- Si se supera esa tolerancia al mover/copiar/arrastrar geometría, RFEM **deja de fusionar automáticamente** nodos/líneas/aberturas cercanas — hay que revisar a mano.
- Al importar de CAD con la opción de minimizar tolerancias activada, RFEM las baja a **0,00001m** automáticamente — más estricto, para no perder precisión de geometría real de un DXF/IFC.
- Fuente: [Settings and Options | Model Base Data | RFEM 6](https://www.dlubal.com/en/downloads-and-information/documents/online-manuals/rfem-6/000369), [FAQ Model Tolerances](https://www.dlubal.com/en/support-and-learning/support/faq/005072).

## Primer chequeo de resultados, siempre: equilibrio global

Antes de mirar deformaciones o diseño de secciones, el primer chequeo de sanidad tras cualquier "Calculate All" es el **equilibrio global**: la suma de reacciones de apoyo tiene que ser igual y opuesta a la suma de cargas aplicadas (teóricamente cero). RFEM muestra esto en la tabla resumen de resultados ("equilibrium of forces in the structural system"). Si no cierra, hay un problema de vínculo/rigidez en el modelo antes que un problema de norma o de carga — no seguir analizando sin que este chequeo cierre. Fuente: [FAQ — Displaying Results of Support Reactions](https://www.dlubal.com/en/support-and-learning/support/faq/001193).

## Mallado FE en superficies (chapa de reparto, losas, etc.)

- Tamaño de malla objetivo por defecto: **50cm** — razonable para superficies "normales"; reducir para superficies chicas.
- Densidad mínima recomendada por Dlubal: **8 a 10 elementos finitos entre los bordes** de una superficie rectangular, nunca menos de 4. Con distribución de esfuerzo axial no constante, mínimo **5 elementos**.
- Malla más fina = más precisión pero más tiempo de cálculo (cada nodo nuevo de malla agrega ecuaciones a resolver) — es un trade-off, no "cuanto más fino mejor" sin costo.
- **Mesh Refinements**: permite densificar la malla solo en zonas de interés (ej. cerca de una conexión, zona de concentración de tensiones) sin encarecer el cálculo en el resto del modelo — o al revés, definir una malla más gruesa en superficies de reparto de carga que no importan por sí mismas (como las superficies "Load transfer" que ya usamos para chapa, ver `rfem-gotchas.md`), ya que ahí la precisión del mallado no aporta nada real.
- Fuente: [FE Mesh Refinements | RFEM 6](https://www.dlubal.com/en/downloads-and-information/documents/online-manuals/rfem-6/000476), [Mesh Settings | FE Mesh | Calculation | RFEM 6](https://www.dlubal.com/en/downloads-and-information/documents/online-manuals/rfem-6/000242).

## Reducir combinaciones de carga generadas (distinto del riesgo de "Envelope" en Steel Design)

En **Load Cases & Combinations → Combination Wizard**, la opción **"Reduce number of generated combinations"** reduce drásticamente el número de combinaciones a generar: clasifica cada load case variable en "aporta positivo" o "aporta negativo" al resultado, y arma las combinaciones solo con las relevantes para máximos/mínimos — Dlubal documenta un caso real de 105 combinaciones bajadas a 5.

**Esto NO es lo mismo que el método "Envelope" de Steel Design** (ver gotcha "Envelope puede dar secciones sobredimensionadas" en `rfem-gotchas.md`): acá se decide **qué load cases entran en cada combinación** (lógica combinatoria estándar, la misma que haría un ingeniero a mano para no generar combinaciones absurdas), no se mezclan componentes de esfuerzo de distintas combinaciones en un solo chequeo — no tiene el mismo riesgo de conservadurismo no físico. Es una reducción segura de aplicar de entrada, no algo a evitar por miedo a resultados irreales.

Fuente: [Reducing the Number of Load Combinations in RFEM 6 (KB 001763)](https://www.dlubal.com/en/support-and-learning/support/knowledge-base/001763).

## Backups y versionado del archivo

- RFEM crea un backup automático (`.rf6bak`) a intervalos regulares Y antes de cada cálculo — pero **se sobreescribe** apenas se abre otro modelo o se deja el mismo abierto más de un día. **No sirve como historial real**, solo como red de seguridad ante un cierre inesperado inmediato.
- En modelos grandes, el backup automático antes de cada cálculo puede introducir una demora notable — se puede desactivar en **Settings → Program Options → pestaña "Program"** si molesta.
- Para versionado real (poder volver a un estado anterior con intención, ej. "antes de migrar a método dinámico de sismo" o "antes de correr la primera optimización de secciones"): usar **Save as Version** — queda guardado dentro del mismo archivo, visible en la pestaña **History** de Model Base Data. Es la herramienta correcta para marcar checkpoints deliberados, a diferencia del backup automático que es solo una copia de emergencia de corto plazo.
- Fuente: [Automatic Backup File (FAQ 000639)](https://www.dlubal.com/en/support-and-learning/support/faq/000639), [Save as Version](https://www.dlubal.com/en/support-and-learning/support/product-features/002422).

## Entregables para el cliente / cotización: IFC, planos, BOM, informe

Antes de generar cualquier entregable con secciones "finales" (post-optimización), ver el gotcha *"la Optimization puede NO actualizar automáticamente la sección del modelo"* en `rfem-gotchas.md` — comparar Cross-Sections vs. Design Ratios antes de confiar en la exportación.

- **Exportar IFC** (File → Export → IFC): RFEM 6 exporta solo en **IFC4.0** (no 2x3, esa versión solo se puede importar). Elegir la vista **"Reference View" (Coordination View)** para compartir con otros programas CAD/BIM — incluye geometría + secciones + materiales. La alternativa **"Structural Analysis View"** está pensada para intercambio entre programas de cálculo estructural, no para coordinación con arquitectura/otras disciplinas.
- **Planos/vistas 2D**: exportar a **DXF/DWG** (File → Export → DXF/AutoCAD) da geometría real con líneas de cota, útil para que un tercero arme planos formales. Para algo más rápido, cualquier vista en pantalla se puede exportar directo como imagen.
- **Informe combinado (memoria técnica)**: **Printout Report** — armar una compilación con las vistas (planta, elevaciones, isométrica) + tablas de resultados/diseño que hagan falta, exportar todo junto a **PDF**. Se puede guardar como plantilla para no rearmar la selección en cada proyecto.
- **Bill of Materials / tonelaje** (el dato que más le importa a quien cotiza estructura metálica): **Results → Parts Lists** — agrupa automáticamente miembros con propiedades idénticas ("items") y da el **peso total por ítem**, exportable a Excel. Cruzar contra la pestaña **Statistics** de Cross-Sections (peso total por sección usada) como verificación rápida.

Fuentes: [IFC Export (Guideline, KB 001797)](https://www.dlubal.com/en/support-and-learning/support/knowledge-base/001797), [IFC | Export | RFEM 6](https://www.dlubal.com/en/downloads-and-information/documents/online-manuals/rfem-6-interfaces/004032), [DXF / AutoCAD | Export | RFEM 6](https://www.dlubal.com/en/downloads-and-information/documents/online-manuals/rfem-6-interfaces/004030), [Printout Reports | RFEM 6](https://www.dlubal.com/en/downloads-and-information/documents/online-manuals/rfem-6/000363), [Parts Lists | Results | RFEM 6](https://www.dlubal.com/en/downloads-and-information/documents/online-manuals/rfem-6/000485).

## Cálculo lento con uso de CPU bajo — revisar el Solver

Si el cálculo tarda mucho y el uso de CPU (Task Manager) se mantiene bajo (no saturando los núcleos disponibles), revisar **Static Analysis Settings → Solver**: si está en **Iterative**, probar cambiar a **Direct** (descomposición matricial tipo LU, resuelve en un paso — normalmente más rápido salvo que la RAM disponible sea una limitación real para modelos muy grandes). No cambiar en medio de un cálculo ya corriendo — dejarlo para la próxima corrida. Fuente: [Direct vs. Iterative Solver (FAQ 005477)](https://www.dlubal.com/en/support-and-learning/support/faq/005477).
