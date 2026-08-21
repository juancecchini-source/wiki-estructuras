# Wiki de Ingeniería — Setup

## Qué es esto

Volcado inicial de la wiki técnica, destilado del chat de precálculo del polideportivo de Añelo. Mismo protocolo que la wiki de Taller ICE (carpeta plana, ~150 líneas por archivo, frontmatter con status, edición in-place).

## Pasos para dejarlo andando

1. **Creá el repo** (separado de Taller ICE — son dominios de conocimiento distintos):
   ```bash
   mkdir wiki-ingenieria && cd wiki-ingenieria
   git init
   ```
2. **Copiá la carpeta `wiki/`** de este volcado adentro del repo recién creado.
3. **Subilo a GitHub** (recomendado como fuente de verdad en vez de OneDrive, por el tema de conflictos de sync concurrente):
   ```bash
   git add .
   git commit -m "Primer volcado: viento y nieve completos, sismo en curso"
   git remote add origin <URL de tu repo privado en GitHub>
   git push -u origin main
   ```
4. **Abrí Claude Code parado en esa carpeta** para seguir manteniéndola con el mismo protocolo mecánico de Taller ICE (edición in-place, grep de duplicados antes de cerrar sesión, partir archivos que superen ~160-180 líneas).
5. **Armá un Project en claude.ai** para las consultas de trabajo del día a día (como la de este chat) — subí ahí los archivos de `metodo-*.md` y `normativa-vigente.md` como project knowledge. Recordá: cuando Claude Code edite un archivo, hay que reemplazarlo a mano en el Project (no hay sync automático salvo herramientas de terceros no oficiales).

## Qué falta para la próxima sesión de Claude Code

- Terminar de cerrar `metodo-inpres-cirsoc103-sismo.md` (status ACTIVO) una vez que el precálculo de sismo esté completo en el chat — pasarlo a ESTABLE.
- Actualizar `proyecto-polideportivo-anelo.md` con los resultados finales cuando el precálculo esté cerrado.
- Si empezás otro proyecto (Bariloche, Cutral Có, etc.), crear su propio `proyecto-*.md` — los archivos de método ya están listos para reusar tal cual.
