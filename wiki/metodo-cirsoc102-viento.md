---
name: metodo-cirsoc102-viento
description: Metodología para determinar y aplicar carga de viento según CIRSOC 102-25, con RFEM como herramienta.
status: ESTABLE
last_updated: 2026-08-20
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
