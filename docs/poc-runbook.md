# Runbook: hub SDD con repos de código en `repos/`

Con los dos repositorios remotos ya creados, la guía asume que partes de cero: sin carpeta local, sin `.meta.yaml`, sin Spec-Kit inicializado. El objetivo es que termines con un **Meta-Repo funcional** donde `repo-hub-sdd-polyrepo-poc` es el hub de specs y `repo-service-sdd-polyrepo-poc` es el repo de código, ambos visibles para Copilot en un solo workspace.

Los specs canónicos viven en `specs/`; los repos de código independientes se agrupan en `repos/`, sin versionar su contenido ni gitlinks en el hub. Si el hijo ya está clonado en la raíz, sigue la sección **Migrar un workspace existente** en lugar de volver a clonarlo.

### 📋 Requisitos previos

Antes de empezar, verifica que tienes instalado:
- **Git** (con Git Credential Manager para autenticarte contra GitHub)
- **`meta` CLI** (`meta --version` debe responder)
- **`uv`** y **Spec-Kit** (`specify --version`)
- **VS Code** con GitHub Copilot Chat en Agent Mode

#### Instalar meta

Para instalar la CLI de `meta`, sigue las instrucciones oficiales según tu sistema operativo. Por ejemplo, en Windows:

```powershell
irm https://raw.githubusercontent.com/gitkb/meta/main/install.ps1 | iex
```
O de esta manera:
```powershell
powershell -Command "irm https://raw.githubusercontent.com/gitkb/meta/main/install.ps1 | iex"
meta --version
```
# Ver la ruta exacta del ejecutable que se está llamando
```powershell
Get-Command meta | Select-Object Source
```

---

### 🪜 Paso 1: Clonar el repo hub y darle un commit inicial

El repo `repo-hub-sdd-polyrepo-poc` será el meta-repo. Como está vacío, Git no permite clonarlo de forma útil hasta que tenga al menos un commit. La forma más limpia es inicializarlo localmente, hacer el primer commit y pushear:

```powershell
# Crear una carpeta temporal para el hub
mkdir D:\MyProjects\poc-sdd-polyrepo
cd D:\MyProjects\poc-sdd-polyrepo

# Inicializar el repo local
git init
git branch -M main

# Crear un README mínimo (el repo no puede estar vacío)
echo "# Hub de Specs (SDD Polyrepo POC)" > README.md
git add README.md
git commit -m "chore: initial commit"

# Conectar con el remoto y pushear
git remote add origin https://github.com/dariopalminio/repo-hub-sdd-polyrepo-poc.git
git push -u origin main
```

> **Si el repo remoto ya tiene algún commit** (por ejemplo, un README generado por GitHub), omite el `git init` y haz directamente `git clone https://github.com/dariopalminio/repo-hub-sdd-polyrepo-poc.git D:\MyProjects\poc-sdd-polyrepo`.

---

### 🪜 Paso 2: Declarar el repo servicio en `.meta.yaml`

Dentro de `D:\MyProjects\poc-sdd-polyrepo`, crea el archivo `.meta.yaml` que declara el repo de código como hijo del hub:

```yaml
# D:\MyProjects\poc-sdd-polyrepo\.meta.yaml
projects:
  repo-service-sdd-polyrepo-poc:
    repo: https://github.com/dariopalminio/repo-service-sdd-polyrepo-poc.git
    path: repos/repo-service-sdd-polyrepo-poc
    tags: [backend]
```

Este archivo es el corazón del Meta-Repo: `meta` lo lee para saber qué repos hijos debe clonar y gestionar. `path` indica la ruta relativa al hub donde debe materializarse el hijo. La URL HTTPS funciona sin configuración adicional; `meta` la pasa directamente a `git clone`.

---

### 🪜 Paso 3: Commitear el `.meta.yaml` y clonar el repo hijo

El `.meta.yaml` debe estar versionado en el hub para que `meta git clone` funcione desde fuera en el futuro. Además, el repo servicio debe tener al menos un commit para que `git clone` no falle con *"empty repository"*.

**3.1. Asegura un commit inicial en el repo servicio** (si está vacío, clónalo temporalmente, añade un README y pushea):

```powershell
# Temporal: clonar el repo servicio, añadir README y pushear
cd D:\temp
git clone https://github.com/dariopalminio/repo-service-sdd-polyrepo-poc.git service-init
cd service-init
echo "# Service (SDD Polyrepo POC)" > README.md
git add README.md
git commit -m "chore: initial commit"
git branch -M main
git push -u origin main
cd D:\temp
Remove-Item -Recurse -Force service-init
```

**3.2. Commitear el `.meta.yaml` en el hub y clonar el hijo:**

