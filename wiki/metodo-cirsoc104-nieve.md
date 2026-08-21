---
name: metodo-cirsoc104-nieve
description: Metodología para determinar y aplicar carga de nieve según CIRSOC 104-2005 (vigente), incluyendo arrastre y deslizamiento en techos escalonados.
status: ESTABLE
last_updated: 2026-08-20
---

# Carga de nieve — CIRSOC 104-2005

**Ojo: es la edición 2005, no la 2026** (todavía en trámite, no vigente — ver `normativa-vigente.md`).

## Secuencia de parámetros

1. **pg (carga de nieve sobre el suelo)**: Tabla 1.x por provincia, listada por localidad + HSNM (altura sobre nivel del mar). A diferencia del viento, acá la altitud SÍ es un dato de entrada directo.
2. **Ce (exposición)**: según categoría de terreno (misma A-D que viento) + exposición del techo (totalmente expuesto / parcialmente expuesto [default] / protegido). Totalmente expuesto + Cat.C → Ce=0,9.
3. **Ct (térmico)**: 1,0 si el espacio está calefaccionado, 1,2 si NO — ojo, esto se define **por cubierta**, no por edificio completo. Un edificio puede tener una cubierta calefaccionada (vestuarios) y otra no (nave principal sin clima) con Ct distintos.
4. **I (importancia)**: misma lógica de Categoría de Riesgo que viento. Categoría III → I=1,1.
5. **pf (carga en cubierta plana)** = 0,7 × Ce × Ct × I × pg. Mínimo aplicable si pendiente < (21/W)+0,5° (umbral de "baja pendiente"): pf,mín = I×pg — usar el mayor de los dos.
6. **Cs (factor de pendiente)**: 1,0 si pendiente ≤15° (superficie lisa) o ≤45° (otra superficie), con Ct=1,2. Umbrales más bajos si Ct=1,0.
7. **ps = Cs × pf** — carga balanceada final, proyectada en horizontal.

## Carga no balanceada

Aplica si 2,1°(aprox, depende de W) < pendiente < 70°. Con W>6m: sotavento = 1,2×(1+β/2)×ps/Ce (β=1,0 si pg≤1); barlovento = 0,3×ps. Gobierna el sotavento.

## Cap. 7 — Acumulación por arrastre de viento en escalones de techo

Para techo bajo adosado a pared más alta:
1. hc = desnivel real entre cubiertas.
2. hb = ps(del techo bajo)/γ, con γ=0,426×pg+2,2 (peso específico de la nieve).
3. Chequeo de aplicabilidad: hc/hb > 0,2 → aplica.
4. hd = leer de Figura 9 (curvas por lu = longitud de cubierta a barlovento, eje x = pg) — **interpolación logarítmica** en lu si el valor está entre curvas tabuladas (30/60/120/180m). Barlovento = 0,75×hd(sotavento); gobierna el mayor entre sotavento y barlovento.
5. Si hd > hc: TRUNCAR — altura de diseño = hc, ancho w = 4×hd²/hc.
   Si hd ≤ hc: NO truncar — altura de diseño = hd, ancho w = 4×hd (fórmula simple).
6. Chequear w ≤ 8×hc y w ≤ ancho de la cubierta baja.
7. pd = (hc o hd, el que se usó como altura de diseño) × γ — carga puntual que se suma a la balanceada, decreciendo linealmente hasta 0 en el borde del triángulo de ancho w.

## Cap. 9 — Nieve caída por deslizamiento

Aplica si la cubierta superior es lisa (chapa cuenta) con pendiente >2%, u otra superficie con pendiente >16%. Fórmula: carga total por metro de alero = 0,4 × pf(de la cubierta SUPERIOR, no la baja) × W(distancia horizontal alero-cumbrera de la cubierta superior), distribuida uniformemente sobre 4,5m de ancho desde el alero de la cubierta baja (reducir proporcionalmente si el ancho disponible es menor a 4,5m).

## Regla crítica: Cap.7 y Cap.9 NO se suman entre sí

Confirmado contra la norma madre (ASCE): son mecanismos físicos distintos (arrastre = durante la nevada; deslizamiento = derretimiento posterior) y el reglamento los trata como **casos de carga alternativos a envolver**, no aditivos. Cada uno se suma por separado a la balanceada. Armar 3 casos: (1) balanceada sola, (2) balanceada+arrastre, (3) balanceada+deslizamiento — dejar que el software tome la envolvente.

## Cargas parciales (Cap. 5)

Solo aplica a **correas continuas multi-tramo**. Correas simplemente apoyadas (tramo único entre pórticos): no aplica, y es una simplificación válida para precálculo.

## RFEM

El Snow Load Wizard solo reconoce geometría de cubierta rectangular plana, monopitch o duopitch simple. NO genera automáticamente arrastre (Cap.7) ni deslizamiento (Cap.9) en techos escalonados — hay que cargarlos manualmente como superficie/línea con patrón trapezoidal/triangular, usando los valores calculados a mano.
