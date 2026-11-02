# Runbook: Configurar Commits Automáticos en Spec Kit

- **Responsable:** Equipo de Ingeniería / DevOps
- **Último ensayo:** 2026-10-31
- **Resultado esperado:** Commits automáticos ejecutándose tras completar cada comando Spec Kit (specify → plan → tasks → implement), manteniendo un historial Git limpio y ordenado sin intervención manual

## Cuándo usar

Utiliza este runbook cuando necesites habilitar commits automáticos en Spec Kit para:
- Automatizar la generación de commits tras cada etapa del ciclo SDD
- Mantener un historial Git limpio y ordenado sin ejecución manual de `git commit`
- Implementar una estrategia de control de versiones consistente en el equipo
- Sincronizar automáticamente los cambios en el repositorio Git tras modificaciones de especificaciones, planes, tareas e implementación

Este procedimiento aplica a cualquier proyecto con Spec Kit inicializado y un repositorio Git válido.

## Preparación

- **Herramientas y acceso necesarios:**
  - Git instalado y configurado (`git config user.name` y `git config user.email`)
  - Spec-Kit inicializado en el proyecto (debe existir `.specify/` en la raíz)
  - Permisos de escritura en el repositorio Git
  - Un repositorio Git válido en el directorio del proyecto
  - Editor de texto o VS Code para modificar archivos YAML

- **Servicio y entorno de destino:**
  - Repositorio local en `D:\MyProjects\poc-sdd-polyrepo` (o ruta equivalente del proyecto)
  - Archivo de configuración: `.specify/extensions/git/git-config.yml`

- **Ventana de ejecución aprobada:**
  - Cualquier momento en que tengas acceso al repositorio
  - Recomendado fuera de horas de trabajo colaborativo intenso

- **Paneles de control y documentación pertinentes:**
  - [Documentación oficial de Spec Kit - Git Extension](https://docs.example.com/spec-kit/git)
  - [Conventional Commits](https://www.conventionalcommits.org/)
  - [Guía SDD del Polyrepo POC](../sdd-and-polyrepo.md)

## Procedimiento

1. **Verificar el entorno de destino**
   - Instrucción: Confirma que Git, Spec Kit y el repositorio están disponibles
   - Comandos: `specify --version`, `git status`, `Test-Path .specify -Type Container`
   - Observación esperada: Los comandos se ejecutan sin errores; el repositorio está disponible

2. **Ubicar el archivo de configuración**
   - Instrucción: Verifica que existe `.specify/extensions/git/git-config.yml`
   - Comando: `Test-Path .specify\extensions\git\git-config.yml -Type Leaf`
   - Observación esperada: Devuelve `True`

3. **Familiarizarse con opciones de configuración**
   - Instrucción: Abre `.specify/extensions/git/git-config.yml` e identifica las secciones: `commit_style`, `auto_commit.default`, y configuraciones específicas por comando
   - Observación esperada: El archivo está bien formado con indentación YAML correcta (2 espacios)

4. **Elegir estrategia de implementación**
   - Instrucción: Selecciona entre:
     - A) Habilitar globalmente (`default: true`)
     - B) Habilitar solo en etapas clave
     - C) Usar Conventional Commits
   - Observación esperada: Estrategia documentada y lista para aplicar

5. **Aplicar la configuración**
   - Instrucción: Edita `.specify/extensions/git/git-config.yml` con VS Code o PowerShell según tu estrategia
   - Comando: `code .specify\extensions\git\git-config.yml`
   - Observación esperada: El archivo es guardado; sin errores de sintaxis YAML

6. **Validar la configuración**
   - Instrucción: Verifica que el YAML es válido
   - Comandos: `cat .specify/extensions/git/git-config.yml` y `(Get-Content .specify/extensions/git/git-config.yml) | Select-String -Pattern "default:"`
   - Observación esperada: La salida muestra `default: true` (o `false`, según estrategia elegida)

7. **Configurar usuario Git**
   - Instrucción: Ejecuta los comandos si no está configurado
   - Comandos: `git config user.name "Tu Nombre"` y `git config user.email "tu.email@ejemplo.com"`
   - Verificación: `git config --list | Select-String "user\."`
   - Observación esperada: Aparecen nombre y email en la salida

8. **Probar la configuración**
   - Instrucción: Ejecuta un comando Spec Kit pequeño
   - Comandos: `echo "test" > test-file.txt`, `git add test-file.txt`, `specify clarify my-feature`, `git log --oneline -5`
   - Observación esperada: Aparecen dos commits nuevos: uno de Spec Kit y uno anterior

9. **Verificar el resultado**
   - Instrucción: Confirma que los commits automáticos se crean tras cada comando
   - Comando: `git log --grep="\[Spec Kit\]"`
   - Observación esperada: Se muestran commits con etiqueta `[Spec Kit]` en el historial

10. **Documentar el estado**
    - Instrucción: Registra la estrategia utilizada y los cambios realizados en la configuración
    - Observación esperada: Registro completado en `git log`

## Si un paso falla

**Detenerse ante las siguientes condiciones:**
- El archivo `.specify/extensions/git/git-config.yml` no existe o no es accesible
- Errores de validación YAML (indentación incorrecta)
- Git no está configurado con `user.name` y `user.email`
- No hay cambios pendientes para commitear tras ejecutar comandos Spec Kit
- El valor `default` sigue siendo `false` tras la edición

