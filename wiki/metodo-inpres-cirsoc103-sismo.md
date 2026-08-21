---
name: metodo-inpres-cirsoc103-sismo
description: Metodología de carga sísmica según INPRES-CIRSOC 103 Parte I, incluye combinación de acciones (Art. 3.7). EN DESARROLLO — ver sección Pendientes al pie.
status: ACTIVO
last_updated: 2026-08-21
---

# Carga sísmica — INPRES-CIRSOC 103 Parte I

Ver `normativa-vigente.md` — zonificación clásica de 5 zonas sigue vigente pese al mapa PSHA nuevo (no incorporado al reglamento todavía).

## Secuencia de parámetros

1. **Zona sísmica**: Anexo A del reglamento lista zona por departamento/localidad directamente — no hace falta interpolar. Consultar también www.inpres.gob.ar para cruzar.
2. **Método estático vs. dinámico**: admitido directo (sin necesidad de chequear regularidad de Tabla 2.5) si altura < 9m o ≤3 niveles. Para estructuras más altas, chequear Tabla 2.5. La regularidad (Tablas 2.3/2.4) igual importa después para la excentricidad accidental, aunque no bloquee la elección del método.
3. **Grupo/factor de riesgo γr**: Grupo Ao=1,5 (esencial/emergencia), Grupo A=1,3 (uso público >300m² y >100 personas — incluye estadios/gimnasios), Grupo B=1,0 (estándar), Grupo C=0,8.
4. **Clasificación del sitio**: SA-SF según velocidad de onda de corte/SPT. Sin estudio de suelo real, adoptar SD como hipótesis conservadora razonable — NUNCA reemplaza un estudio geotécnico real antes del cálculo definitivo.
5. **Ca, Cv** (Tabla 3.1, según zona+sitio) → T2=Cv/(2,5·Ca), T1=0,2×T2, T3 (Tabla 3.2), as (aceleración efectiva de zona).
6. **Período aproximado Ta** = Cr×H^x (Tabla 6.2). Para sistemas mixtos sin categoría específica en la tabla, usar "Otros sistemas estructurales" (Cr=0,0488, x=0,75).
7. **Coeficiente sísmico C**: si Ta≤T2 → ec.6.3 = 2,5×Ca×γr/R. Comparar SIEMPRE contra el piso mínimo (ec.6.5 para zonas 3-4, ec.6.6 para zonas 0-1-2: C≥0,11×Ca×γr) — no es una elección entre fórmulas, son dos condiciones independientes, tomar la mayor.
8. **R (factor de reducción)**: Tabla 5.1 por sistema estructural. Para precálculo, adoptar el nivel NO especial/convencional (más conservador, no compromete a detallado dúctil todavía). **Estructura mixta (5.1.1)**: tomar el R MÍNIMO entre los sistemas que actúan en paralelo (ej. pórtico arriostrado acero R=3 + pórtico hormigón ductilidad limitada R=3,5 → usar R=3).
9. **W (peso sísmico)**: NO estimar a mano si se puede evitar — modelar primero con viento+nieve+gravitatorias (que no dependen del sismo), dejar que el software dimensione secciones reales, y usar el peso propio real resultante. Sumar sobrecarga con factor f1 (Tabla 3.3, según probabilidad de ocupación — con tribunas/asambleas usar probabilidad intermedia f1=0,50, no la reducida) y nieve con f2=0,20 si el techo evacúa nieve.

## Simultaneidad con viento

NO se combinan viento y sismo (Art. 3.7.4, confirmado con el texto del reglamento) — son casos independientes a envolver, no sumar.

## Combinación de acciones (Art. 3.7, confirmado con el texto del reglamento)

- **[3.16]** 1,20D ± 1,00E + f1×L + f2×S
- **[3.17]** 0,9D ± 1,00E
- **[3.18]** E = EH + EV (definición — NO es la fórmula de magnitud de Ev, esa está en otra parte, pendiente de ubicar)
- **[3.19]** (estructura de acero, componentes sensibles a sobrerresistencia) 1,20D ± Ω0×EH + 0,5L + 0,2S
- **[3.20]** 0,9D ± Ω0×EH

## W (peso sísmico) — estado actual

W = D + f1×L + f2×S. Para este proyecto puntual:
- **D**: resuelto (autopeso real LC1 + peso de chapa LC30, ya modelados).
- **f2×S**: resuelto conceptualmente (0,20 × envolvente de nieve RC1) — falta armar la combinación en RFEM.
- **f1×L**: probablemente **≈0 para este proyecto** — las tribunas están apoyadas en el suelo con fundación propia, independientes de columnas/vigas/pórticos (no le transmiten masa sísmica al sistema resistente que se está diseñando), y todo el edificio es a nivel de suelo (sin entrepisos apoyados en la estructura). **Condición para que esto sea válido**: debe existir junta de separación sísmica real entre tribuna y edificio (evitar golpeteo/pounding) — confirmar que está contemplada en el proyecto.

## Pendientes por cerrar (no dar por completo el cálculo sin esto)

- **Fórmula de magnitud de Ev**: la ec.3.18 solo define E=EH+EV, no da el valor de Ev — buscar en el reglamento (probablemente cerca del Cap.6, junto a las ecuaciones de C).
- **Art. 6.3(a)**: acción sísmica vertical específica para voladizos/balcones/aleros — aparte de Ev general, verificación puntual para cualquier voladizo de cubierta.
- **Junta de separación sísmica tribuna-edificio**: confirmar que está prevista en el proyecto (condición para que f1×L≈0 sea válido).
- **Torsión accidental (Tabla 6.3)**: 0%/5%/10% según regularidad en planta (Tabla 2.3) — pendiente de clasificar con la geometría en planta completa (el volumen adosado más bajo genera asimetría a revisar).
- **Rigidez de diafragma (8.2.1)**: techo de chapa sobre reticulado — **asumido FLEXIBLE como hipótesis de precálculo** (no rígido), sin demostración formal todavía. Permite diseñar cada pórtico por área de influencia sin torsión entre pórticos, y además exime del cálculo de los casos torsionales de viento (ASCE 7 Apéndice D — exención requiere h≤9,1m, que ya se cumple, + diafragma flexible). Confirmar en el cálculo definitivo antes de dar por buena la exención.
- **Fundaciones (Cap.9)**: arriostramiento de bases — pendiente para etapa de detallado.

## RFEM

RFEM SÍ tiene INPRES-CIRSOC 103 soportado nativamente (a diferencia de viento/nieve) — vía add-on **RF-DYNAM Pro** (Modal Analysis + Response Spectrum Analysis). Chequear que el add-on esté tildado en Edit Model→Add-ons (puede aparecer con punto verde "Purchased" pero destildado igual — hay que activarlo ahí antes de usarlo). Para el método estático no hace falta ese add-on — se puede aplicar como carga estática equivalente distribuida en altura, sin modelo de masa detallado.
