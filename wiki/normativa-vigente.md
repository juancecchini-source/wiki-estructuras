---
name: normativa-vigente
description: Estado de vigencia de cada reglamento CIRSOC/INPRES-CIRSOC relevante. Verificar de nuevo si pasaron varios meses desde last_updated.
status: ACTIVO
last_updated: 2026-09-02
---

# Normativa vigente — última verificación 2026-08-21

| Reglamento | Edición vigente | Base | Notas |
|---|---|---|---|
| CIRSOC 101 (cargas permanentes/sobrecargas) | 2025 (101-25) | — | Aprobado junto con 102-25, 200-24, 201-25. Art. 4.8.1 (sobrecarga de mantenimiento de cubierta, pesada/liviana) verificado y documentado en `metodo-cirsoc101-sobrecargas.md`. Resto del reglamento sin leer completo todavía. |
| CIRSOC 102 (viento) | 2025 (102-25) | ASCE 7-10 (+ incorporaciones puntuales de 7-16 y 7-22) | Vigente desde 22/01/2026. Cambio de filosofía vs. 2005: mapas por Categoría de Riesgo, factor de carga de viento en combos = 1,0 (no 1,6). |
| CIRSOC 104 (nieve) | **2005** | ASCE 7-02 | Hay una edición 2026 en trámite, pero NO fue puesta en vigencia — la Resolución 11/2026 que activó 101/102/200/201 explícitamente no incluyó al 104. Sigue rigiendo 2005. Volver a chequear cada tanto. |
| INPRES-CIRSOC 103 Parte I (sismo, general) | 2018 (revisión 2013) | — | Hay noticias de una actualización de "Parte I, III y V" con vigencia declarada desde 01/06/2026, pero no encontramos evidencia de que haya cambiado el contenido técnico (zonificación, tablas). El mapa de peligrosidad probabilístico (PSHA) del INPRES es un producto científico nuevo, NO incorporado al reglamento — sigue rigiendo la zonificación clásica de 5 zonas. Verificar zona puntual en www.inpres.gob.ar si hay dudas. |
| INPRES-CIRSOC 103 Parte II (hormigón, detallado sísmico) | ~2021 (resolución MOP 112/23) | — | No revisada en profundidad todavía. |
| INPRES-CIRSOC 103 Parte IV (acero, detallado sísmico) | Primera edición (sin fecha exacta confirmada) | — | No revisada en profundidad todavía. |
| CIRSOC 301 (acero) | 2018 (301-18) | AISC 360-2010 | Remite a normas IRAM/IRAM-IAS para material, no directo a ASTM. |
| **CIRSOC 303** (conformados en frío, sección abierta) | 2009 | AISI S100 (método de anchos efectivos) | **El que rige correas, perfiles C/U/Z y todo lo conformado en frío** — el 301 lo remite explícitamente. No tiene tabla propia de deformaciones: su C-A.4.4 remite al Capítulo L del 301. Materiales: solo normas IRAM-IAS (A.2.1), ninguna ASTM. PDF con capa de texto: ver `reference-cirsoc303-online` en memoria. |

## Lineamiento — jerarquía de fuentes cuando el CIRSOC no cubre el caso

**Adoptado 02/09 (criterio de Juan). Aplica siempre, en cualquier proyecto.** Cuando el reglamento argentino no responde una pregunta de diseño, **no se pasa directo a suponer ni se acepta el default del software**: se sube por la cadena de la que ese CIRSOC deriva, en este orden.

1. **El CIRSOC específico del material o elemento.** Ojo con cuál es: para una correa de chapa conformada el que rige es el **303**, no el 301, aunque el 301 sea el reglamento de acero "general".
2. **El CIRSOC al que ese reglamento remite.** Los CIRSOC se remiten entre sí — el 303 manda al Capítulo L del 301 para deformaciones, el 301 manda al INPRES-CIRSOC 103 Parte IV para sismo. Seguir la cadena hasta agotarla.
3. **La norma extranjera en la que se basa ese CIRSOC, en la edición correspondiente.** Está en la tabla de arriba: 301-18 → **AISC 360-2010**; 303 → **AISI S100**; 102-25 → **ASCE 7-10**; 104-05 → ASCE 7-02. Usar la edición que corresponde, no la última publicada: los criterios cambian entre ediciones y el CIRSOC quedó anclado a una.
4. **Los documentos de apoyo de esa norma** — Design Guides de AISC, Commentary, Apéndices. Suelen ser justamente donde vive lo que el reglamento no fija.
5. **Recién ahí**, criterio propio o especificación del fabricante — anotándolo como decisión con su porqué.

**Por qué**: los CIRSOC son adaptaciones, y lo que no fijan explícitamente muchas veces sí está resuelto en la norma de origen o en su documentación de apoyo. Saltear ese paso lleva a adoptar el default del software (que responde a otra tradición normativa) o a inventar un criterio, y las dos cosas se pagan en acero de más o en una decisión que no se puede defender ante un tercero.

**Caso que originó el lineamiento (02/09)**: flecha admisible de una correa de pared. El 301 no la tiene y el 303 tampoco — remite al 301. Subiendo a AISC (**Design Guide 3**, documento de apoyo de AISC 360) apareció el criterio explícito y con fuente: **L/120 con viento de 10 años**. RFEM traía L/200 con viento pleno, y un criterio intermedio que habíamos adoptado por analogía (L/150) también resultó innecesariamente exigente. Ver `metodo-cirsoc301-acero.md`.

## Cómo re-verificar rápido

- Página oficial de reglamentos: buscar "Publicaciones tecnológicas CIRSOC" (INTI).
- Zona sísmica puntual: www.inpres.gob.ar, herramienta de geozonas.
- Herramienta de geozonas Dlubal (viento/nieve/sismo por coordenadas, cruza contra INPRES-CIRSOC 103): dlubal.com/es/zonas-de-cargas-para-nieve-viento-y-sismos.
