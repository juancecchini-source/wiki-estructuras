---
name: optimizacion-costo-estructura
description: Palancas que bajan el costo de una nave metálica de forma significativa, con cuánto movieron la aguja en un caso real y qué las condiciona. Colección viva — agregar cada una que aparezca.
status: ACTIVO
last_updated: 2026-09-02
---

# Palancas para bajar el costo de una nave metálica

**Para qué es este archivo**: juntar las decisiones que bajan el tonelaje o el costo de montaje de forma **significativa** (no el 2%), para poder recorrerlas de entrada en cada proyecto nuevo en vez de redescubrirlas. Cada una con **cuánto movió la aguja en un caso real**, **qué la condiciona** y **cómo verificarla**.

**Cómo usarlo**: recorrer la lista al arrancar el precálculo, no al final. Varias de estas se vuelven caras de aplicar una vez que el modelo está armado y verificado (cambiar la separación de correas obliga a rehacer cargas de superficie, wizards de viento y verificaciones).

**Regla que atraviesa todo el archivo**: ninguna de estas es gratis — cada una tiene una condición que hay que confirmar con un tercero (fabricante, proveedor, comitente). **Anotar la condición junto con la palanca**, o en la próxima sesión se aplica sin confirmarla.

---

## 1. La separación de correas, antes que la sección de la correa

El momento de la correa escala **lineal con la separación**, pero la **cantidad** de correas escala con su **inversa** — y gana la cantidad. Separar más las correas obliga a un perfil mayor y aun así baja el peso total.

**Caso Añelo (correas de pared, luz 4,334 m, F-24)** — peso de correa por m² de pared, tomando en cada caso la opción más liviana que verifica:

| Separación | kg/m² de pared |
|---|---|
| 0,95 m (lo que estaba modelado) | 9,19 |
| 1,20 m | 8,11 |
| **1,40 m** | **6,95** — −24% |
| 1,80 m | 5,83 — −37% |

Y además baja la **cantidad de piezas** de todo tipo: correas, uniones, riostras, montaje.

**Qué la condiciona**: la **luz admisible del revestimiento entre apoyos**, que es dato del fabricante del panel. Es lo primero a pedir — **no tiene sentido afinar la sección de la correa mientras la separación esté sin decidir**.

**Por qué se pasa por alto**: la separación suele venir heredada del modelo inicial o de "lo que se usa siempre", y una vez que las cargas de superficie y los wizards de viento están armados sobre esa grilla, cambiarla cuesta rehacer todo. Decidirla **al principio**.

---

## 2. Contar piezas, no solo kilos

Una solución más liviana en acero puede ser más cara si multiplica las piezas a fabricar y montar. **El peso es visible en el modelo; las piezas no.**

**Caso Añelo (correas de pared)**: arriostrar el ala interior con ángulos permitía usar un perfil más chico y ahorraba ≈5,8 kg de acero por tramo. Contado sobre toda la obra: ≈150 m de pared → ~105 correas-tramo → **210 piezas chicas con montaje en altura, para ahorrar ≈0,6 t de acero**. No cierra. Ganó el perfil más grande sin riostras.

**Cómo verificarlo**: antes de adoptar una solución con elementos secundarios repetitivos, **multiplicar por la cantidad real de tramos del edificio**, no razonar por elemento. El ahorro por tramo engaña.

**Cuándo se da vuelta**: si el salto de sección disponible es grande (porque la serie comercial no tiene el escalón intermedio), la pieza extra vuelve a convenir. En el mismo caso, si en vez del C 200x80 hubiera que ir al 250x85, la diferencia pasaba de 5,8 a 16,7 kg por tramo y las riostras ganaban.

---

## 3. El criterio de flecha, cuando el reglamento no lo fija

Los defaults de ELS del software responden a otra tradición normativa y suelen ser **más exigentes que lo aplicable** — y en elementos que gobierna el ELS, eso decide solo qué perfil se compra.

**Caso Añelo (correa de pared)**: RFEM corría con **L/200 y viento pleno**. El criterio aplicable, de AISC Design Guide 3 para *girts* de paneles metálicos, es **L/120 con viento de 10 años (0,7W)**. Con el default el η daba **1,73**; con el criterio correcto, **0,73**. El ELS pasó de gobernar el dimensionado a no ser el problema.

**Ojo con la trampa**: L/120 y 0,7W **van juntos**. Tomar el límite permisivo con el viento pleno no es válido.