**Registrar el paso fallido:** Documenta qué paso falló, el error exacto recibido y las circunstancias de la falla.

**Procedimiento de recuperación:**
1. **Verificar causa raíz** — Revisa la sección "Solución de problemas" en la documentación del runbook
2. **Solucionar específicamente:**
   - Si `default: false`: Cambia a `true` según Paso 3
   - Si Git no está configurado: Ejecuta `git config user.name "Tu Nombre"` y `git config user.email "tu.email@ejemplo.com"`
   - Si YAML está malformado: Verifica indentación con `Get-Content .specify/extensions/git/git-config.yml \| Select-String -Pattern "auto_commit" -Context 2`
   - Si no hay cambios: Verifica con `git status` que hay cambios sin stagear
3. **Contactar al equipo de guardia:** Utiliza el canal #spec-kit-support en Slack o abre un issue en el repositorio si el problema persiste

## Checklist de completitud

- [ ] Archivo `.specify/extensions/git/git-config.yml` existe y es accesible
- [ ] He elegido una estrategia (A: global, B: por etapa, o C: Conventional Commits)
- [ ] He actualizado `auto_commit.default` o los valores específicos en la configuración
- [ ] He validado el YAML sin errores de indentación
- [ ] He ejecutado `git config user.name` y `git config user.email`
- [ ] He probado con un comando Spec Kit pequeño
- [ ] He verificado que el commit se creó automáticamente
- [ ] He revisado el historial con `git log --oneline`
- [ ] Resultado esperado verificado
- [ ] Cambios registrados en el historial de cambios (`git log`)
- [ ] Documentación de estrategia actualizada (si se modificó el procedimiento)

## Documentación adicional: Estrategias de configuración

### Archivo de configuración `.specify/extensions/git/git-config.yml`

El archivo contiene las siguientes secciones principales:

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
```

#### Opciones de configuración:

| Clave | Valores | Descripción |
|-------|---------|-------------|
| `commit_style` | `fixed`, `conventional` | `fixed`: usa mensajes predefinidos. `conventional`: genera mensajes automáticos siguiendo Conventional Commit |
| `auto_commit.default` | `true`, `false` | Habilita/deshabilita auto-commit globalmente |
| `auto_commit.after_<cmd>.enabled` | `true`, `false` | Habilita/deshabilita auto-commit para un comando específico |
| `auto_commit.after_<cmd>.message` | string | Mensaje personalizado (solo con `commit_style: fixed`) |

### Estrategia A: Habilitar globalmente (Recomendado)

```yaml
auto_commit:
  default: true
```

Commits automáticos tras todos los comandos Spec Kit. Configurable simple.

### Estrategia B: Habilitar solo en etapas clave

```yaml
auto_commit:
  default: false
  after_specify:
    enabled: true
  after_plan:
    enabled: true
  after_tasks:
    enabled: true
  after_implement:
    enabled: true
```

Commits automáticos solo en las 4 etapas principales; control granular.

### Estrategia C: Conventional Commits

```yaml
commit_style: conventional
auto_commit:
  default: true
```

Mensajes generados automáticamente: `feat:`, `docs:`, `refactor:`, etc. Compatible con release automation.

## Solución de problemas

### Problema: "El commit no se ejecutó automáticamente"

**Causa y solución:**

| Causa | Verificación | Solución |
|-------|--------------|----------|
| `default` sigue siendo `false` | `(Get-Content .specify/extensions/git/git-config.yml) \| Select-String -Pattern "^\s*default:"` | Cambia a `true` en `.specify/extensions/git/git-config.yml` |
| No hay cambios para commitear | `git status` | Spec Kit solo commitea si hay cambios sin stagear; esto es correcto |
| Git no está configurado | `git config --list \| Select-String "user\."` | Ejecuta `git config user.name "Tu Nombre"` y `git config user.email "tu.email@ejemplo.com"` |
| YAML malformado | `Get-Content .specify/extensions/git/git-config.yml \| Select-String -Pattern "auto_commit" -Context 2` | Verifica indentación: `default:` debe estar indentado con 2 espacios bajo `auto_commit:` |

### Problema: "¿Puedo deshabilitar auto-commit después?"

Sí. Cambia `auto_commit.default` a `false` en `.specify/extensions/git/git-config.yml`. Los commits anteriores permanecen en el historial.

### Problema: "¿Cómo customizo los mensajes?"

Con `commit_style: fixed`, edita los mensajes en `.specify/extensions/git/git-config.yml`:

```yaml
after_specify:
  enabled: true
  message: "[MI-ETIQUETA] Nueva especificación"
```

Con `commit_style: conventional`, los mensajes se generan automáticamente leyendo los cambios.

### Referencia rápida de comandos

| Tarea | Comando |
|-------|---------|
| Verificar configuración actual | `cat .specify/extensions/git/git-config.yml` |
| Editar configuración | `code .specify/extensions/git/git-config.yml` |
| Ver historial de commits | `git log --oneline -10` |
| Deshabilitar auto-commit | Cambiar `default: true` a `false` en git-config.yml |
| Ver commits automáticos | `git log --grep="\[Spec Kit\]"` |
| Verificar usuario Git | `git config --list \| Select-String "user\."` |
