---
name: rfem-gotchas
description: Comportamientos no obvios de RFEM 6 descubiertos en la práctica. Revisar antes de arrancar un modelo nuevo.
status: ACTIVO
last_updated: 2026-08-21
---

# RFEM — gotchas y comportamientos no obvios

## Wind Load Wizard

- Tipo **"Duopitch/Flat/Monopitch roof"** = solo cubiertas EXENTAS sin pared (marquesinas). Para un edificio real con paredes, usar **"Vertical Walls with Roof"**.
- Nodos base I,J,K,L: deben estar en el piso real (z=0) de las paredes, alineados verticalmente bajo los nodos de techo correspondientes (A bajo I, etc.). Nunca en una altura intermedia.
- Con voladizo de cubierta: usar nodo superior de PARED como referencia, no el borde exterior del techo volado.
- Edificio escalonado (dos volúmenes de distinta altura adosados): el wizard NO lo resuelve en una sola corrida. Correr una corrida por volumen, y excluir manualmente de "Loaded" la pared compartida entre ambos (no recibe viento directo).
- Tabla "Generated on Members/Surfaces/Lines No.", columna "Loaded": si queda vacía, la carga NO se distribuye a los elementos — se queda como bloque de superficie sin repartir, sin aviso de error. Hay que seleccionar ahí explícitamente qué miembros reciben la carga.
- No confundir "Without Load Parallel to" (excluye un miembro puntual de recibir carga aunque esté en el plano) con la selección de "Loaded" (sin la cual no hay redistribución en absoluto) — son pasos distintos, hace falta hacer ambos.
- El wizard NO genera automáticamente: presión de voladizo de cubierta (Art. 5.11.4, Cp=0,8 en cara inferior) ni zonas especiales de techo escalonado — cargar manual.

## Snow Load Wizard

- Solo reconoce geometría rectangular plana, monopitch o duopitch simple. NO genera arrastre (Cap.7) ni deslizamiento (Cap.9) en techos escalonados — manual.

## Free Rectangular Load (cargas manuales tipo arrastre, no balanceada custom, etc.)

- A diferencia del Snow/Wind Wizard (que reparte directo a Members), **Free Rectangular Load exige una Surface real asignada** ("Assigned to Surfaces No."). Con techos modelados solo con correas/cabios (sin Surface), crear una Surface nueva con **Stiffness Type** que NO aporte rigidez real, solo para transferencia de carga — buscar en el desplegable una opción tipo "Load transfer surface" (habilita el checkbox "Apply loads using the load transfer surface", que aparece en gris con Stiffness Type "Standard").
- Distribución "Linear in X/Y": el **signo de la coordenada** del segundo corner point define hacia qué lado crece/decrece la rampa — si el triángulo sale espejado hacia el lado contrario a la superficie real, invertir el signo de esa coordenada (no alcanza con elegir bien la distribución). Verificar en **vista 3D/isométrica**, no en planta — en vista de planta (top view) no se nota la altura del triángulo y puede parecer que la carga "no se ve" o se ve como una simple línea.

## Load Transfer Surface — restringir qué miembros reciben la carga

Por defecto, una Surface de reparto (Stiffness Type "Load transfer surface") tiende a repartir la carga a TODOS los miembros dentro de su huella, no solo a las correas — problema cuando conviven correas y cabreadas/cabios en el mismo paño (la carga real de chapa apoya en correas primero, no directo en la cabreada). Se soluciona en las propiedades de la Surface, pestaña **"Load Transfer"** → campo **"Remove influence from members"** → seleccionar ahí los miembros que NO deben recibir carga (cabreadas, cabios, etc.), dejando que solo las correas la reciban. Puede haber una forma más directa de seleccionar positivamente "solo estos miembros" en vez de excluir el resto — pendiente de confirmar si existe.

## Visualización

- Clic derecho sobre una carga generada → **"Display separately"**: cambia de vista de bloque de superficie a carga real por elemento. Usar para diagnosticar si una redistribución (a correas, por ejemplo) funcionó.

## Casillas de opciones de wizards

- **"Lock for new objects"**: destildada mientras se sigue modelando (la carga se extiende sola a miembros nuevos al regenerar). Tildar solo cuando el modelo esté cerrado, para "congelar" el resultado.
- **"Consider member eccentricity"**: destildada por defecto — tildarla EXCLUYE del reparto a miembros con excentricidad fuera de plano (comportamiento contraintuitivo). Solo tildar si hay excentricidades deliberadas que deban quedar afuera.
- **"Consider section distribution"**: solo relevante con secciones ahusadas (tapered) — sin efecto en secciones prismáticas.

## Add-ons (Edit Model → Add-ons)

- Un add-on puede aparecer con punto **verde ("Purchased")** pero el checkbox destildado — significa que la licencia está pero no está activado para ESE modelo. Hay que tildarlo ahí antes de que la funcionalidad esté disponible.
- Modal Analysis / Response Spectrum Analysis: necesarios para método dinámico sísmico. No corrsr sin modelo de masa completo (correas, chapas, todo cargado) — con masa incompleta, el período fundamental sale sesgado.

## Normas de referencia (ASCE dentro de RFEM)

- RFEM no tiene CIRSOC nativo para viento/nieve — usar la edición ASCE más cercana a la base real del CIRSOC vigente (ver `normativa-vigente.md`) y validar manualmente contra las tablas argentinas.
- El generador de combinaciones de carga puede no ofrecer la misma edición ASCE que el generador de cargas — no es un problema real si el factor de carga de viento (1,0) es igual en ambas ediciones, lo cual aplica desde ASCE 7-10 en adelante.
- Diferencias de qh >5% entre ediciones ASCE casi siempre son un dato mal cargado (unidades, campo por defecto en otra unidad), no una diferencia editorial genuina.

## Materiales

- Perfiles W: material ASTM A992 (o A572 Gr.50 si A992 no aparece en la biblioteca — mismo Fy/Fu).
