# POC Runbook Spec 002

Probar el flujo con una segunda spec

### 🥈 Ejercicio: `GET /greet/{name}` con parámetros de ruta (Path Parameters)
Esta opción es más sencilla que la anterior, pero prueba la capacidad del agente para definir y extraer parámetros dinámicos en la URL.

**Prompt para `/speckit.specify` (en el Hub):**
> *"Añadir un endpoint `GET /greet/{name}` en el repo servicio. Debe extraer el nombre de la ruta y devolver HTTP 200 con `{"greeting": "Hello, <name>!"}`. Si el nombre está vacío o contiene caracteres no alfanuméricos, devolver HTTP 400. El contrato OpenAPI debe reflejar el parámetro de ruta y las respuestas."*

**Qué valida de la POC:**
*   Cómo maneja el agente la lógica de enrutamiento (routing) condicional en `server.js` para capturar `/greet/juan` pero no `/greet/`.
*   La definición de `parameters` en OpenAPI.


1. Ejecutar /speckit-specify

/speckit-specify Añadir un endpoint `GET /greet/{name}` en el repo servicio. Debe extraer el nombre de la ruta y devolver HTTP 200 con `{"greeting": "Hello, <name>!"}`. Si el nombre está vacío o contiene caracteres no alfanuméricos, devolver HTTP 400. El contrato OpenAPI debe reflejar el parámetro de ruta y las respuestas. *Asegúrate de mantener la lógica existente de `GET /health` y el manejo de 404. El agente debe leer el `server.js` actual en `repos/repo-service-sdd-polyrepo-poc/src/` antes de proponer el nuevo código.

El hook before_specify de Git creó la rama 002-greet-endpoint en el hub.
Revisar la spec y aprobarla en review-spec.

```powershell
git add -- specs/002-greet-endpoint
git commit -m "docs(spec): specify greet endpoint"
```

