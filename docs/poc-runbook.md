Perfecto. Con los dos repositorios remotos ya creados, la guía asume que partes de cero: sin carpeta local, sin `.meta.yaml`, sin Spec-Kit inicializado. El objetivo es que termines con un **Meta-Repo funcional** donde `repo-hub-sdd-polyrepo-poc` es el hub de specs y `repo-service-sdd-polyrepo-poc` es el repo de código, ambos visibles para Copilot en un solo workspace.

### 📋 Requisitos previos

Antes de empezar, verifica que tienes instalado:
- **Git** (con Git Credential Manager para autenticarte contra GitHub)
- **`meta` CLI** (`meta --version` debe responder)
- **`uv`** y **Spec-Kit** (`specify --version`)
- **VS Code** con GitHub Copilot Chat en Agent Mode

---

### 🪜 Paso 1: Clonar el repo hub y darle un commit inicial

El repo `repo-hub-sdd-polyrepo-poc` será el meta-repo. Como está vacío, Git no permite clonarlo de forma útil hasta que tenga al menos un commit. La forma más limpia es inicializarlo localmente, hacer el primer commit y pushear:

```powershell
# Crear una carpeta temporal para el hub
mkdir D:\Projects\poc-sdd-polyrepo
cd D:\Projects\poc-sdd-polyrepo

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

> **Si el repo remoto ya tiene algún commit** (por ejemplo, un README generado por GitHub), omite el `git init` y haz directamente `git clone https://github.com/dariopalminio/repo-hub-sdd-polyrepo-poc.git D:\Projects\poc-sdd-polyrepo`.

---

### 🪜 Paso 2: Declarar el repo servicio en `.meta.yaml`

Dentro de `D:\Projects\poc-sdd-polyrepo`, crea el archivo `.meta.yaml` que declara el repo de código como hijo del hub:

```yaml
# D:\Projects\poc-sdd-polyrepo\.meta.yaml
projects:
  repo-service-sdd-polyrepo-poc:
    repo: https://github.com/dariopalminio/repo-service-sdd-polyrepo-poc.git
    tags: [backend]
```

Este archivo es el corazón del Meta-Repo: `meta` lo lee para saber qué repos hijos debe clonar y gestionar. La URL HTTPS funciona sin configuración adicional; `meta` la pasa directamente a `git clone`.

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
cd D:\Projects\poc-sdd-polyrepo
git add .meta.yaml
git commit -m "chore: add meta config"
git push

# Clonar el repo servicio como carpeta hermana dentro del hub
meta git update
```

`meta git update` lee el `.meta.yaml` local y clona cualquier repositorio hijo que aún no exista. El resultado debe ser:

```
D:\Projects\poc-sdd-polyrepo\
├── .meta.yaml
├── README.md
├── .git/
└── repo-service-sdd-polyrepo-poc\   ← repo hijo clonado aquí
    ├── README.md
    └── .git/
```

Verifica con `meta project list` (deben aparecer ambos) y `meta git status` (debe reportar el estado de los dos repos en un solo comando).

---

### 🪜 Paso 4: Inicializar Spec-Kit en el hub

Spec-Kit **solo se inicializa en el hub**, nunca en el repo servicio. El hub es el único lugar donde vivirán los specs.

```powershell
cd D:\Projects\poc-sdd-polyrepo

