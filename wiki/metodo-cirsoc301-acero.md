---
name: metodo-cirsoc301-acero
description: Metodología de diseño de estructuras de acero según CIRSOC 301-18, base AISC 360-2010, configuración correcta del add-on Steel Design de RFEM (Strength/Serviceability/Seismic) para no aplicar provisiones de EE.UU. que no corresponden.
status: ACTIVO
last_updated: 2026-08-31
---

# Estructuras de acero — CIRSOC 301-18 (base AISC 360-2010) y su uso en RFEM Steel Design

CIRSOC 301-18 dice explícitamente en su introducción que adoptó como base **"ANSI/AISC 360-2010"** (verificado en el texto del reglamento, `Reglamento-CIRSOC-301-18.pdf`, línea ~89) — mismo patrón que viento/nieve (`metodo-cirsoc102-viento.md`, `metodo-cirsoc104-nieve.md`): RFEM no tiene un módulo CIRSOC nativo, así que se usa el módulo **AISC 360** de Steel Design como base de cálculo.

**Ojo con la edición**: RFEM 6 solo ofrece el módulo **"AISC 360 | 2016"** — CIRSOC 301-18 se basa en la edición **2010**. Puede haber diferencias menores de fórmula entre ediciones (ej. Capítulo E de estabilidad tuvo revisiones entre 2010 y 2016) — no verificado en detalle artículo por artículo, tratar como simplificación aceptable para un precálculo, igual que la brecha de edición ya documentada para ASCE 7 en viento.

## Global Settings | Steel Design | AISC 360 — qué tocar y qué no

### Strength Configuration (pestañas Main/Stability) — usar tal cual, no requiere ajuste por CIRSOC

Los parámetros ahí (umbrales η para simplificar la interacción de Cap. H, "Position of Positive Transverse Load Application" para pandeo lateral-torsional, factores de Cap. B/F/G) son metodología de cálculo interna de AISC 360 — CIRSOC 301 adopta esos mismos capítulos, así que los defaults de RFEM son válidos sin necesidad de tocarlos.

### Serviceability Configuration — los defaults de RFEM son MÁS ESTRICTOS que CIRSOC 301 (no inseguro, pero sobredimensiona)

RFEM trae por default (parecen valores IBC/EE.UU.) algo como **Beam limits L/360, Cantilever limits Lc/180** — no corresponden a la Tabla L.3.1 del CIRSOC 301-18 (Cap. L, verificada en el texto real del reglamento):

| Elemento (CIRSOC 301-18, Tabla L.3.1 — "Para otros edificios") | Flecha total | Flecha por sobrecarga útil |
|---|---|---|
| Cubiertas y techos en general | **L/200** | **L/250** |
| Cubiertas y techos con carga frecuente de personas (no mantenimiento) | L/250 | L/300 |
| Pisos en general | L/250 | L/300 |
| Miembros que soportan elementos susceptibles de fisuración | L/300 | L/350 |
| Donde la deformación puede afectar el aspecto | L/250 | — |

Para **voladizos** (nota b de la tabla): **L = 2 × longitud del voladizo** — o sea, para la fila "cubiertas en general" el límite efectivo de un voladizo es ≈ **Lc/100** (total) o **Lc/125** (sobrecarga), mucho más permisivo que un Lc/180 tipo AISC/IBC.

**Cómo elegir el valor correcto en RFEM**: revisar contra qué Result/Load Combination está corriendo el chequeo de Serviceability (Design Situations) — si es una combinación de **carga total de servicio** (D+S+viento juntos), usar la columna "Flecha total" (L/200); si es solo el incremento de **sobrecarga variable**, usar "Flecha por sobrecarga útil" (L/250). Con el default L/360 el resultado es del lado seguro pero puede estar exigiendo secciones más grandes de lo que CIRSOC realmente pide — relevante para un precálculo de cotización, donde el objetivo es no sobrar acero de más.

**Desplazamiento lateral por viento**: la misma Tabla L.3.1 da valores propios (ej. HT/300 desplazamiento total del edificio, HP/400 o HP/300 entrepisos según si hay previsión de junta) — separados de la deriva sísmica.

**Desplazamiento lateral por sismo**: la Tabla L.3.1 remite explícitamente (nota d) al **Reglamento INPRES-CIRSOC 103, Parte IV** para las combinaciones con acciones sísmicas — NO usar los valores de esta tabla para deriva sísmica. Ya está resuelto correctamente en este proyecto vía Tabla 6.4 de INPRES-CIRSOC 103 (ver `metodo-inpres-cirsoc103-sismo.md`, sección de distorsión horizontal) — no confundir ambos chequeos ni mezclar límites de una tabla con combinaciones de la otra.

