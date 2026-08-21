---
name: metodo-inpres-cirsoc103-sismo
description: Metodología de carga sísmica según INPRES-CIRSOC 103 Parte I. EN DESARROLLO — ver sección Pendientes al pie.
status: ACTIVO
last_updated: 2026-08-20
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

NO se combinan viento y sismo (Art. 3.7.4) — son casos independientes a envolver, no sumar.

## Pendientes por cerrar (no dar por completo el cálculo sin esto)

- **Ev (componente vertical del sismo)**: E = EH ± EV (ec.3.18) — falta calcular y sumar.
- **Art. 6.3(a)**: acción sísmica vertical específica para voladizos/balcones/aleros — aparte de Ev general, verificación puntual para cualquier voladizo de cubierta.
- **Torsión accidental (Tabla 6.3)**: 0%/5%/10% según regularidad en planta (Tabla 2.3) — pendiente de clasificar con la geometría en planta completa (el volumen adosado más bajo genera asimetría a revisar).
- **Rigidez de diafragma (8.2.1)**: techo de chapa sobre reticulado probablemente es diafragma FLEXIBLE (no rígido) — permitiría diseñar cada pórtico por área de influencia sin torsión entre pórticos. Confirmar antes de complicarse con torsión global.
- **Fundaciones (Cap.9)**: arriostramiento de bases — pendiente para etapa de detallado.

## RFEM

RFEM SÍ tiene INPRES-CIRSOC 103 soportado nativamente (a diferencia de viento/nieve) — vía add-on **RF-DYNAM Pro** (Modal Analysis + Response Spectrum Analysis). Chequear que el add-on esté tildado en Edit Model→Add-ons (puede aparecer con punto verde "Purchased" pero destildado igual — hay que activarlo ahí antes de usarlo). Para el método estático no hace falta ese add-on — se puede aplicar como carga estática equivalente distribuida en altura, sin modelo de masa detallado.
