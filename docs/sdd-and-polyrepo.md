# SDD sobre polyrepo

## Definición del problema: 

SDD y polyrepo tienen unidades de trabajo estructuralmente incompatibles. SDD opera sobre la especificación (una historia, una feature, un comportamiento end-to-end), que es inherentemente vertical y cross-cutting: atraviesa frontend, BFF, y N servicios. Polyrepo opera sobre el repositorio, que es inherentemente horizontal y bounded: cada repo tiene su propio .git, su propio CI, su propia historia y su propio ciclo de release. La herramienta de SDD asume que el spec y su implementación viven en el mismo árbol de trabajo; el polyrepo garantiza que no.

El problema surge cuando:
- El frontend y el backend deben estar en repositorios separados.
- Un solo producto, varios repositorios (microservicios, bff, etc.).
- Un mismo producto varias aplicaciones (product, android-app, ios-app, web-app).

## Los siete sub-problemas
1. Fragmentación de la fuente de verdad
El spec canónico debe vivir en un solo lugar, pero los repos de código no pueden verlo si no está dentro de su .git. Si el spec vive en product, los agentes que corren en front o service-1 no lo ven. Si el spec se copia a cada repo, se fragmenta y diverge. No hay una ubicación que sea simultáneamente única y visible para todos los agentes.
2. Frontera de contexto del agente
Copilot, Codex y Claude operan con un cwd (o un sandbox de escritura) acotado a un directorio raíz. El agente del backend no ve el frontend, y viceversa. Los workarounds (abrir el directorio padre, --add-dir, multi-root experimental) o saturan el contexto con repos irrelevantes, o fallan de formas documentadas (submódulos invisibles, @ picker ciego al segundo folder).
3. No atomicidad transaccional
Un tasks.md vertical slicing requiere cambios coordinados en cuatro repos. Git no puede commitear atómicamente en cuatro repos independientes. El mantenedor de Spec-Kit lo dice explícitamente: "Spec Kit does not currently provide one atomic project that writes spec.md, plan.md, tasks, and code across several independent Git repositories". Cada repo produce su propio PR, y el merge es necesariamente secuencial.
4. Drift contractual
Sin un contrato con dueño único y versionado, frontend y backend divergen silenciosamente. El frontend asume un campo que el backend nunca implementó; el backend cambia un endpoint que el frontend sigue consumiendo con la firma vieja. El drift no se detecta hasta que algo falla en integración, y para entonces ya hay múltiples commits en múltiples repos.
5. Trazabilidad cross-repo
¿Qué versión del spec corresponde a qué commit en cada repo? Sin un mecanismo explícito, la respuesta es "la que estaba en main cuando se mergeó cada PR", que es una respuesta insuficiente cuando hay que auditar o hacer rollback. Los PRs vinculados por un feature ID ayudan, pero no son una garantía estructural.
6. Coordinación del orden de merge
Los repos tienen dependencias: el contrato debe mergear antes que el consumidor; service-1 antes que bff; bff antes que front. Si el orden se rompe, main queda en un estado inconsistente. Sin un plan.md que declare el orden y sin CI que lo verifique, el orden depende de la memoria del equipo.
7. Dispersión vs. centralización del spec
Las dos únicas opciones puras son malas: specs dispersos (cada repo con su specs/, la recomendación oficial de Spec-Kit para polyrepo, pero con fragmentación y drift) o specs centralizados sin visibilidad (todo en product, pero los agentes no ven el hub). No hay una opción pura que sea simultáneamente centralizada y visible.

## La pregunta que el problema plantea

Formulado en una sola frase: ¿Cómo dar a un agente de IA visibilidad de un contrato centralizado y capacidad de escribir en N repositorios independientes, sin dispersar los specs, sin romper la independencia de git de cada repo, y sin requerir atomicidad transaccional que Git no soporta?

## Estrategias de solución

### Estrategia A: "Cada repo con su spec" (Repo-Local)