```powershell
cd D:\MyProjects\poc-sdd-polyrepo
git add .meta.yaml
git commit -m "chore: add meta config"
git push

# Clonar el repo servicio dentro del contenedor repos
meta git update
```

`meta git update` lee el `.meta.yaml` local y clona cualquier repositorio hijo que aún no exista. El resultado debe ser:

```
D:\MyProjects\poc-sdd-polyrepo\
├── .meta.yaml
├── README.md
├── .git/
└── repos\
    └── repo-service-sdd-polyrepo-poc\   ← repo Git independiente
        ├── README.md
        └── .git/
```

Verifica con `meta project list` (deben aparecer ambos) y `meta git status` (debe reportar el estado de los dos repos en un solo comando).

---

### 🪜 Paso 4: Inicializar Spec-Kit en el hub

Spec-Kit **solo se inicializa en el hub**, nunca en el repo servicio. El hub es el único lugar donde vivirán los specs.

```powershell
cd D:\MyProjects\poc-sdd-polyrepo

# Inicializar Spec-Kit en el directorio actual (--here) con integración Copilot
specify init --here --integration copilot
```

El flag `--here` le dice a Spec-Kit que inicialice en el directorio existente en lugar de crear uno nuevo. Esto crea `.specify/` y los comandos `/speckit.*` para Copilot.

El `.gitignore` actual del hub usa este patrón para el contenedor de repos independientes:

```text
# .gitignore del hub
repos/*
!repos/*/.git/
```

**Advertencia:** este patrón no garantiza la detección del hook ni que `repos/` aparezca como untracked. Git no puede re-incluir `.git/` si su directorio padre sigue ignorado: en esta estructura, `git check-ignore -v -- repos/repo-service-sdd-polyrepo-poc` indica que el hijo está ignorado, y el hook instalado respeta ese filtro. Comprueba la detección del paso 6 antes de dar por válida la configuración; aumentar `scan_depth` no evita el filtro de `.gitignore`.

**Nunca ejecutes `git add .` en la raíz.** Si un repo hijo queda visible para Git, podrías añadirlo como gitlink sin la configuración de un submódulo y dejar un clon del hub incompleto. Usa rutas específicas, por ejemplo `git add specs/` o `git add .meta.yaml .gitignore`.

Luego:
```powershell
git add .gitignore
git commit -m "chore: ignore child repo"
```

---

### 🪜 Paso 5: Configurar el repo servicio (sin Spec-Kit)

Entra en `repos/repo-service-sdd-polyrepo-poc/` y crea el archivo de configuración que declara dónde viven los specs canónicos:

```powershell
cd D:\MyProjects\poc-sdd-polyrepo\repos\repo-service-sdd-polyrepo-poc

# Crear el archivo de store
mkdir .specify-store
```

Crea `.specify-store/config.yaml`:

```yaml
# repos/repo-service-sdd-polyrepo-poc/.specify-store/config.yaml
specs_repo: "../../"
specs_path: "specs"
```

Añade también un `.github/copilot-instructions.md` en el repo servicio:

```markdown
# repo-service (POC)

- El contrato API y el spec canónico viven en `../../specs/` (el hub).
- Implementa SOLO lo que indique `../../specs/001-health-check/tasks.md`.
- No crees `specs/` ni `.specify/` en este repo.
```

Commit y push en el repo servicio:

```powershell
git add .specify-store .github
git commit -m "chore: configure store pointer to hub specs"
git push
```

**No instales Spec-Kit aquí.** `repo-service-sdd-polyrepo-poc` debe permanecer sin `.specify/` ni `specs/`. Ese es el criterio de éxito central.

---

### 🪜 Paso 6: Instalar la extensión `multi-repo-sync` (opcional pero recomendada)

La extensión propaga la rama de feature a los repos hijos automáticamente cuando ejecutas `/speckit.plan` y `/speckit.tasks`. Sin ella, tendrás que crear la rama `feat/001-health-check` manualmente en el repo servicio.

Desde el hub:

```powershell
cd D:\MyProjects\poc-sdd-polyrepo
specify extension add multi-repo-sync
```
O así si no funciona:
```powershell
specify extension add multi-repo-sync --from https://github.com/fyloss/spec-kit-multi-repo-sync/releases/download/v1.0.0/spec-kit-multi-repo-sync.zip
```

La extensión usa hooks `after_plan` y `after_tasks`, no sobrescribe los comandos core, y sobrevive a `specify self upgrade`.

En `.specify/init-options.json`, conserva las demás claves y configura:

```json
{
  "multi_repo_branching": {
    "type": "independent",
    "scan_depth": 3
  }
}
```

