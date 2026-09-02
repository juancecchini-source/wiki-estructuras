---
name: materiales-perfiles
description: Equivalencias entre perfiles/materiales de acero americanos y argentinos tradicionales.
status: ACTIVO
last_updated: 2026-09-02
---

# Materiales de acero — perfiles W vs. IPN/UPN tradicionales

| Perfil | Material estándar | Fy | Fu | Uso típico Argentina |
|---|---|---|---|---|
| W (ala ancha, americano) | ASTM A992 (o A572 Gr.50, equivalente) | 345 MPa (50ksi) | 450 MPa (65ksi) | Más eficiente estructuralmente, depende de disponibilidad/flete según zona. Sí se lamina en Argentina (Acindar). |
| IPN/UPN/ángulos (tradicional argentino) | F-24 (IRAM-IAS U 500-503) | 235-240 MPa | 370 MPa | Default histórico de la industria argentina, más fácil de conseguir sin sobrecosto. |
| IPN/UPN/ángulos, alta resistencia | F-36 (IRAM-IAS U 500-503) | 355-360 MPa | 510 MPa | ~15-20% menos peso que F-24 para igual capacidad; se justifica cuando el ahorro en toneladas compensa el mayor costo/kg — típico en cargas elevadas (viento patagónico, zonas sísmicas altas). |

## Calidades adoptadas por defecto — criterio de trabajo (Juan, 02/09)

**Estas son las calidades con las que se trabaja en Argentina. Usarlas siempre en cualquier cálculo, salvo que Juan indique otra cosa para un proyecto puntual.**

| Elemento | Calidad | Fy a usar |
|---|---|---|
| Perfiles **W** | **F-36** | **355 MPa** |
| **Correas** (conformados en frío: C, U, Z) | **F-24** | **235 MPa** |
| **Chapas** | **F-24** | **235 MPa** |
| **Redondos** (tensores, barras) | **F-24** | **235 MPa** |

**Por qué importa fijarlo**: los defaults de RFEM son designaciones **ASTM** (A992, A572 Gr.50 con Fy=344,7 MPa), que además de no ser las normas IRAM-IAS que exigen los CIRSOC, no coinciden en Fy con lo que se compra acá. En las **correas** la diferencia es grande y en el sentido inseguro — A572 Gr.50 tiene **47% más de Fy que el F-24** real, y eso solo se descubre mirando qué material tiene puesto el modelo. En los **W** el error va al revés y es chico: A992 (345) contra F-36 (355) son 2,9% **del lado seguro**.

> Ojo con la trampa de las correas: el conformado en frío se verifica por CIRSOC 303, cuyo art. A.2.1 admite **solo normas IRAM-IAS**. Ver `metodo-cirsoc301-acero.md`, sección de perfiles conformados en frío.

## Decisión práctica

No hay opción "correcta" única — depende de disponibilidad/costo local. Para el precálculo, correr con la opción más económica/disponible primero (F-24 o el material default de la zona) y comparar tonelaje contra la alternativa de mayor resistencia si la diferencia de peso parece significativa. CIRSOC 301 remite a IRAM/IRAM-IAS, no directo a ASTM — confirmar designación IRAM certificada con el proveedor real antes de comprar.
