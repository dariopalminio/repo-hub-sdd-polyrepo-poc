
## 1. Ignorado y descubrimiento deben verificarse juntos

El patrón histórico `repo-service-sdd-polyrepo-poc/*` + `!repo-service-sdd-polyrepo-poc/.git/` se planteó para permitir la detección del hijo. Con la nueva estructura, el hub usa `repos/*` + `!repos/*/.git/`, pero no es equivalente: el directorio padre del hijo queda ignorado y Git no puede re-incluir su `.git/`. El hook instalado respeta `.gitignore`; no basta con que el repo exista en disco. Verifica siempre la salida de su script de estado.

## 2. Staging explícito en el hub

Si el folder del servicio aparece como untracked, `git add .` puede añadir un gitlink sin configurar un submódulo: el código no se sube al hub y otro clon no puede materializarlo a partir de esa referencia. Usa rutas específicas, por ejemplo `git add specs/`, para mantener la integridad del polyrepo.

## 3. Separación conceptual

Agrupar los hijos en `repos/` evita mezclar código y artefactos del hub en la raíz. Si el contenedor aparece como untracked, su nombre hace más evidente su propósito; el movimiento por sí solo no determina lo que muestra `git status`.

El hecho de que git status ya no muestre repos/ es una señal positiva: significa que la exclusión visual de VS Code está funcionando y que Git no ve el directorio como un conjunto de archivos sueltos. La validación real de los hooks se hace, como siempre, comprobando que el agente sigue siendo capaz de navegar y operar sobre el repositorio hijo a través de sus comandos.

## 4. Targets explícitos: soporte pendiente

Los lectores de configuración de la versión instalada de `multi-repo-sync` no leen `targets` desde `.specify/init-options.json`. No confundir el array `targets` de la salida de estado con una opción de configuración. Antes de ignorar `repos/` completamente, confirma que una versión compatible permite descubrimiento y sincronización mediante destinos explícitos.

## 5. La estructura `repos/` mejora la claridad, no elimina el trade-off

La estructura `repos/` mejora la claridad del workspace y documenta la separación hub/repos, pero no elimina el trade-off del `.gitignore` para la detección del hook. Evaluar `targets` explícitos como solución definitiva, sujeto al soporte real de la extensión.

La migración requiere actualizar `path` en `.meta.yaml`, las rutas del workspace y de los planes, y `specs_repo: "../../"` en el store del servicio. El hub configura `scan_depth: 3`; el script PowerShell instalado ya cubre `repos/<repo>` con profundidad 2, por lo que 3 es el valor adoptado, no un mínimo universal. El procedimiento y las verificaciones están en el [runbook](./poc-runbook.md).
