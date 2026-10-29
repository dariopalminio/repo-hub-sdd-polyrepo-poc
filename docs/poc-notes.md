
El patrón repo-service-sdd-polyrepo-poc/* + !repo-service-sdd-polyrepo-poc/.git/ en .gitignore permite que el hook multi-repo-sync detecte el repo hijo, pero hace que git status del hub lo muestre como untracked. Aceptable para la POC; en producción, evaluar si la extensión soporta targets explícitos para evitar el trade-off.

Actualmente git status muestra el folder del repo servicio. Entonces alguien puede hacer git add ., aunque no sube el código del servicio al hub, pero sí crea una referencia rota que rompe el clonado para cualquiera que descargue el repositorio padre. La disciplina de usar git add <ruta-específica> (por ejemplo, git add specs/) es la forma correcta de evitar este problema y mantener la integridad del polyrepo.

Mover los repos hijos a una carpeta repos/ (o components/) no resuelve por sí solo el trade-off del .gitignore que discutimos, pero sí mejora la claridad conceptual del hub y hace que el git status sea auto-explicativo: un directorio repos/ untracked es evidente que contiene repos coordinados, mientras que un repo-service-sdd-polyrepo-poc/ untracked en la raíz parece un error.

Si más adelante quieres explorar otra opción, revisa la documentación de multi-repo-sync para confirmar si soporta targets en .specify/init-options.json. Si lo soporta, es la solución más limpia: git status limpio y hook funcionando.

