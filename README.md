# POC: Meta-Repo + Spec-Kit para SDD en Polyrepo

## 1. Resumen ejecutivo
Una POC que valida si el patrón Meta-Repo permite a un agente de IA (Copilot en Agent Mode) 
implementar una feature de slicing vertical en un repositorio de código (`repo-service`), 
leyendo la especificación y el contrato desde un repositorio hub central (`repo-hub`), 
sin dispersar los specs ni requerir atomicidad transaccional entre repos.

**Decisión:** ✅ **GO** — La arquitectura es viable para producción con los ajustes documentados.

## 2. Problema y hipótesis
**Problema:** SDD asume que spec e implementación viven en el mismo árbol; el polyrepo garantiza que no. 
El agente no puede ver el hub desde el repo de código, y los specs se fragmentan si se copian.

**Hipótesis:** Con `meta` CLI materializando ambos repos como *peers*, un workspace multi-root en VS Code, 
y las anotaciones `[repo]` en `tasks.md`, el agente escribirá el código en el repo correcto 
sin crear `specs/` dentro de él.

## 3. Arquitectura de la POC (Estructura)
```
poc-sdd-polyrepo/
├── specs/          ← specs canónicos (Spec-Kit vive aquí)
├── repos/          ← repos de código (no versionados en este hub)
│   └── repo-service-sdd-polyrepo-poc/
└── .meta.yaml      ← declara los repos coordinados
```
- **Herramientas:** `meta` CLI 0.2.22, Spec-Kit, `multi-repo-sync`, Copilot Agent Mode, VS Code.
- **Feature de prueba:** `GET /health` → `200 {"status":"ok"}` (contrato OpenAPI 3.1 en el hub).

## 4. Qué se validó (criterios de éxito)

| Criterio | Resultado | Evidencia |
|---|---|---|
| Fuente de verdad única | ✅ | `find` solo devuelve `specs/` en el hub |
| Visibilidad multi-repo del agente | ✅ | Escribió `src/server.js` en `repo-service` |
| Sin dispersión de specs | ✅ | `repo-service` no tiene `specs/` ni `.specify/` |
| Contrato con dueño único | ✅ | `openapi.yaml` solo en el hub, lintado con Redocly |
| Propagación automática de ramas | ✅ | `after_tasks` creó `001-health-endpoint` en `repo-service` |
| Trazabilidad | ✅ | Commit del servicio referencia el feature ID |

## 5. Hallazgos y limitaciones
### Bugs / comportamientos no documentados
- **El `.gitignore` del hub bloquea la detección del repo hijo** en modo `independent`. Solución: 
```gitignore
# .gitignore del hub
repos/*
!repos/*/.git/
```
- **`after_plan` devuelve `state: absent`** cuando la rama no existe, no cuando falta el repo. Mensaje ambiguo.
- **`after_tasks` crea la rama sin el prefijo `feat/`** declarado en `plan.md`.

### Limitaciones de la POC
- **Coding Agent en la nube no soportado:** está sandboxed a un solo repo; la POC usa Agent Mode local.
- **Node 20.20 ejecutado, Node 22+ declarado:** pendiente re-verificar con la versión objetivo.
- **Sin atomicidad transaccional:** cada repo produce su propio commit y PR; el merge es secuencial.

## 6. Cómo reproducir
```bash
# 1. Clonar el hub y los repos hijos
git clone <hub-url> && cd poc-sdd-polyrepo
meta git update

# 2. Abrir el workspace multi-root en VS Code
code poc-sdd-polyrepo.code-workspace

# 3. Ejecutar el flujo en Copilot Agent Mode
/speckit.specify → /speckit.plan → /speckit.tasks → /speckit.implement
```

## 7. Recomendación y próximos pasos
**GO.** La arquitectura Meta-Repo es viable para SDD en polyrepo con Spec-Kit, 
siempre que se documenten y mitiguen los tres hallazgos de la sección 5.

**Próximos pasos:**
1. Reportar los tres hallazgos al mantenedor de `multi-repo-sync`.
2. Validar el flujo completo de merge ordenado (hub → service).
3. Probar con un segundo repo de código (`bff` o `front`) para validar el escalado a N repos.

## Reglas de trabajo

- **Nunca ejecutes `git add .` en la raíz.** Git trataría `repos/` como
  gitlink y rompería el clonado. Usa rutas específicas: `git add specs/`.
- Los specs viven **solo** en `specs/`. No crees `specs/` en los repos hijos.
- Los repos hijos se clonan y sincronizan con `meta git update`.
- Abre `poc-sdd-polyrepo.code-workspace` en VS Code para ver hub y repos
  en un solo workspace multi-root.

## Clonar el workspace completo

```bash
git clone <hub-url> && cd poc-sdd-polyrepo
meta git update 
```
Esto clona el hub y materializa los repos hijos en repos/.