El workspace actual usa `scan_depth: 3`. En el script PowerShell instalado, la profundidad cuenta directorios por debajo del hub: el hijo en `repos/<repo>` está a nivel 2, por lo que 2 ya lo cubre y 3 deja margen para otro nivel. No todas las versiones cuentan la profundidad igual; verifica siempre el resultado real. Las claves activas de `.specify/extensions/multi-repo-sync/multi-repo-sync-config.yml` tienen precedencia sobre este JSON.

Verifica el descubrimiento sin crear ramas ni modificar el plan:

```powershell
cd D:\MyProjects\poc-sdd-polyrepo
& .\.specify\extensions\multi-repo-sync\scripts\powershell\sync-status.ps1 -Json
```

El array `targets` debe incluir `repos/repo-service-sdd-polyrepo-poc`. Si está vacío, revisa `.gitignore`, las exclusiones y la profundidad antes de continuar con la sincronización automática. Luego puedes ejecutar `/speckit.multi-repo-sync.analyze` desde Copilot para registrar los repos afectados en el plan.

En la comprobación del workspace actual, el script devolvió `mode: independent`, `scan_depth: 3` y `targets: []`, coherente con el directorio hijo ignorado. La estructura ya está migrada, pero el descubrimiento automático sigue pendiente de resolver.

**Alternativa a evaluar, no configuración soportada actualmente:** ignorar `repos/` por completo y declarar `targets` explícitos permitiría mantener limpio el estado del hub si la versión del hook admitiera esa opción. Los lectores de configuración de la versión instalada no leen `multi_repo_branching.targets`; no añadas esa clave suponiendo que habilita el descubrimiento. El array `targets` de la salida JSON no implica soporte de esa opción de entrada.

---

### 🪜 Paso 7: Abrir el workspace multi-root en VS Code

Crea el archivo `poc-sdd-polyrepo.code-workspace` en la raíz del hub:

```json
{
  "folders": [
    { "path": ".", "name": "hub (specs)" },
    { "path": "repos/repo-service-sdd-polyrepo-poc", "name": "service (code)" }
  ]
}
```

Ábrelo en VS Code con `File > Open Workspace from File...`. Ahora Copilot Chat tiene visibilidad de ambos directorios: el hub (donde viven los specs) y el repo servicio (donde se implementa).

---

### 🪜 Paso 8: Ejecutar el flujo de Spec-Kit

Con el workspace multi-root abierto y Copilot en **Agent Mode**, ejecuta los comandos desde el hub. El agente leerá el spec del hub y escribirá el código en el repo servicio.

**8.1. `/speckit.specify`**  
Describe la feature: *"Añadir un endpoint `GET /health` en el repo servicio que devuelva `{"status":"ok"}`. El contrato OpenAPI debe reflejarlo."*  
Spec-Kit generará `specs/001-health-check/spec.md` en el hub.

**8.2. `/speckit.plan`**  
Una vez verificado el descubrimiento del paso 6, el hook `after_plan` de `multi-repo-sync` detectará `repos/repo-service-sdd-polyrepo-poc/` y añadirá la tabla **"Affected Repositories"** a `plan.md`. La columna `Repo Path` debe contener la ruta completa relativa al hub, incluido `repos/`.

**8.3. `/speckit.tasks`**  
Se genera `tasks.md`. **Edítalo manualmente** para anotar el repo destino en cada tarea:

```markdown
# specs/001-health-check/tasks.md
- [ ] T001 [hub] Actualizar `contracts/openapi.yaml` con `GET /health`
- [ ] T002 [repo-service-sdd-polyrepo-poc] Implementar `GET /health` en `src/` según el contrato
- [ ] T003 [repo-service-sdd-polyrepo-poc] Test que verifique `{"status":"ok"}`
```

El hook `after_tasks` creará la rama `feat/001-health-check` en el repo servicio automáticamente.

Verifica:
```powershell
cd D:\MyProjects\poc-sdd-polyrepo\repos\repo-service-sdd-polyrepo-poc
git branch
# Ejemplo de salida de la POC:
* 001-health-endpoint
  master
cd ..\..
```

**8.4. `/speckit.implement`**  
El agente leerá `tasks.md`, verá `[repo-service-sdd-polyrepo-poc]` en T002 y T003, navegará al folder `repos/repo-service-sdd-polyrepo-poc/` del workspace, leerá `.specify-store/config.yaml` (que confirma que los specs están en `../../specs/`), e implementará el endpoint en `src/health.py` (o el equivalente en tu stack).

---

### 🪜 Paso 9: Verificar los criterios de éxito

Desde `D:\MyProjects\poc-sdd-polyrepo`:

