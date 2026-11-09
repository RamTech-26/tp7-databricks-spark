# TP7 – Introducción a Databricks y Apache Spark

Trabajo práctico de **Análisis de Datos en la Nube** (ITU – UNCuyo).
Notebook en PySpark sobre el dataset **Iris**, ejecutado en **Databricks Free Edition** (cómputo serverless).

## Contenido del repositorio

| Archivo | Qué es |
|---|---|
| `notebook-ciencia_de_datos.ipynb` | Notebook con todo el trabajo |
| `notebook-ciencia_de_datos.html` | El mismo notebook exportado **con los resultados** de cada celda |
| `capturas/` | Capturas del workspace, el cómputo, el Volume, la tabla Delta y los gráficos |
| `README.md` | Este archivo |

> El `.ipynb` que se guarda desde Git puede no traer las salidas de las celdas.
> Para ver los resultados sin ejecutar nada, abrir el `.html`.

## Qué hace el notebook

1. Lee `iris.csv` con Spark desde un **Volume** de Unity Catalog.
2. Revisa el esquema y las estadísticas básicas (`printSchema`, `describe`).
3. Consulta con **Spark SQL** (cantidad de flores por especie).
4. Limpia: `dropDuplicates` + `dropna` (150 → 149 filas).
5. Agrega la columna calculada `petal_ratio` (largo / ancho del pétalo).
6. Filtra la especie *virginica* y calcula promedios por especie (`groupBy` + `agg`).
7. Hace un **join** con una tabla auxiliar de especies (código y color).
8. Guarda el resultado como tabla **Delta** (`iris_transformada`) y la valida.
9. Prueba el **versionado de Delta Lake**: `DELETE`, `VERSION AS OF 0`, `DESCRIBE HISTORY` y `RESTORE`.
10. Grafica con matplotlib (convirtiendo con `toPandas()`).

## Resultados principales

- 3 especies con 50 muestras cada una; al eliminar un duplicado exacto quedan 149 filas (virginica pasa a 49).
- *Virginica* tiene los pétalos más largos (promedio ≈ 5,56 cm).
- El ratio largo/ancho separa bien a *setosa* (≈ 6,9) de *versicolor* (≈ 3,2) y *virginica* (≈ 2,8).
- Delta Lake permitió borrar filas, leer la versión original y restaurarla.

## Cómo reproducirlo (paso a paso)

El notebook lee el CSV desde un Volume, así que **hay que crearlo y subir el archivo antes de ejecutar**.
Cada persona lo hace en su propio workspace.

### 1. Descargar el dataset

Descargar el CSV de Iris desde seaborn-data:

<https://raw.githubusercontent.com/mwaskom/seaborn-data/master/iris.csv>

(Abrir el enlace, **Guardar como…** y conservar el nombre `iris.csv`.)
Debe tener 150 filas más la cabecera, con las columnas
`sepal_length, sepal_width, petal_length, petal_width, species`.

### 2. Crear el Volume

En Databricks: **Catálogo** → `workspace` → `default` → **Crear** → **Volume**.

- Tipo: **Administrado** (*managed*)
- Nombre: `datos`

La ruta queda: `/Volumes/workspace/default/datos/`

### 3. Subir el CSV al Volume

Entrar al Volume `datos` → **Subir a este volumen** → elegir `iris.csv`.

Verificar que la ruta final sea exactamente:

```
/Volumes/workspace/default/datos/iris.csv
```

### 4. Importar el notebook

Opción A (desde este repo): **Workspace** → **Crear** → **Git folder** → pegar la URL HTTPS del repositorio.

Opción B (archivo suelto): **Workspace** → **Importar** → subir el `.ipynb`.

### 5. Conectar el cómputo y ejecutar

Arriba a la derecha del notebook, elegir **Serverless** y luego **Ejecutar todo**.
No hace falta crear ningún clúster.

Si las celdas corren sin errores, la tabla `workspace.default.iris_transformada`
aparece en **Catálogo → workspace → default**.

## Problemas frecuentes

| Síntoma | Causa probable | Solución |
|---|---|---|
| `PATH_NOT_FOUND` o `FileNotFound` al leer el CSV | El Volume o el archivo no existen, o el nombre difiere | Revisar paso 2 y 3: Volume `datos`, archivo `iris.csv` |
| Dice que no encuentra el catálogo/esquema | Cambió el nombre de `workspace` o `default` | Ajustar la ruta en la primera celda de lectura |
| `Table or view already exists` / conflictos al reejecutar | Quedó una tabla de una corrida anterior | Ejecutar `DROP TABLE IF EXISTS iris_transformada` y volver a correr |
| `DELETE`/`RESTORE` fallan | La tabla no es Delta | Reejecutar la celda que la crea con `.format("delta")` |

## Autoras

Agustina y Romina – cátedra de Análisis de Datos en la Nube (prof. Luis A. Chiaramonte).