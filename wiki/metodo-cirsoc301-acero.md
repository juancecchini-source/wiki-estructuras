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

### Correas de PARED (flecha horizontal por viento) — la Tabla L.3.1 NO cubre este caso

Buscado en el texto completo del CIRSOC 301-18 y del CIRSOC 303 el 02/09. **No existe un límite reglamentario para la flecha horizontal de una correa de pared bajo viento.** Conviene saberlo antes de aceptar el número que ponga el software:

- Las filas de **"Deformaciones verticales"** de la Tabla L.3.1 son para miembros que soportan **cubiertas o pisos** — son flechas verticales por gravedad. Una correa de pared flectada horizontalmente por viento no entra.
- Las filas de **"Desplazamiento lateral"** son para **columnas y para el edificio** (H/150, HT/300, HP/400…), no para un miembro flexionado.
- **CIRSOC 303-2009 es el reglamento que rige estos perfiles** (el 301 lo remite explícitamente: *"Para el proyecto de elementos estructurales resistentes de chapa de acero doblada o conformada en frío de sección abierta… se aplicarán las especificaciones del Reglamento CIRSOC 303-2009"*). **No tiene tabla propia**: su Comentario C-A.4.4 dice que *"las especificaciones adoptadas son las del Capítulo L y el Apéndice L del Reglamento CIRSOC 301-2005"*. O sea remite de vuelta a la misma Tabla L.3.1, que tampoco cubre el caso.
- Los dos reglamentos son explícitos en que esto queda **abierto a criterio**: el 303 (C-A.4.4) dice que la especificación base *"no contiene requisitos específicos ni combinación de acciones aplicables dejando las mismas libradas al criterio del Proyectista o al acuerdo entre Proyectista y Comitente"*; el 301 (L.3) admite fijar límites por *"especificaciones particulares (por ejemplo revestimientos especialmente sensibles a fisuración o daño por deformación)"*.

**Consecuencia práctica**: el valor que RFEM aplique acá (L/200 por default) **no sale del reglamento** — es un default del software. Es una decisión de proyecto que hay que tomar y justificar, y no es menor: en un caso real (Añelo, correa de pared) mover el criterio entre L/120 y L/200 cambió el η de 1,04 a 1,73, o sea decidía sola qué perfil comprar.

**Criterio adoptado: L/120 con viento de 10 años (0,7W)** — de **AISC Design Guide 3, *Serviceability Design Considerations for Steel Buildings*, 2ª ed.**, que es documento de apoyo del AISC 360 en el que se basa el CIRSOC 301-18 (ver el lineamiento de jerarquía de fuentes en `normativa-vigente.md`). Texto exacto:

> *"For the design of girts and wind columns supporting metal wall panel systems a deflection limit of **span divided by 120 using ten-year wind loading** is recommended for both girts and wind columns. The wind loading should be based on either the 'component and cladding' values using ten-year winds or the 'component and cladding' values (using the code required 'basis wind speed') **multiplied by 0.7**, as allowed in footnote f in IBC 2003, Table 1604.3…"*

**Los dos números van juntos y no se pueden separar**: L/120 es admisible *porque* se verifica con el viento de 10 años, no con el de 50 años del diseño de resistencia. Tomar L/120 con W pleno sería demasiado permisivo, y L/200 con W pleno (el default de RFEM) es doblemente conservador — exige más flecha *y* la mide con más viento.

Si el fabricante del revestimiento fija un límite más exigente, ese manda (L.3 del 301 lo habilita expresamente).

> **Descartado**: por analogía dentro de la Tabla L.3.1 se puede llegar a L/150 (las filas "miembros soportando cubiertas flexibles" L/150 y "desplazamiento de columnas por viento" H/150 convergen ahí). Es defendible, pero **queda superado por el DG3**, que trata exactamente este elemento en vez de razonar por analogía, y que además fija con qué viento medirlo. Anotado para no volver a recorrer el mismo camino.

### Perfiles conformados en frío: el material y el método NO son los del 301

Tres cosas a chequear cada vez que RFEM verifique un perfil conformado en frío (correas, C, U, Z), porque los defaults del software no coinciden con el marco argentino:

