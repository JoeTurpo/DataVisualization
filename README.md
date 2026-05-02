Proyecto: Análisis y limpieza del dataset SIDPOL
===============================================

Resumen
-------
Este repositorio contiene los datos y el notebook usados para consolidar, limpiar y preparar un dataset llamado SIDPOL para análisis de policía, comisarías y población distrital. El objetivo es reproducir el flujo: carga, limpieza, transformaciones, QA y exportación de archivos finales.

Estructura del proyecto
-----------------------
- [data.ipynb](data.ipynb): Notebook principal con todo el análisis y pasos de limpieza.
- poblacion_distrital_2018_2026_larga.csv: Fuente de población por distrito.
- policias_distrito_ref_2025.csv: Referencia de policías por distrito (2025).
- qa_merge_comisarias_2020_por_nombre.csv: Resultado intermedio de QA para comisarías.
- qa_merge_comisarias_2020.csv: Otro resultado intermedio de QA para comisarías.
- qa_merge_policias_ref_2025.csv: Resultado intermedio de QA para policías.
- revision_dataset_final_unico.csv: Export de revisión final.
- sidpol_consolidado_final_limpio*.csv: Varias versiones del dataset consolidado y limpiado.
- salida_revision/: Carpeta con resúmenes y salidas de revisión.

Requisitos
----------
Se recomienda crear un entorno con Python 3.10+ e instalar las dependencias principales usadas en el notebook:

```bash
python -m venv .venv
.venv\Scripts\activate    # Windows
pip install pandas numpy jupyterlab openpyxl matplotlib seaborn
```

Cómo reproducir el flujo (resumen)
---------------------------------
1. Abrir `data.ipynb` en JupyterLab o Jupyter Notebook.
2. Ejecutar las celdas en orden: carga de archivos, limpieza, merges, QA y guardado de resultados.
3. Revisar la carpeta `salida_revision/` para resúmenes y archivos de control.

Descripción paso a paso (explicación técnica)
---------------------------------------------
1) Carga de datos
- Se cargan los CSV principales con `pandas.read_csv()` usando `encoding='utf-8'` o `latin1` cuando es necesario.
- Verificación inicial con `df.head()`, `df.info()` y `df.describe()`.

2) Inspección y diagnóstico
- Se evalúan columnas faltantes (`df.isna().sum()`), tipos erróneos y valores atípicos.
- Se generan tablas de conteo para columnas categóricas y resúmenes por distrito.

3) Limpieza aplicada (lo que hicimos)
- Normalización de nombres: aplicar `str.strip().str.lower()` y correcciones manuales para nombres de comisarías y distritos para poder hacer merges confiables.
- Eliminación de duplicados: `df.drop_duplicates()` usando subset de columnas clave (por ejemplo, identificador de registro, nombre y fecha).
- Manejo de valores faltantes:
  - Para columnas numéricas críticas se rellenó con 0 o con el valor interpolado según contexto.
  - Para categorías, se etiquetaron como "desconocido" si no era posible inferir.
- Cast de tipos: asegurar `datetime` para fechas (`pd.to_datetime()`), `int` para contadores y `category` para variables con pocas categorías.
- Corrección de codificaciones y caracteres especiales: normalizar acentos y guiones.

4) Uniones y consolidación
- Merge entre tablas de policías, comisarías y población mediante claves de distrito y nombres estandarizados (`pd.merge(left, right, how='left', on=...)`).
- Resolución de conflictos: cuando hay múltiples referencias, se priorizó la fuente más reciente o la que tiene QA disponible.

5) QA y validación
- Se crearon archivos `qa_merge_*` que contienen checks post-merge: conteos por distrito, filas sin match, y estadísticas básicas.
- Se registraron discrepancias y se corrigieron manualmente las reglas de mapeo cuando fue necesario.

6) Exportación final
- Se generaron múltiples versiones de salida `sidpol_consolidado_final_limpio.csv` (v2, v3, v4, v5) para mantener trazabilidad.
- El archivo `revision_dataset_final_unico.csv` contiene la versión consolidada y lista para análisis.

Secciones clave del README para cada transformación
---------------------------------------------------
- Carga: indicar la celda o sección del notebook donde se realiza `pd.read_csv` y las opciones de lectura.
- Limpieza: enumerar las transformaciones aplicadas (normalización, deduplicación, imputación), con fragmentos de código ejemplo.
- Merge: detallar las claves usadas y ejemplos de `pd.merge`.
- QA: explicar las comprobaciones y cómo interpretar `salida_revision/resumen_columnas_por_hoja.csv`.

Fragmentos de código útiles
--------------------------
Estandarizar nombres:

```python
def clean_name(s):
    if pd.isna(s):
        return "desconocido"
    s = str(s).strip().lower()
    s = s.replace('\u00a0', ' ')
    # aplicar correcciones específicas
    s = s.replace('comisaria', 'comisaría')
    return s

df['nombre_clean'] = df['nombre'].apply(clean_name)
```

Eliminar duplicados por columnas clave:

```python
df = df.drop_duplicates(subset=['id_registro', 'nombre_clean', 'fecha'])
```

Merge con población por distrito:

```python
pop = pd.read_csv('poblacion_distrital_2018_2026_larga.csv')
pop['distrito_clean'] = pop['distrito'].apply(clean_name)
df = df.merge(pop[['distrito_clean','poblacion']], left_on='distrito_clean', right_on='distrito_clean', how='left')
```

Recomendaciones para reproducir exactamente
-------------------------------------------
- Ejecutar el notebook en orden sin saltarse celdas.
- Si falta algún paquete, instalarlo con `pip install`.
- Revisar `salida_revision/` para ver logs y resúmenes de validación.

Notas sobre versiones y trazabilidad
-----------------------------------
- Mantuvimos versiones numeradas del dataset para poder volver a estados previos (`sidpol_consolidado_final_limpio_v2.csv`, `_v3.csv`, etc.).
- Cada versión incluye un hash simple (fecha + conteo de filas) en su nombre o en metadatos dentro del notebook.

Próximos pasos sugeridos
-----------------------
- Añadir un `requirements.txt` o `environment.yml` para fijar entornos.
- Automatizar el flujo con un script `run_pipeline.py` que ejecute las celdas importantes del notebook o replique los pasos en scripts Python.
- Incorporar tests de validación (p. ej. comprobar que la suma de policías por distrito coincide con la referencia) y guardarlos en `salida_revision/`.

Contacto y ayuda
----------------
Si quieres que detallen más cada celda del `data.ipynb` (por ejemplo, con referencias a números de celda y extractos exactos), puedo:

- Generar una sección por cada bloque/celda del notebook con el fragmento de código y su explicación.
- Crear un `requirements.txt` y un script reproducible.

Archivo creado: [README.md](README.md)
