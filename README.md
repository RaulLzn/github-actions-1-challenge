
# Reto Avanzado: Creador de Acciones Reutilizables

¡Bienvenido al desafío\! Este no es un reto de "Hola Mundo", sino un ejercicio de DevOps del mundo real.

## El Porqué: ¿Por qué hacemos esto?

En proyectos grandes, a menudo repetimos los mismos pasos en diferentes flujos de trabajo (Workflows). Por ejemplo, podemos tener un workflow para `push`, otro para `pull_request` y otro para `release`, y todos necesitan *contar errores en un log*.

En lugar de copiar y pegar el mismo script de `bash` en 10 archivos `.yml` diferentes (lo cual es malo para el mantenimiento), podemos **empaquetar esa lógica en una "Acción Compuesta" (Composite Action)**.

Esto nos da una sola fuente de verdad: creamos una acción personalizada que vive en nuestro repositorio y la llamamos con una sola línea (`uses: ./.github/actions/mi-accion`).

**Tu misión es construir una de estas acciones reutilizables.**

-----

## Tu Misión: ¿Qué vas a construir?

Tu reto es crear una "Acción Compuesta" que analice un archivo de log y cuente cuántas líneas contienen la palabra "ERROR".

**Deberás crear UN solo archivo:**

  * **Ruta:** `_actions/log-analyzer/action.yml`

**Este `action.yml` debe cumplir con estos requisitos técnicos:**

1.  **Entrada (Input):** Debe aceptar una entrada (input) llamada `log-file`. El validador usará esto para decirte *qué* archivo analizar.
2.  **Salida (Output):** Debe proveer una salida (output) llamada `error-count`. El validador usará esto para leer *tu respuesta*.
3.  **Lógica (Runs):** Debe usar `bash` para:
      * Leer el archivo que se le pasa en `log-file`.
      * Contar el número de líneas que contienen "ERROR".
      * Exponer ese número final a través de la salida `error-count`.

-----

## El Cómo: Guía Paso a Paso

Sigue estas instrucciones al pie de la letra.

### Paso 1: Prepara tu Entorno (El Flujo de Git)

No trabajes nunca directamente en `main`.

1.  Asegúrate de tener la última versión del repositorio:
    ```bash
    git checkout main
    git pull
    ```
2.  Crea tu propia rama para trabajar. (¡Usa tu nombre/usuario\!)
    ```bash
    git checkout -b solucion-minombre
    ```

### Paso 2: Crea la Estructura de la Acción

GitHub Actions busca acciones en rutas específicas.

1.  Crea los directorios necesarios. (El `-p` es para que cree `_actions` y `log-analyzer` a la vez):
    ```bash
    mkdir -p _actions/log-analyzer
    ```
2.  Crea el archivo `action.yml` donde vivirás tu solución:
    ```bash
    touch _actions/log-analyzer/action.yml
    ```

### Paso 3: El Corazón del Reto (El `action.yml`)

Abre tu archivo `_actions/log-analyzer/action.yml` y empieza a construirlo. Te explicamos cada bloque:

#### A. Metadatos (Lo básico)

Dale un nombre y descripción.

```yaml
name: 'Analizador de Logs'
description: 'Cuenta líneas de ERROR en un archivo de log.'
```

#### B. Entradas (Inputs)

Tu acción necesita saber *qué* archivo analizar. Para eso usamos `inputs`.

```yaml
# ... (debajo de la descripción)
inputs:
  log-file: # Este es el ID que usamos (el reto pide 'log-file')
    description: 'Ruta al archivo de log para analizar'
    required: true # El validador siempre te lo proporcionará
```

#### C. Salidas (Outputs)

Tu acción necesita *reportar* su resultado. Para eso usamos `outputs`.

```yaml
# ... (debajo de los inputs)
outputs:
  error-count: # Este es el ID que el validador leerá
    description: 'Número total de errores encontrados'
    # Esta línea es CLAVE: le dice a la salida "error-count"
    # que su valor vendrá del paso con ID "count-step" y su salida "count".
    value: ${{ steps.count-step.outputs.count }}
```

#### D. La Lógica (Runs)

Aquí es donde ocurre la magia. Le decimos a GitHub que esto es una acción `composite` que ejecutará unos pasos de `bash`.

```yaml
# ... (debajo de los outputs)
runs:
  using: 'composite' # ¡Importante! Define el tipo de acción
  steps:
    # Este es el único paso que necesitamos
    - id: count-step # ¡Este ID debe coincidir con el 'value' de arriba!
      run: |
        echo "Analizando el archivo: ${{ inputs.log-file }}"
        
        # 1. Contamos las líneas con "ERROR"
        #    Usamos '|| true' para que el script no falle si grep no encuentra nada
        COUNT=$(grep -c "ERROR" ${{ inputs.log-file }} || true)
        
        echo "Errores encontrados: $COUNT"
        
        # 2. Esta es la línea MÁS IMPORTANTE
        #    Así es como un script de bash se comunica con las 
        #    'outputs' de un 'step' en GitHub Actions.
        echo "count=$COUNT" >> $GITHUB_OUTPUT
        
      shell: bash # Le decimos que use bash
```

> **¡Pista Crucial\!**
> La línea `echo "count=$COUNT" >> $GITHUB_OUTPUT` es el "puente" entre tu script de `bash` y la salida (`output`) de la Acción.
>
>   * `count=` coincide con el nombre de la salida que definiste en `steps.count-step.outputs.count`.
>   * `$GITHUB_OUTPUT` es una variable de entorno especial que GitHub te da para escribir resultados.

### Paso 4: Entrega y Validación

1.  Una vez que tu `action.yml` esté listo, guárdalo y haz commit:
    ```bash
    git add _actions/log-analyzer/action.yml
    git commit -m "Solución: Creando mi acción composite log-analyzer"
    ```
2.  Sube tu rama a GitHub:
    ```bash
    git push --set-upstream origin solucion-minombre
    ```
3.  Ve a GitHub, busca tu rama y **abre una Pull Request (PR)** contra la rama `main`.
4.  ¡Espera\! En el momento en que abras la PR, nuestro validador (`.github/workflows/validator.yml`) se disparará.
5.  Ve a la pestaña "Checks" de tu PR.

#### ¿Cómo funciona el validador?

El validador hará esto:

1.  Obtendrá tu código.
2.  Llamará a **tu** acción (`uses: ./_actions/log-analyzer`).
3.  Le pasará nuestro log de prueba (`log-file: 'data/log-produccion.txt'`).
4.  Leerá la salida de tu acción (`${{ steps.student-action.outputs.error-count }}`).
5.  Comparará tu resultado con la respuesta correcta (que es `3`).

<!-- end list -->

  * **Si tu PR muestra un ✅ verde:** ¡Felicidades\! Lo lograste.
  * **Si tu PR muestra una ❌ roja:** No te preocupes. Haz clic en "Details" para leer los logs, ver qué falló, arreglarlo en tu rama local, hacer `commit` y `push` de nuevo. La PR se actualizará y volverá a correr la prueba automáticamente.

  Prueba 3