**1. El acero tiene que ser una norma IRAM-IAS, no ASTM.** El artículo A.2.1 del CIRSOC 303 lista exclusivamente normas **IRAM-IAS** (U 500-42, 500-72, 500-131, 500-205-x, 500-206-x, 500-503…). No admite ASTM. Si RFEM tiene puesto **A572 Gr.50 "HR Structural Shapes and Bars"** hay dos errores encima: es una designación **ASTM**, y es de un producto **laminado en caliente** aplicado a un perfil **conformado en frío**.

**Y no es cosmético — el Fy cambia el resultado.** El acero de referencia del ejemplo oficial del CIRSOC 303 es **F24, Fy = 235 MPa**; el A572 Gr.50 que trae RFEM tiene **Fy = 344,7 MPa**. Son **47% de resistencia a flexión de diferencia**, y en el sentido inseguro. Confirmar con el proveedor qué chapa se usa realmente (F24 = 235; los galvanizados estructurales tipo ZAR llegan a 340) **antes** de cerrar cualquier sección.

*Lo que sí suele estar bien*: E = 200.000 MPa. El 303 lo fija explícitamente (*"a los efectos del cálculo, en el Reglamento se utiliza un valor de 200.000 MPa"*) y coincide con el E del A572 en RFEM (29.000 ksi ≈ 199.950). Ojo si alguien elige un acero europeo (S235/S355), que en RFEM viene con **210.000 MPa** — ahí las flechas salen ~5% menores de lo que el reglamento admite.

**2. El método de RFEM no es el del CIRSOC 303.** RFEM verifica por **AISI S100-16 / Direct Strength Method** (se lo reconoce en el Design Check Details por los "Applicability Limits Acc. to Tab. B4.1-1" y por la ausencia de anchos efectivos). El CIRSOC 303-2009 usa el **método de anchos efectivos** del AISI S100 clásico (Cap. B: `be`, λ, ρ). Dan resultados parecidos pero no idénticos, y **el que hay que poder mostrar en una memoria argentina es el del 303**. Para un precálculo de cotización el DSM sirve; para el cálculo definitivo, no.

**3. "Sección totalmente efectiva" NO es una propiedad del perfil solo.** Es la pregunta de si hay pandeo local, y depende del acero: el criterio del 303 es λ ≤ 0,673 con

> λ = √(f / Fcr) , Fcr = k·π²E / [12(1−μ²)] · (t/b)²   *(Ec. B.2.1-4 y B.2.1-5)*

`Fcr` es pura geometría, pero **`f` es la tensión actuante, que para resistencia vale Fy**. O sea λ escala con **√Fy**: un perfil totalmente efectivo con F24 puede dejar de serlo con un acero de mayor grado. **Por eso no existe —ni puede existir— un listado de perfiles "sin pandeo local" válido en general**: solo tiene sentido *para un acero dado*.

Lo que sí es puramente geométrico, y conviene mirar primero como filtro rápido, son los límites del Cap. B.1: **Tabla B.1-1** (máxima relación be/b según L/bf, del *shear lag* en alas anchas) y **B.1.2** (h/t ≤ 200 en almas no rigidizadas).

**Por qué vale la pena verificarlo igual**: si el perfil resulta totalmente efectivo, `Se = S` y la resistencia sale directo (`Ma = φb·S·Fy`, φb=0,90) sin iterar anchos efectivos — se puede comparar secciones a mano, en una planilla, sin depender del software. Y la verificación es barata: alcanza con escalar el λ publicado por √(Fy_nuevo/Fy_ejemplo).

### Con qué carga se verifica el ELS — no es la combinación de resistencia

CIRSOC 101-25 (C 1.3.2 y C 2.3) no da valores numéricos de flecha: fija el principio y remite. Su comentario dice que *"los estados límite de servicio y los factores de carga asociados se consideran en el Apéndice C de ASCE 7-2010"*, y el Comentario C-A.4.4 del CIRSOC 303 lo refuerza: *"Las cargas de servicio adecuadas para verificar los estados límites de servicio pueden ser apenas una fracción de las cargas nominales."*

Concretamente, para deflexiones bajo viento el Apéndice C de ASCE 7 usa **0,7W** (viento de recurrencia menor), no W pleno. **Verificar contra qué combinación está corriendo la Design Situation de Serviceability en RFEM**: si arrastra el viento sin reducir, la flecha sale ~30% sobreestimada y eso se paga en sección.

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
