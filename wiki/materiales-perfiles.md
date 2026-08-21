---
name: materiales-perfiles
description: Equivalencias entre perfiles/materiales de acero americanos y argentinos tradicionales.
status: ESTABLE
last_updated: 2026-08-20
---

# Materiales de acero — perfiles W vs. IPN/UPN tradicionales

| Perfil | Material estándar | Fy | Fu | Uso típico Argentina |
|---|---|---|---|---|
| W (ala ancha, americano) | ASTM A992 (o A572 Gr.50, equivalente) | 345 MPa (50ksi) | 450 MPa (65ksi) | Más eficiente estructuralmente, depende de disponibilidad/flete según zona. Sí se lamina en Argentina (Acindar). |
| IPN/UPN/ángulos (tradicional argentino) | F-24 (IRAM-IAS U 500-503) | 235-240 MPa | 370 MPa | Default histórico de la industria argentina, más fácil de conseguir sin sobrecosto. |
| IPN/UPN/ángulos, alta resistencia | F-36 (IRAM-IAS U 500-503) | 355-360 MPa | 510 MPa | ~15-20% menos peso que F-24 para igual capacidad; se justifica cuando el ahorro en toneladas compensa el mayor costo/kg — típico en cargas elevadas (viento patagónico, zonas sísmicas altas). |

## Decisión práctica

No hay opción "correcta" única — depende de disponibilidad/costo local. Para el precálculo, correr con la opción más económica/disponible primero (F-24 o el material default de la zona) y comparar tonelaje contra la alternativa de mayor resistencia si la diferencia de peso parece significativa. CIRSOC 301 remite a IRAM/IRAM-IAS, no directo a ASTM — confirmar designación IRAM certificada con el proveedor real antes de comprar.