```text
Descripción: Cada repositorio de código contiene su propio directorio de especificaciones (ej. .specify/ o openspec/). No existe una fuente de verdad centralizada fuera del repositorio.

     ┌─────────────┼─────────────┐
     │             │             │
 repo-01        repo-02       repo-03
(code repo)    (code repo)    (code repo)
repo-01/specs/001-feature/ # specs del repo r1
repo-02/specs/001-feature/ # specs del repo r2
repo-03/specs/001-feature/ # specs del repo r3
```
### Estrategia B: "Un store (hub), muchos repos"

Un repositorio central y dedicado (el "hub" o "store") contiene todas las especificaciones y cambios. Los repositorios de código consumen estas especificaciones desde el hub.

```text
          product-spec (hub)
             └── specs/
                   ▲
                   │ 
     ┌─────────────┼─────────────┐
     │             │             │
 repo-01        repo-02       repo-03
(code repo)    (code repo)    (code repo)
```
### Estrategia C: "Spec paraguas, una implementación por repositorio"

Un "spec paraguas" (umbrella spec) en un repo de coordinación define el comportamiento end-to-end y los contratos. Luego, cada repositorio de código tiene su propia "spec de implementación" que detalla cómo ese repo cumple su parte del contrato.

```text
product-spec/          # repo de coordinación
  specs/001-feature/
    spec.md           # comportamiento end-to-end y contratos
    plan.md           # ownership por repo y orden de rollout
    contracts/        # APIs/eventos compartidos
                   ▲
                   │ 
     ┌─────────────┼─────────────┐
     │             │             │
 repo-01        repo-02       repo-03
(code repo)    (code repo)    (code repo)
repo-01/specs/001-feature/ # specs de implementación para r1
repo-02/specs/001-feature/ # specs de implementación para r2
repo-03/specs/001-feature/ # specs de implementación para r3
```

## Implementaciones

### Estrategia A: "Cada repo con su spec" (Repo-Local)

**Implementaciones principales:**
- **Spec-Kit (modo por defecto):** La herramienta está diseñada para operar dentro del directorio que contiene .specify/. Cada repo puede tener su propio proyecto de Spec-Kit con su constitution.md, specs/ y numeración de features independiente.
- **OpenSpec (modo repo-local):** Su funcionamiento por defecto es tener una carpeta openspec/ junto al código en cada repositorio.
- **Spec-First (modo multirepo):** Su modelo de workspace multi-repo trata cada repositorio hijo como una frontera de autoridad independiente.

✅ Ventajas:
Simplicidad y aislamiento: Cada equipo es dueño de sus specs sin dependencias externas.
Baja fricción inicial: No requiere configurar capas de coordinación adicionales.

❌ Desventajas:
Visión fragmentada: Un agente en un repo no puede ver los specs de los demás, lo que dificulta las features que cruzan múltiples servicios.
Riesgo de drift: Las especificaciones de diferentes repos pueden desincronizarse con el tiempo.

### Estrategia B: "Un store (hub), muchos repos"