**Cómo verificarlo**: ver `metodo-cirsoc301-acero.md` (dónde vive cada criterio) y `normativa-vigente.md` (el lineamiento de jerarquía de fuentes: cuando el CIRSOC no lo fija, subir a la norma base y sus Design Guides antes de aceptar el default o inventar).

---

## 4. La calidad del acero puede ser más barata que los kilos

Subir de grado cuesta más por kilo pero puede evitar un salto de sección entero. **Solo se puede evaluar si el modelo tiene declarado el acero real**, no el default del software.

**Caso Añelo (correas)**: el modelo tenía A572 Gr.50 (Fy=344,7 MPa) sobre perfiles conformados en frío que acá se compran en **F-24 (Fy=235)**. Son **47% de resistencia de diferencia** — y no es un error conservador: el chequeo estaba dando por buena una sección que con el acero real no verificaba. Con Gr.50 alcanzaba el 2,5 mm; con F-24 hay que ir al 3,2 (+27%).

**Cómo verificarlo**: al arrancar, poner las calidades reales (ver `materiales-perfiles.md`: **W → F-36; correas, chapas y redondos → F-24**) y recién ahí dimensionar. Si una sección queda al límite, **preguntar al proveedor si consigue el grado superior antes de subir de perfil** — puede salir más barato.

---

## 5. Un η absurdo casi nunca es "falta sección"

Antes de agrandar un perfil por un η alto, sospechar del modelo. Un elemento secundario con η > 3 no está mal dimensionado: está mal modelado. Y el sobredimensionado que sale de un modelo con un error se paga entero en la cotización.

**Casos Añelo, los dos reales**:
- **Correas de pared dibujadas giradas 90°** → η = 4,93 (ELU) y 11,87 (ELS). Resistían el viento con la menor inercia.
- **Cerramiento cargado con 0,64 kN/m² en vez de 0,06** (error de unidades ×10) → ~1.000 kN de peso propio inexistente, que infló todo lo gobernado por gravitatorias (correas, reticulado) durante semanas.

**Cómo verificarlo**: ver la buena práctica de **verificar la orientación de los elementos** y el chequeo de **superficie implícita** (`Σ reacciones Z ÷ carga unitaria = superficie`, comparar contra la real) en `rfem-buenas-practicas.md`.

---

## 6. Arriostrar en vez de agrandar — cuando son pocas piezas

Es la palanca inversa de la #2: cuando el elemento es **uno por pórtico** y no cientos, una tornapunta puede evitar un salto de sección en una barra pesada.

**Caso Añelo (columna del pórtico, W530x92)**: la tornapunta a 1,90 m justifica **Lb = 0,95 m** en vez de la longitud completa. Sin ella el η subía de 0,827 a ~0,93 (margen de 17% → 7%) y quedaba al borde de tener que cambiar una sección de 92 kg/m.

**Por qué acá sí conviene y en las correas no**: son ~20 piezas contra ~210, sobre una barra mucho más pesada. **La misma idea gana o pierde según la cantidad** — que es exactamente el punto de la palanca #2.

---

## 7. Verificar dónde está el problema antes de gastar acero

Cuando algo no verifica, medir **de dónde viene** antes de reforzar. El elemento que no cumple no siempre es el que causa el problema.

**Caso Añelo (deriva lateral ELS)**: la deriva total daba 26,0 mm contra 19,5 admisibles. Medido nodo por nodo, **el 91% venía de las columnas de H°A° de planta baja** (sección 40x25 que vino del dibujo de los arquitectos); la columna metálica aportaba 2,39 mm sobre 2,85 m. Cambiar la sección de acero o arriostrar el pórtico metálico habría corregido el 9% del problema, con todo el costo del lado del acero — que además era el único lado en la cotización.

---

## Candidatas a evaluar (todavía sin cuantificar)

Anotar acá lo que suene prometedor pero no esté medido, para no perderlo:

- **Longitudes efectivas reales en vez del default K=1,0** — un chequeo con Lb equivocado sobredimensiona. Ver el pendiente de Effective Lengths en `proyecto-polideportivo-anelo.md`.
- **Método "Envelope" en Steel Design** — puede dar secciones sobredimensionadas por mezclar componentes de esfuerzo de combinaciones distintas. Ver `rfem-gotchas.md`.
- **Optimización automática de secciones de RFEM** — útil, pero no actualiza sola la sección del modelo. Ver `rfem-gotchas.md`.
- **Aprovechar que una carga no gobierna para no refinarla** — en Añelo la nieve balanceada superaba al Lr en todo punto, así que no hizo falta refinar R1/R2 por elemento (`metodo-cirsoc101-sobrecargas.md`). No baja acero, baja horas.
