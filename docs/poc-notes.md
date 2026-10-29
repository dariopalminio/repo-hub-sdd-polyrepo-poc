
El patrón repo-service-sdd-polyrepo-poc/* + !repo-service-sdd-polyrepo-poc/.git/ en .gitignore permite que el hook multi-repo-sync detecte el repo hijo, pero hace que git status del hub lo muestre como untracked. Aceptable para la POC; en producción, evaluar si la extensión soporta targets explícitos para evitar el trade-off.

Si más adelante quieres explorar otra opción, revisa la documentación de multi-repo-sync para confirmar si soporta targets en .specify/init-options.json. Si lo soporta, es la solución más limpia: git status limpio y hook funcionando.

