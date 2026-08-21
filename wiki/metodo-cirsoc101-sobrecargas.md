---
name: metodo-cirsoc101-sobrecargas
description: Metodología para sobrecarga de mantenimiento de cubierta según CIRSOC 101-25 (vigente desde enero 2026), artículo 4.8.1.
status: ESTABLE
last_updated: 2026-08-21
---

# Sobrecarga de mantenimiento de cubierta — CIRSOC 101-25, Art. 4.8.1

**CIRSOC 101-25 es la edición vigente desde enero 2026** (reemplaza a 101-2005) — ver `normativa-vigente.md`. Aplica a cubiertas planas u horizontales, con pendiente o curvas, inaccesibles salvo con fines de mantenimiento.

## Cubiertas pesadas vs. livianas (Art. 4.8.1)

Se definen por **peso propio total de la cubierta** (estructura soporte + cerramiento superior):
- **Pesada**: peso total > 0,5 kN/m² → Art. 4.8.1.(a)
- **Liviana**: peso total ≤ 0,5 kN/m² → Art. 4.8.1.(b)

Techos de chapa sobre correas metálicas (sin losa) suelen caer en **liviana** — verificar sumando peso de chapa + correas antes de asumirlo.

## Fórmula — cubiertas livianas, Art. 4.8.1.(b)

**Lr = 0,45 × R1 × R2** (kN/m² de proyección horizontal), con **0,203 ≤ Lr ≤ 0,765**

**R1** (por área tributaria At, en m², del elemento estructural analizado — ej. la correa, no el pórtico):
- R1 = 1 para At < 20 m²
- R1 = 1,125 − 0,00625×At para 20 ≤ At ≤ 60 m²
- R1 = 0,75 para At > 60 m²

**R2** (por pendiente p, en % — para cubiertas planas; para curvas p=200×f/L con f=flecha, L=luz de tramo):
- R2 = 1,70 para 0 ≤ p < 3
- R2 = 1,04 − 0,008×p para 3 ≤ p ≤ 55
- R2 = 0,60 para p > 55

## RFEM

**Lr es horizontal-proyectado** ("por m² de proyección horizontal", texto literal) — igual convención que la nieve (`metodo-cirsoc104-nieve.md`). Cargarlo con **Free Rectangular Load, proyección "XY Plane"** (NO como Surface Load directa sobre el área real, que es la técnica correcta para peso propio de chapa, no para esta carga).

## Combinaciones

Lr **no se suma con la nieve** (van en el mismo "casillero" de la combinación de diseño, se envuelve el mayor — ver `metodo-cirsoc104-nieve.md` sección de combinaciones). Conviene agregarlo como un caso alternativo más dentro de la misma Result Combination de "cubierta variable" que ya envuelve los casos de nieve, en vez de armar un bloque aparte.

## R1 por elemento — cuándo vale la pena refinar

El área tributaria de R1 es del **elemento estructural que se está chequeando** (correa, pórtico, columna), no de la chapa ni de las uniones — cada uno tiene su propia área tributaria, creciente a medida que se sube en el camino de cargas. En RFEM esto no se puede resolver con una sola carga de superficie (todo el camino de cargas queda atado a una única magnitud dentro de un mismo Load Case) — si hace falta el refinamiento, se resuelve con **Load Cases separados por nivel** (ej. uno con R1=1 para diseñar correas, otro con R1 reducido —hasta 0,75— para pórticos/columnas), aplicados con la misma técnica pero magnitudes distintas, usando los resultados del caso que corresponda según el elemento a verificar.

**Antes de invertir tiempo en el refinamiento, comparar Lr contra la envolvente de nieve del proyecto**: si la nieve balanceada sola ya supera el Lr sin reducir (R1=1), la nieve gobierna en todos lados y el valor de R1 elegido para Lr no cambia ningún resultado — no vale la pena refinar, alcanza un solo Lr conservador (R1=1) por cumplimiento normativo. **Esto se invierte en zonas sin nieve significativa** (Rosario y gran parte del país fuera de la cordillera/meseta patagónica, donde CIRSOC 104 prácticamente no aplica): ahí el Lr pasa a ser la sobrecarga variable de cubierta dominante (no hay nieve compitiendo), así que el refinamiento de R1 por elemento sí vale la pena y conviene hacerlo desde el principio.
