---
name: index
description: Puerta de entrada obligatoria. Leer primero, saltar solo al archivo que hace falta.
status: ACTIVO
last_updated: 2026-09-02
---

# Índice — Wiki de Ingeniería (precálculos estructurales)

## Metodología (evergreen — sirve para cualquier proyecto futuro)

- `metodo-cirsoc102-viento.md` — ESTABLE. Metodología completa de carga de viento (CIRSOC 102-25 / ASCE 7-10), parámetros, voladizos, techos escalonados, modelado en RFEM.
- `metodo-cirsoc104-nieve.md` — ESTABLE. Metodología completa de carga de nieve (CIRSOC 104-2005), balanceada/no balanceada, arrastre (Cap.7), deslizamiento (Cap.9).
- `metodo-cirsoc101-sobrecargas.md` — ESTABLE. Sobrecarga de mantenimiento de cubierta (CIRSOC 101-25, Art. 4.8.1), cubiertas livianas/pesadas, fórmula Lr=0,45·R1·R2.
- `metodo-inpres-cirsoc103-sismo.md` — ACTIVO (en desarrollo). Metodología de carga sísmica, método estático, R para sistemas mixtos. Faltan puntos por cerrar — ver el archivo.
- `metodo-cirsoc301-acero.md` — ACTIVO. Diseño de acero (CIRSOC 301-18 / AISC 360-2010), configuración correcta de Steel Design en RFEM: límites de flecha reales (Tabla L.3.1) y por qué "Seismic Configuration (OMF)" no corresponde a INPRES-CIRSOC 103.
- `normativa-vigente.md` — ACTIVO. Estado de vigencia de cada reglamento CIRSOC, con fecha de última verificación, y **el lineamiento de jerarquía de fuentes** (qué hacer cuando el CIRSOC no cubre el caso). Revisar antes de arrancar un proyecto nuevo.
- `rfem-gotchas.md` — ACTIVO. Comportamientos no obvios del software (wizards, add-ons, visualización).
- `rfem-buenas-practicas.md` — ACTIVO. Prácticas recomendadas por Dlubal para cualquier proyecto: verificación de modelo (Model Check/Plausibility Check), **verificación de orientación/rotación de secciones**, tolerancias, equilibrio global, mallado FE, reducción de combinaciones, backups/versionado.
- `materiales-perfiles.md` — ESTABLE. Equivalencias W/A992 vs IPN-UPN/F24-F36.

## Proyectos (viven mientras dura el proyecto, después pasan a HISTORICO)

- `calculo-polideportivo-anelo.md` — ACTIVO. **Los números calculados de Añelo** (pesos adoptados, W, V₀, Fk, η). Buscar acá antes de volver a medir algo en RFEM.
- `proyecto-polideportivo-anelo.md` — ACTIVO. Polideportivo en Añelo, Neuquén — precálculo para cotización.

## Changelog

Ver `historial_resuelto.md` para hallazgos ya cerrados y superados (ej. correcciones de interpolaciones, ediciones de norma descartadas).