**Implementaciones principales:**
- **OpenSpec Stores (beta):** Un "store" es un repositorio independiente cuyo único trabajo es la planificación. Los repos de código se registran en la máquina para apuntar a él, y el agente lee los specs del store mientras escribe código en el repo correspondiente.
- **Meta-Repo (con Spec-Kit):** Aunque Spec-Kit no tiene un "store" nativo, puedes lograr el mismo efecto creando un repo central (product) que albergue .specify/ y todos los specs. Al abrir un workspace multi-root (con meta git clone), el agente puede ver el hub y los repos de código simultáneamente, y usar /speckit.implement para escribir en los repos hijos.
- **Specularis:** Una capa que se añade sobre tu proyecto para gobernar todos los markdown desde un solo lugar (canónico + human-in-the-loop), con auto-sincronización y gestión de conflictos para que todos los agentes y repos vean las mismas especificaciones canónicas.
- **Polyrepo Workspace Kit:** Una capa de coordinación ligera que ayuda a humanos y agentes a compartir un modelo de workspace explícito sin pretender que todos los repos pertenezcan a un monorepo.
- **Workspace de VS Code + .gitmodules + Make:** es una implementación plausible en teoría, pero en la práctica falla en puntos crítico. El workspace multi-root es útil para ti, pero Copilot Coding Agent no soporta sesiones multi-root de forma nativa. Los submódulos son, con diferencia, la pieza más problemática de la combinación. Hay tres fallos documentados que afectan directamente a tu flujo con Copilot. El code review de Copilot falla cuando un PR toca un submódulo. ¿Y la extensión multi-repo-sync de Spec-Kit? Existe y soporta submódulos: detecta los repos afectados parseando .gitmodules y crea la rama de feature en cada uno. Pero esto solo resuelve la propagación de ramas, no el acceso del agente. La extensión crea la rama, pero Copilot sigue sin poder leer ni escribir en el submódulo. Es un parche parcial, no una solución.

✅ Ventajas:
- Fuente de verdad única: Todos los agentes y equipos consultan las mismas especificaciones.
- Ideal para features cross-repo: El agente tiene contexto completo para planificar cambios que afectan a múltiples servicios.

❌ Desventajas:
- Requiere configuración: Necesitas una herramienta o patrón (Stores, Meta-Repo) para que los agentes lean del hub y escriban en los repos.
- Acoplamiento de herramientas: OpenSpec Stores te ata a OpenSpec; el Meta-Repo con Spec-Kit requiere configurar el workspace multi-root.


### Estrategia C: "Spec paraguas, una implementación por repositorio"

#### Implementaciones principales:
- **Recomendación oficial de Spec-Kit (Discussion #4465):** El mantenedor TongyiDai recomienda explícitamente este patrón: un system-spec/ con el spec.md (comportamiento end-to-end), plan.md (ownership por repo y orden de rollout) y contracts/ (esquemas compartidos). Cada repo (r1, r2, r3) tiene su propio specs/001-feature/ con la implementación específica.
- **Contract-Driven Development (CDD) como metodología:** La guía de Spec-Kit para CDD es el complemento perfecto para esta estrategia. Define que debe haber un dueño autoritativo del contrato, y que los consumidores en otros repositorios deben fijar una versión o sincronizar una copia de solo lectura.

✅ Ventajas:
- **Alineada con la filosofía de Spec-Kit:** Es la aproximación que los mantenedores consideran "la recomendada" para polyrepo.
- **Contratos explícitos:** CDD asegura que las interfaces entre servicios tengan un dueño claro y versionado.

❌ Desventajas:
- **Dispersión de specs (si no se gestiona bien):** Cada repo termina con su propio specs/, lo que puede llevar a la fragmentación que buscas evitar.
- **Requiere disciplina:** Necesitas un proceso claro para vincular las implementation specs al umbrella spec y mantener los contratos sincronizados.

#### 💎 Conclusión

La taxonomía de tres estrategias es correcta y completa. Cada una responde a una filosofía distinta sobre dónde debe vivir la "verdad" de las especificaciones. La elección depende de las prioridades:

- **Si buscas “máxima centralización” y usas Spec-Kit:** la Estrategia B con un Meta-Repo es la que mejor equilibra tus requisitos de no dispersar specs y mantener compatibilidad con /speckit.implement.
- **Si prefieres “seguir la recomendación oficial de los mantenedores de Spec-Kit”:** la Estrategia C es el camino, aceptando que cada repo tendrá su propia spec de implementación.
- **La Estrategia A es la más simple pero la que más dispersa el conocimiento, y probablemente no sea la adecuada para tu caso de microservicios con features verticales.**

La buena noticia es que puedes hibridar: por ejemplo, usar el Meta-Repo (B) como infraestructura, pero organizar los specs dentro de él siguiendo el patrón de umbrella spec + contratos (C) para mantener la disciplina de CDD.