```powershell
# 1. Fuente de verdad única
Get-ChildItem -Recurse -Directory -Filter "specs" | Select-Object FullName
# Debe devolver SOLO ...\poc-sdd-polyrepo\specs

# 2. Visibilidad multi-repo
Get-ChildItem repos\repo-service-sdd-polyrepo-poc\src
# Debe contener la implementación según el stack (en esta POC: server.js)

# 3. Sin dispersión
Test-Path repos\repo-service-sdd-polyrepo-poc\.specify
Test-Path repos\repo-service-sdd-polyrepo-poc\specs
# Ambos deben devolver False

# 4. Trazabilidad
cd repos\repo-service-sdd-polyrepo-poc
git log --oneline -1
# El mensaje debe referenciar 001-health-check

# 5. meta CLI
cd ..\..
meta git status
# Debe reportar el estado de ambos repos en un solo comando
```

---

### Migrar un workspace existente

Si el hijo todavía vive en la raíz, muévelo sin re-clonar. `Move-Item` mueve el directorio completo, incluido su `.git`; no hace falta usar `git mv` en el hub porque el hijo no debe estar versionado allí. Si ya existe en `repos/`, omite el movimiento.

```powershell
cd D:\MyProjects\poc-sdd-polyrepo

# Antes: registra rama, cambios locales y últimos commits
git -C repo-service-sdd-polyrepo-poc status --short --branch
git -C repo-service-sdd-polyrepo-poc log --oneline -3

# Si repos no existe, créalo; no sobrescribas un destino existente
if (Test-Path -LiteralPath .\repos\repo-service-sdd-polyrepo-poc) {
    throw "El destino ya existe; verifica la migración antes de mover el repo."
}
New-Item -ItemType Directory -Path repos -Force
Move-Item -LiteralPath .\repo-service-sdd-polyrepo-poc -Destination .\repos\ -ErrorAction Stop

# Después: deben conservarse rama, cambios e historial
git -C repos\repo-service-sdd-polyrepo-poc status --short --branch
git -C repos\repo-service-sdd-polyrepo-poc log --oneline -3
```

Actualiza conjuntamente:

1. `.meta.yaml`: `path: repos/repo-service-sdd-polyrepo-poc`.
2. `.gitignore`: revisa el patrón del paso 4 y verifica su efecto en el hook.
3. `poc-sdd-polyrepo.code-workspace`: ruta del folder `repos/repo-service-sdd-polyrepo-poc`, preservando las demás opciones.
4. El store del servicio: `specs_repo: "../../"` y referencias a specs en sus instrucciones.
5. `.specify/init-options.json`: configuración del paso 6, preservando las demás claves.
6. La tabla **Affected Repositories** y las rutas de tareas/planes existentes que aún apunten al hijo en la raíz.

Verifica desde el hub:

```powershell
meta project list
meta git status
& .\.specify\extensions\multi-repo-sync\scripts\powershell\sync-status.ps1 -Json
git status --short
```

Desde Copilot, ejecuta `/speckit.multi-repo-sync.analyze` para refrescar el plan de la feature activa. Si no detecta el hijo, no consideres completada la integración del hook.

La carpeta `repos/` separa claramente hub y código y escala a N repos. Si aparece como untracked, su propósito es más evidente que el de N hijos sueltos en la raíz; esa aparición depende de las reglas de ignorado, no del movimiento en sí. La tensión entre descubrimiento del hook y limpieza del estado del hub sigue necesitando una decisión explícita.

### Documentar el hub

El README del hub debería explicar la estructura `specs/` + `repos/` + `.meta.yaml`, la prohibición de `git add .` en la raíz, la fuente única de specs y la apertura del workspace multi-root. Para un clon nuevo de un hub ya inicializado:

```powershell
git clone https://github.com/dariopalminio/repo-hub-sdd-polyrepo-poc.git D:\MyProjects\poc-sdd-polyrepo
cd D:\MyProjects\poc-sdd-polyrepo
meta git update
```

Así se materializan los hijos en las rutas declaradas en `.meta.yaml`. Consulta también los [hallazgos de la POC](./poc-notes.md). El README actual requiere una revisión separada de sus conflictos; este runbook no los resuelve.

---

### 💎 Resultado esperado

Si los cinco criterios se cumplen, habrás validado que **`meta` CLI materializa el Meta-Repo, Spec-Kit centraliza la intención en el hub, y Copilot escribe en el repo servicio sin dispersar specs**. La arquitectura funciona con solo dos repos, y escala a N agregando más entradas al `.meta.yaml` y más folders al workspace multi-root.

**Limitación que la POC asume:** el Coding Agent en la nube de GitHub está sandboxed a un solo repositorio y no puede ver ambos. Esta POC usa **Agent Mode local en VS Code**, que sí tiene visibilidad del workspace multi-root.
