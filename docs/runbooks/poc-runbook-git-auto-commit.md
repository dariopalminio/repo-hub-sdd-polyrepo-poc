# Runbook: Configurar Commits Automáticos en Spec Kit

Este runbook te guía a través de la configuración de commits automáticos en Spec Kit, eliminando la necesidad de ejecutar `git commit` manualmente después de cada etapa del ciclo SDD (specify → plan → tasks → implement).

## 🎯 Objetivo

Habilitar commits automáticos que se ejecuten tras completar cada comando Spec Kit, manteniendo un historial Git limpio y ordenado sin intervención manual.

## 📋 Requisitos previos

Antes de empezar, verifica que tienes:
- **Git** instalado y configurado (`git config user.name` y `git config user.email`)
- **Spec-Kit inicializado** en tu proyecto (debe existir `.specify/` en la raíz)
- **Permisos de escritura** en el repositorio Git
- **Un repositorio Git válido** en el directorio del proyecto

Verifica la inicialización:

```powershell
# Confirmar que Spec Kit está instalado
specify --version

# Confirmar que estamos en un repo Git
git status

# Verificar que la carpeta .specify existe
Test-Path .specify -Type Container
```

---

## 🪜 Paso 1: Ubicar el archivo de configuración Git

El archivo de configuración se encuentra en:

```
.specify/extensions/git/git-config.yml
```

Verifica que existe:

```powershell
cd D:\MyProjects\poc-sdd-polyrepo  # O tu ruta del proyecto
Test-Path .specify\extensions\git\git-config.yml -Type Leaf
```

Si devuelve `True`, estás listo para continuar.

---

## 🔧 Paso 2: Entender las opciones de configuración

Abre el archivo [`.specify/extensions/git/git-config.yml`](./.specify/extensions/git/git-config.yml) y familiarízate con las secciones:

```yaml
# Estilo de commit: "fixed" o "conventional"
commit_style: fixed

# Toggle global y configuración por comando
auto_commit:
  default: false           # Toggle global (false = deshabilitado)
  after_specify:
    enabled: false
    message: "[Spec Kit] Add specification"
  after_plan:
    enabled: false
    message: "[Spec Kit] Add implementation plan"
  after_tasks:
    enabled: false
    message: "[Spec Kit] Add tasks"
  after_implement:
    enabled: false
    message: "[Spec Kit] Implementation progress"
  # ... más comandos disponibles
```

### Opciones principales:

| Clave | Valores | Descripción |
|-------|---------|-------------|
| `commit_style` | `fixed`, `conventional` | `fixed`: usa mensajes predefinidos. `conventional`: genera mensajes automáticos siguiendo el estándar Conventional Commit |
| `auto_commit.default` | `true`, `false` | Habilita/deshabilita auto-commit globalmente para todos los comandos |
| `auto_commit.after_<cmd>.enabled` | `true`, `false` | Habilita/deshabilita auto-commit para un comando específico |
| `auto_commit.after_<cmd>.message` | string | Mensaje de commit personalizado (solo aplica con `commit_style: fixed`) |

---

## ✅ Paso 3: Elegir tu estrategia

### Estrategia A: Habilitar globalmente (Recomendado para principiantes)

Abre [`.specify/extensions/git/git-config.yml`](./.specify/extensions/git/git-config.yml) y cambia:

```yaml
auto_commit:
  default: true  # ← Cambiar de 'false' a 'true'
```

**Resultado:** Commits automáticos tras **todos** los comandos Spec Kit (specify, plan, tasks, implement, etc.).

**Ventaja:** Configuración simple, historial Git limpio sin esfuerzo.

---

### Estrategia B: Habilitar solo en etapas clave

Si prefieres más control, habilita solo los comandos importantes:

```yaml
auto_commit:
  default: false  # Mantener deshabilitado globalmente
  after_specify:
    enabled: true
    message: "[Spec Kit] Add specification"
  after_plan:
    enabled: true
    message: "[Spec Kit] Add implementation plan"
  after_tasks:
    enabled: true
    message: "[Spec Kit] Add tasks"
  after_implement:
    enabled: true
    message: "[Spec Kit] Implementation progress"
```

**Resultado:** Commits automáticos solo en las 4 etapas principales, sin commits intermedios.

**Ventaja:** Control granular; evita commits innecesarios.

---

### Estrategia C: Mensajes dinámicos (Conventional Commits)

Para un historial más estructurado, usa Conventional Commits:

```yaml
commit_style: conventional  # ← Cambiar de 'fixed'

auto_commit:
  default: true
```

**Resultado:** Mensajes generados automáticamente como:
- `feat: add health-check endpoint specification`
- `docs: update implementation plan with API schema`
- `refactor: reorganize task structure`

**Ventaja:** Historial Git más semántico; compatible con herramientas de release automation.

---

## 🛠️ Paso 4: Aplicar la configuración

Una vez hayas elegido tu estrategia, edita el archivo:

### Opción 1: Con VS Code

1. Abre VS Code en tu workspace
2. Presiona `Ctrl+P`
3. Busca y abre `.specify/extensions/git/git-config.yml`
4. Realiza los cambios según tu estrategia (Paso 3)
5. Guarda el archivo (`Ctrl+S`)

### Opción 2: Con PowerShell