### Seismic Configuration — NO corresponde a CIRSOC, destildar

RFEM (módulo AISC 360) trae una pestaña **"Seismic Configuration"** con parámetros tipo **"Seismic force-resisting system: OMF (Ordinary Moment Frames)"** y distancia a rótula plástica (Sh) — esto es la categorización de **AISC 341** (provisiones sísmicas de EE.UU.: pórticos a momento ordinario/intermedio/especial, arriostrados concéntricos/excéntricos, con su propio detallado dúctil).

**Confirmado por búsqueda de texto completo sobre el CIRSOC 301-18 (15.833 líneas)**: no aparece ninguna mención a OMF/IMF/SMF ni a "pórtico a momento" como sistema sismorresistente — el diseño sísmico argentino se rige por **INPRES-CIRSOC 103** con su propia clasificación de sistemas (Tabla 5.1: pórtico arriostrado convencional R=3, etc.), un marco completamente distinto al de AISC 341.

Si "Seismic" queda tildado en **Configurations to Calculate** con esta config activa, RFEM aplicaría verificaciones de detallado dúctil de pórtico a momento AISC 341 a una estructura que en realidad es **arriostrada** (no a momento) y que no se rige por ese reglamento — resultado, en el mejor caso, irrelevante; en el peor, confuso o incorrecto.

**Recomendación**: destildar **"Seismic"** en Global Settings → Configurations to Calculate. Las combinaciones de acciones sísmicas de INPRES-CIRSOC 103 (Art. 3.7, ya armadas a mano en Load Combinations — ver `metodo-inpres-cirsoc103-sismo.md`) ya están incluidas en lo que **Strength** verifica — no hace falta la capa extra de AISC 341.

Fuente primaria: `Reglamento-CIRSOC-301-18.pdf` (Cap. B.3.7, Cap. L completo — Tabla L.3.1), texto completo consultado directamente, no por búsqueda web. Ubicación del PDF: ver `reference_reglamentos_pdf_location` en memoria.

## Verificación manual de una barra (Cap. E/F/H) — cuando conviene no esperar al Design de RFEM

Si RFEM está trabado (Effective Lengths sin resolver, cálculo lento, etc.) y hace falta avanzar en la elección de sección, se puede verificar a mano con lo que RFEM ya calculó de análisis estático (esfuerzos internos, que no dependen de la configuración de estabilidad) — método usado el 31/08 para la columna del pórtico Añelo, resultado consistente y rápido:

1. **Esfuerzos gobernantes**: sacar de RFEM (Results → Members - Internal Forces, extreme values, o Design Check Details si ya corrió aunque sea parcialmente) P, My, Mz, Vy, Vz de la combinación más desfavorable.
2. **Propiedades de sección reales**: no buscar en internet si hay un catálogo de fabricante disponible (ej. `Anexo/Folleto Perfiles W Gerdau (MAR 23).pdf` en este proyecto) — las búsquedas web para un perfil puntual dieron resultados contradictorios/mezclados con otras normas (ej. confundió una designación canadiense con una australiana). Extraer con `pdftotext -layout` y grep por el nombre del perfil, confirmando el orden de columnas contra el header real de la tabla, no asumiendo el orden de memoria.
3. **Compresión (Cap. E)**: con KL real (según arriostramiento físico real del eje, no lo que RFEM tenga configurado si está mal): Fe=π²E/(KL/r)², comparar KL/r contra 4,71√(E/Fy) para elegir pandeo elástico (E3-3) vs. inelástico (E3-2).
4. **Flexión (Cap. F)**: clasificar ala/alma (compacta si bf/2tf y d'/tw están debajo de los λp de Tabla B4.1b) → Mn=Mp=Fy×Zx si es compacta y Lb≤Lp=1,76×ry×√(E/Fy) (sin reducción por LTB).
5. **Interacción (Cap. H1)**: Pr/Pc<0,2 → H1-1b: Pr/(2Pc)+Mrx/Mcx+Mry/Mcy≤1. Pr/Pc≥0,2 → H1-1a: Pr/Pc+8/9×(Mrx/Mcx+Mry/Mcy)≤1.

Este método usa el mismo Cap. F/E/H que ya adoptó CIRSOC 301 de AISC 360 (ver arriba) — no es una norma distinta, es la misma cuenta que hace RFEM, hecha a mano como puente mientras se resuelve la configuración del software.
