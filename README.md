### Reto Avanzado: Creador de Acciones

> Tu reto es crear una "Acción Compuesta" (Composite Action) que analice archivos de log.

> 1. Crea una nueva rama: `git checkout -b solucion-tu-nombre`.
> 2. Crea un nuevo directorio: `_actions/log-analyzer`.
> 3. Dentro de ese directorio, crea tu archivo `action.yml`.

> **Requisitos del `action.yml`:**

> * Debe tener un `input` llamado `log-file`.
> * Debe tener un `output` llamado `error-count`.
> * En la sección `runs`, debe usar `shell: bash`.
> * El script debe:
> 1. Leer el archivo de la variable de entrada (`${{ inputs.log-file }}`).
> 2. Contar cuántas líneas contienen la palabra "ERROR".
> 3. Establecer ese conteo como la salida `error-count`.

> 4. Haz commit, push y abre una Pull Request. El validador automático te dirá si funciona.