# Inicializar Spec-Kit en el directorio actual (--here) con integración Copilot
specify init --here --integration copilot
```

El flag `--here` le dice a Spec-Kit que inicialice en el directorio existente en lugar de crear uno nuevo. Esto crea `.specify/` y los comandos `/speckit.*` para Copilot.

Añade un `.gitignore` en el hub para que el repo hijo no se versione como parte del padre (es un repo independiente):

```text
# .gitignore del hub
repo-service-sdd-polyrepo-poc/*
!repo-service-sdd-polyrepo-poc/.git/
```
Luego:
```powershell
git add .gitignore
git commit -m "chore: ignore child repo"
```

---

### 🪜 Paso 5: Configurar el repo servicio (sin Spec-Kit)

Entra en `repo-service-sdd-polyrepo-poc/` y crea el archivo de configuración que declara dónde viven los specs canónicos:

```powershell
cd D:\Projects\poc-sdd-polyrepo\repo-service-sdd-polyrepo-poc

# Crear el archivo de store
mkdir .specify-store
```

Crea `.specify-store/config.yaml`:

```yaml
# repo-service-sdd-polyrepo-poc/.specify-store/config.yaml
specs_repo: "../"
specs_path: "specs"
```

Añade también un `.github/copilot-instructions.md` en el repo servicio:

```markdown
# repo-service (POC)

- El contrato API y el spec canónico viven en `../specs/` (el hub).
- Implementa SOLO lo que indique `../specs/001-health-check/tasks.md`.
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
cd D:\Projects\poc-sdd-polyrepo
specify extension add multi-repo-sync
```
O así si no funciona:
```powershell
specify extension add multi-repo-sync --from https://github.com/fyloss/spec-kit-multi-repo-sync/releases/download/v1.0.0/spec-kit-multi-repo-sync.zip
```

La extensión usa hooks `after_plan` y `after_tasks`, no sobrescribe los comandos core, y sobrevive a `specify self upgrade`.

---

### 🪜 Paso 7: Abrir el workspace multi-root en VS Code

Crea el archivo `poc-sdd-polyrepo.code-workspace` en la raíz del hub:

```json
{
  "folders": [
    { "path": ".", "name": "hub (specs)" },
    { "path": "repo-service-sdd-polyrepo-poc", "name": "service (code)" }
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
El hook `after_plan` de `multi-repo-sync` detectará `repo-service-sdd-polyrepo-poc/` y añadirá la tabla **"Affected Repositories"** a `plan.md`, registrando la rama `feat/001-health-check`.

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
cd repo-service-sdd-polyrepo-poc
git branch
poc-sdd-polyrepo\repo-service-sdd-polyrepo-poc> git branch
* 001-health-endpoint
  master
```

**8.4. `/speckit.implement`**  
El agente leerá `tasks.md`, verá `[repo-service-sdd-polyrepo-poc]` en T002 y T003, navegará al folder `repo-service-sdd-polyrepo-poc/` del workspace, leerá `.specify-store/config.yaml` (que confirma que los specs están en `../specs/`), e implementará el endpoint en `src/health.py` (o el equivalente en tu stack).

---

### 🪜 Paso 9: Verificar los criterios de éxito

Desde `D:\Projects\poc-sdd-polyrepo`:

```powershell
# 1. Fuente de verdad única
Get-ChildItem -Recurse -Directory -Filter "specs" | Select-Object FullName
# Debe devolver SOLO ...\poc-sdd-polyrepo\specs

# 2. Visibilidad multi-repo
Get-ChildItem repo-service-sdd-polyrepo-poc\src\health.*
# Debe existir el archivo implementado por el agente

# 3. Sin dispersión
Test-Path repo-service-sdd-polyrepo-poc\.specify
Test-Path repo-service-sdd-polyrepo-poc\specs
# Ambos deben devolver False

# 4. Trazabilidad
cd repo-service-sdd-polyrepo-poc
git log --oneline -1
# El mensaje debe referenciar 001-health-check

# 5. meta CLI
cd ..
meta git status
# Debe reportar el estado de ambos repos en un solo comando
```

---

### 💎 Resultado esperado

Si los cinco criterios se cumplen, habrás validado que **`meta` CLI materializa el Meta-Repo, Spec-Kit centraliza la intención en el hub, y Copilot escribe en el repo servicio sin dispersar specs**. La arquitectura funciona con solo dos repos, y escala a N agregando más entradas al `.meta.yaml` y más folders al workspace multi-root.

**Limitación que la POC asume:** el Coding Agent en la nube de GitHub está sandboxed a un solo repositorio y no puede ver ambos. Esta POC usa **Agent Mode local en VS Code**, que sí tiene visibilidad del workspace multi-root.