```powershell
# Editar con notepad (mantiene formato YAML)
notepad .specify\extensions\git\git-config.yml

# O con tu editor preferido
code .specify\extensions\git\git-config.yml
```

---

## ✨ Paso 5: Validar la configuración

Verifica que tu archivo YAML es válido:

```powershell
# Verificar que el archivo existe y es accesible
cat .specify/extensions/git/git-config.yml | Select-Object -First 20

# Confirmar que `default` está establecido correctamente
(Get-Content .specify/extensions/git/git-config.yml) | 
  Select-String -Pattern "default:" | Select-Object -First 1
```

Ejemplo de salida esperada (Estrategia A):
```
  default: true
```

---

## 🧪 Paso 6: Probar la configuración

Ahora prueba el auto-commit con un pequeño cambio:

```powershell
# 1. Crear un cambio dummy (ej: agregar un archivo temp)
echo "test" > test-file.txt
git add test-file.txt

# 2. Ejecutar un comando Spec Kit (ej: specify o clarify)
specify clarify my-feature

# 3. Verificar el historial Git
git log --oneline -5
```

**Resultado esperado:** Deberías ver dos commits nuevos:
1. `[Spec Kit] ...` (auto-commit de Spec Kit)
2. Tu commit anterior

Si ves solo uno o ninguno, verifica:
- ¿Tienes cambios sin commitear antes de correr Spec Kit?
- ¿`default: true` está activo en la configuración?
- ¿Es un repositorio Git válido? (`git status` sin errores)

---

## 🚀 Paso 7: Usar en tu flujo de trabajo SDD completo

Una vez validado, usa normalmente:

```powershell
# Etapa 1: Especificación
specify specify "Crear endpoint de health check"
# → Auto-commit: "[Spec Kit] Add specification"

# Etapa 2: Planificación (revisar primero)
specify plan
# → Auto-commit: "[Spec Kit] Add implementation plan"

# Etapa 3: Generación de tareas
specify tasks
# → Auto-commit: "[Spec Kit] Add tasks"

# Etapa 4: Implementación
specify implement
# → Auto-commit: "[Spec Kit] Implementation progress"
```

No necesitas ejecutar `git commit` manualmente entre etapas.

---

## 🐛 Solución de problemas

### "El commit no se ejecutó automáticamente"

**Causa 1:** `default` sigue siendo `false`

```powershell
# Verificar valor actual
(Get-Content .specify/extensions/git/git-config.yml) | 
  Select-String -Pattern "^\s*default:"
```

**Solución:** Cambia a `true` según Paso 3.

---

**Causa 2:** No hay cambios para commitear

Spec Kit solo commitea si hay cambios sin stagear. Si no hay cambios, salta silenciosamente (esto es correcto).

```powershell
# Verificar cambios
git status
```

---

**Causa 3:** Git no está configurado

```powershell
# Configurar usuario de Git (requerido para commits)
git config user.name "Tu Nombre"
git config user.email "tu.email@ejemplo.com"

# Verificar configuración
git config --list | Select-String "user\."
```

---

**Causa 4:** Archivo YAML malformado

YAML es sensible a indentación. Verifica:

```powershell
# Comparar con un formato correcto
Get-Content .specify/extensions/git/git-config.yml | 
  Select-String -Pattern "auto_commit" -Context 2
```

Asegúrate de que `default:` está indentado con 2 espacios bajo `auto_commit:`.

---

### "¿Puedo deshabilitar auto-commit después?"

Sí. Simplemente cambia:

```yaml
auto_commit:
  default: false  # ← Vuelve a false
```

Los commits anteriores permanecen en el historial.

---

### "¿Cómo customizo los mensajes?"

Con `commit_style: fixed`, edita los mensajes:

```yaml
after_specify:
  enabled: true
  message: "[MI-ETIQUETA] Nueva especificación"  # ← Personaliza aquí
```

Con `commit_style: conventional`, los mensajes se generan automáticamente leyendo los cambios.

---

## 📚 Referencia rápida

| Tarea | Comando |
|-------|---------|
| Verificar configuración actual | `cat .specify/extensions/git/git-config.yml` |
| Editar configuración | `code .specify/extensions/git/git-config.yml` |
| Ver historial de commits | `git log --oneline -10` |
| Deshabilitar auto-commit | Cambiar `default: true` a `false` en git-config.yml |
| Ver commits automáticos | `git log --grep="\[Spec Kit\]"` |

---

## ✅ Checklist de completitud

- [ ] Archivo `.specify/extensions/git/git-config.yml` existe
- [ ] He elegido una estrategia (A, B o C)
- [ ] He actualizado `auto_commit.default` o los valores específicos
- [ ] He validado el YAML (sin errores de indentación)
- [ ] He ejecutado `git config user.name` y `git config user.email`
- [ ] He probado con un comando Spec Kit pequeño
- [ ] He verificado que el commit se creó automáticamente
- [ ] He revisado el historial con `git log --oneline`

---

## 🎓 Lectura adicional

- [Documentación oficial de Spec Kit - Git Extension](https://docs.example.com/spec-kit/git)
- [Conventional Commits](https://www.conventionalcommits.org/) — Estándar para mensajes semánticos
- [Guía SDD del Polyrepo POC](../sdd-and-polyrepo.md)

---

**Última actualización:** 2026-10-31  
**Autor:** Copilot  
**Estado:** Listo para producción
