---
name: historial_resuelto
description: Changelog corto de cosas cerradas/superadas. Detalle técnico vive en los archivos de método, acá solo la línea de qué cambió y cuándo.
status: HISTORICO
last_updated: 2026-09-02
---

# Historial resuelto

- 2026-08-20 — Primer volcado de la wiki, destilado del chat de precálculo del polideportivo de Añelo (viento + nieve completos, sismo en curso).
- 2026-08-21 — Repo `wiki-ingenieria` creado y pusheado a GitHub. Sesión larga en Claude Code trabajando el modelo RFEM del polideportivo: nieve cerrada en RFEM (arrastre/deslizamiento/no-balanceada + 7 Load Combinations + RC1 envolvente), peso de chapa y Lr (CIRSOC 101-25, no el valor descartado de 0,30kN/m² de una propuesta no vigente) agregados a RC1, RC2 envolvente de viento armada (18 casos incl. succión de voladizo), y avance en sismo (combinaciones Art.3.7 confirmadas, tribunas independientes descartadas del cálculo de W).
- 2026-09-02 — Cerrada la discusión de cómo modelar la estabilidad de la columna del pórtico (Añelo). Investigada la doc de Dlubal: se descartó reunificar el miembro y se descartaron los Member Sets (Member y Member Set son excluyentes por diseño — el ER0032 del 01/09 era comportamiento esperado). Camino adoptado: longitudes de pandeo **absolutas** sobre los miembros partidos, con Mcr por "AISC Chapter F". Decidido además poner **tornapunta** en la columna del pórtico (o dejarlo como pendiente de obra), lo que vuelve real el Lb=0,95m y fija η=0,827. Hallazgos de RFEM en `rfem-gotchas.md` (incl. nodos tipo "On Member", que responden la duda de si la correa queda vinculada).