2. Ejecutar /speckit-plan. Se generaron plan.md, research.md, data-model.md, contracts/openapi.yaml y quickstart.md.
El hook obligatorio after_plan analizó los repos afectados. El escáner no detectó el servicio porque repos/* está en el .gitignore del hub; se registró explícitamente en “Affected Repositories”, usando .meta.yaml.
Revisar el plan y aprobarlo en review-plan.

```powershell
git add -- specs/002-greet-endpoint
git commit -m "docs: plan greet endpoint"
```

3. Ejecutar /speckit-tasks. Se generaron cuatro tareas para las dos historias. El hook after_tasks creó y seleccionó 002-greet-endpoint en el repo servicio.
4. Ejecutar /speckit-analyze después de generar tareas. Encontró que el diseño incluía HTTP 400 para % malformado, pero la spec no lo decía. Se actualizó la spec y el quickstart para alinearlos.

```powershell
git add -- specs/002-greet-endpoint
git commit -m "docs: add greet endpoint planning artifacts"
```

5. Ejecutar /speckit-implement. El checklist de requisitos pasó; las cuatro tareas se completaron: contrato OpenAPI, ruta exitosa, validación 400 y pruebas manuales.

No hizo falta ejecutar /speckit-clarify porque no quedaron aclaraciones pendientes. /speckit-checklist tampoco se ejecutó como skill aparte: Specify creó el checklist incorporado.

Los ocho casos manuales pasaron —saludo, entradas inválidas, %ZZ, /health, ruta desconocida y método no soportado—, pero se probaron con Node 20.20.0. Como package.json exige Node >=22, repetiría el quickstart con Node 22 antes de cerrar el PR.

6. Validación:

npm start

Déjalo ejecutándose y, en otra terminal, prueba:
```powershell
curl.exe -i http://localhost:3000/greet/Ada
curl.exe -i http://localhost:3000/greet
curl.exe -i http://localhost:3000/greet/
curl.exe -i http://localhost:3000/greet/Ada-Smith
curl.exe -i http://localhost:3000/health
curl.exe -i http://localhost:3000/not-a-route
```


7. Git y PR de cierre

Los repos son independientes, así que los cambios se confirman por separado. Antes, revisa git status en ambos. En el hub, stagea solo los artefactos de la feature y deja fuera poc-runbook.md salvo que ese cambio también sea intencional:

```powershell
git add -- specs/002-greet-endpoint
git commit -m "docs: complete greet endpoint feature"
git push
```

En el repo servicio:

```powershell
git -C repos/repo-service-sdd-polyrepo-poc add -- src/server.js
git -C repos/repo-service-sdd-polyrepo-poc commit -m "feat: add greet endpoint"
git -C repos/repo-service-sdd-polyrepo-poc push -u origin 002-greet-endpoint
```

Después abriría un PR en cada repositorio y los enlazaría entre sí. Los hooks de commit automáticos estaban deshabilitados; no hicimos commits ni push.


Sí, especialmente si el Product Owner define el alcance y otra persona hace el diseño técnico. Un PR de la spec permite revisar escenarios, criterios de aceptación y límites antes de invertir tiempo en el plan.

En este workflow abriría un Draft PR del hub después de /speckit-specify, con spec.md y el checklist. Esperaría la aprobación del PO y recién entonces ejecutaría /speckit-plan. No hace falta cerrar ese PR: se puede continuar en la misma rama y añadir después plan, tareas y artefactos de diseño. Así el PR evoluciona con la feature, pero la aprobación de la spec sigue siendo una puerta explícita.

La implementación en el repo servicio tendrá su PR separado, porque es otro repositorio.


Para una segunda historia, lo ideal es elegir un ejercicio que **ponga a prueba una capacidad nueva del flujo SDD** que la primera historia (el simple `GET /health`) no exigió. La historia anterior era puramente "read-only" y sin parámetros.

Aquí tienes tres opciones ordenadas por nivel de desafío. Te recomiendo la **Opción 1**, porque obliga al agente a modificar el contrato (OpenAPI) para añadir un *Request Body* y lógica de validación, algo que no hizo en la primera.

---

## 💡 Próximos ejercicio


### 🥇 Opción 1 (Recomendada): `POST /echo` con validación de payload
Esta historia es perfecta porque introduce el manejo de estados HTTP (400, 200), lectura de streams de datos (body) y la definición de un *schema* de entrada en OpenAPI.

**Prompt para `/speckit.specify` (en el Hub):**
> *"Añadir un endpoint `POST /echo` en el repo servicio. Debe recibir un JSON con la estructura `{"message": "string"}`. Si el JSON es válido y contiene el campo `message`, devolver HTTP 200 con `{"echo": "<mensaje_recibido>", "length": <longitud_del_mensaje>}`. Si falta el campo o el JSON es inválido, devolver HTTP 400 con un objeto de error. El contrato OpenAPI debe definir explícitamente el requestBody y las respuestas 200 y 400."*

**Qué valida de la POC:**
*   El agente debe actualizar el `openapi.yaml` en el hub con un `requestBody`.
*   Debe implementar la lógica para parsear el body en Node.js puro (sin usar frameworks) usando `req.on('data')`.
*   Debe mapear los errores de validación al código HTTP correcto.

---

### 🥉 Opción 2: `GET /info` con metadatos del sistema
Esta es una historia de refactorización simple. No introduce interacción compleja, pero prueba la capacidad del agente para leer variables de entorno y añadir nuevos campos al contrato.

**Prompt para `/speckit.specify` (en el Hub):**
> *"Modificar el endpoint `GET /health` existente para que devuelva también la versión del servicio y el tiempo de actividad. El nuevo contrato debe ser `{"status":"ok", "version":"1.0.0", "uptime": <segundos_desde_que_inició>}`. La versión debe leerse desde la variable de entorno `APP_VERSION` (con fallback a `"1.0.0"`)."*

**Qué valida de la POC:**
*   La capacidad del agente de **modificar** un contrato existente en lugar de crear uno nuevo (actualización de `openapi.yaml`).
*   El manejo de variables de entorno en el repo servicio.

---
