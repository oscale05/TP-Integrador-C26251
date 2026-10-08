# Reporte técnico — Trabajo Práctico Integrado

**Comisión C26251 · Integrante: Franco Oscar Alejandro**
Repositorio: https://github.com/oscale05/TP-Integrador-C26251
Archivos: `TP_Integrador.ipynb` (62 celdas, ejecutado sin errores) · `datos/` · `anexos/` · `README.md`

---

## 0. Criterio general del trabajo

El TP pedía dos etapas: **recopilar/preparar** y **preprocesar/limpiar**. El hilo conductor que se aplicó
en todo el notebook es:

1. **Nunca se modifica el dato original.** Los CSV viven en `datos/` sin tocar y cada transformación se
   hace sobre copias (`ventas.copy()`). Así el notebook puede re-ejecutarse desde cero con el mismo
   resultado y el evaluador puede auditar el antes/después.
2. **Cada celda de código tiene su bloque de texto.** El TP exige "bloques de texto y código" que documenten
   obtención, limpieza y hallazgos; el texto explica el *qué* y el *porqué*, el código es la evidencia.
3. **Toda decisión se justifica con datos, no con gusto personal.** Cuando hubo que elegir (estructura,
   criterio de corte, forma de unir tablas), se compararon alternativas y se mostró la comparación en el
   notebook para que la elección sea auditable.

---

## 1. Preparación del entorno

| Paso | Qué se hizo | Por qué |
|---|---|---|
| Repositorio Git | `git init`, config `user.name=oscale05`, `user.email=oscar.alejandrofranco@gmail.com`, remoto `TP-Integrador-C26251` | El TP se entrega como carpeta compartida; el repo deja historial de versiones y respaldo. El `.gitignore` descarta `.ipynb_checkpoints`, `.DS_Store` y el lock `~$Pre.docx` (artefactos de Word), que no son parte del entregable |
| Organización de carpetas | `datos/` (originales), `anexos/` (gráficos), notebook en la raíz | Responde al requisito de entrega: "sets de datos originales" + notebook + Anexo con los archivos adicionales |
| Celda inicial de librerías | `!pip install -q pandas numpy matplotlib` + imports + opciones de visualización, **antes de la Etapa 1** | Deja el notebook autocontenido: en Colab ya vienen instaladas (el pip sólo lo confirma) y en cualquier otra máquina instala lo mínimo. Poder importar todo en un solo lugar evita repetir imports dentro de las actividades |
| Carga con búsqueda de rutas | Función `buscar_archivo()` que recorre `./datos`, `./`, la carpeta de Drive montada y `/content` | El notebook debe correr igual en Colab (con Drive) y en local. Si no encuentra los archivos, lanza un error con las rutas buscadas en lugar de fallar en la primera celda |

**Librerías elegidas y por qué:** `pandas` (estructura de datos y todo el análisis), `numpy` (operaciones
numéricas y generación aleatoria), `matplotlib` (gráficos exigidos para comunicar resultados), `csv` y
`pathlib` de la biblioteca estándar (para el script *sin pandas* de la Actividad 2 y para las rutas).
No se usó ninguna librería externa al TP: menos dependencias = menos que puede romperse al re-ejecutar.

---

## 2. Etapa 1 — Recopilación y preparación

### 2.1 Actividad 1 · Cargar los sets como DataFrames
**Qué:** `pd.read_csv` de los tres archivos, con impresión de forma, dtypes y primeras filas.
**Por qué:** es la verificación mínima antes de analizar: confirma que el archivo se leyó completo, que
las columnas tienen el nombre esperado y revela los tipos crudos. Ahí se detectó lo que condicionó todo
lo demás: `precio` y las fechas vienen como **texto**, y `cantidad` se carga como `float64`.

### 2.2 Actividad 2 · Script de ventas mensuales sin pandas
**Qué:** lectura con `csv.DictReader`, variables y operadores (`precio * cantidad`, acumulación),
`if/for`, diccionario como acumulador por mes, desempaquetado de la fecha (`dia, mes, anio = split("/")`),
impresión de tabla mensual, total, promedio, mejor y peor mes.

**Por qué se hizo a mano antes de Pandas:**
* El objetivo de la clase es demostrar lógica de Python base; con pandas la misma tarea son 2 líneas y no
  mostraría el razonamiento.
* Al hacerlo manual se entiende *qué* calcula pandas por debajo (agrupar y sumar), lo que hace pedagógico
  el paso siguiente.

**Hallazgo documentado:** total crudo **ARS 1.483.042,93** sobre 3.033 ventas; máximo abril (144.380),
mínimo junio (108.480). Ese total **incluye los 35 duplicados**; después de limpiar da
**ARS 1.467.093,52**, es decir los duplicados inflaban los ingresos en **ARS 15.949,41 (1,1 %)**.
Se dejó escrito en el notebook porque es la justificación empírica de por qué limpiar no es opcional.

**Limitación declarada:** el script sólo calcula totales mensuales; para categoría o producto habría que
reescribirlo. Esa es la razón explícita de pasar a Pandas.

### 2.3 Actividad 3 · Estructuras de datos: ¿lista o diccionario?
**Qué:** se implementaron las dos formas (lista de diccionarios y diccionario indexado por `id_venta`),
se midió el tiempo de búsqueda y se compararon en una tabla criterio por criterio.

**Decisión: diccionario indexado por `id_venta`.** Justificación:

| Criterio | Lista | Dict | Quién gana |
|---|---|---|---|
| Recuperar una venta por su id | recorrido **O(n)** | acceso **O(1)** | dict |
| Claves duplicadas | se guardan sin aviso | imposibles por definición | dict |
| Acceso frecuente "traer todo y agregar" | natural | requiere convertir | lista |

El patrón dominante en un sistema de ventas es *consultar por comprobante*, por eso gana el dict. Se aclaró
explícitamente que **la decisión depende del patrón de acceso, no de la estructura en abstracto**, y que
para el análisis masivo de este TP lo que conviene es Pandas (que internamente trabaja con arreglos de
registros, es decir, la opción "lista").

### 2.4 Actividad 4 · Análisis exploratorio (EDA)
**Qué:** `shape`, `isna()`, `duplicated()`, `describe(include="all")`, `value_counts()` de categorías,
canales y ciudades, rango de fechas, conteo mensual, correlación edad–ingresos.

**Por qué ese orden:** primero el contorno (tamaño, tipos, nulos), luego la distribución de las variables
categóricas, luego el eje temporal. Es el recorrido estándar que permite detectar sorpresas antes de
transformar: la campaña por producto (30 × 3), las categorías casi empatadas, y que **no hay columna de
cliente en ventas** (decisión que aparece en la Etapa 2).

### 2.5 Actividad 5 · Calidad de datos
**Qué:** diagnóstico de los tres datasets (nulos, duplicados exactos, ids repetidos, formato del precio,
formato de fecha, espacios sobrantes) y tabla de "Estado inicial" documentada.

**Resultado verificado:**

| Dataset | Filas | Nulos | Duplicados | Problemas de formato |
|---|---:|---:|---:|---|
| `ventas.csv` | 3.035 | 2 (`precio`, `cantidad`, ids 627 y 2171) | 35 exactos (35 `id_venta` repetidos) | `precio` = texto con `$`; fechas `dd/mm/aaaa` |
| `clientes.csv` | 567 | 0 | 0 | ninguno |
| `marketing.csv` | 90 | 0 | 0 | fechas `dd/mm/aaaa` |

**Por qué se documentó antes de limpiar:** el TP pide "documentar el estado inicial de los datos". Dejar el
número exacto de problemas permite después medir cuánto se limpió y qué costó (2.998 filas finales).
Se aclaró que los apóstrofos de los nombres (`O'Regan`) **son válidos**, no "caracteres no deseados":
confundirlos habría introducido una limpieza destructiva e injustificada.

---

## 3. Etapa 2 — Preprocesamiento y limpieza

### 3.1 Actividad 1 · Limpieza (paso a paso con contadores)
Se hizo en 7 pasos, cada uno con su delta de filas impreso:

| Paso | Acción | Por qué |
|---|---|---|
| 1 | `drop_duplicates()` → **−35 filas** | Son copias idénticas: duplicar una venta no agrega información, sólo infla ingresos y conteos (medido: 1,1 %) |
| 2 | `dropna(subset=["precio","cantidad"])` → **−2 filas** | Sin precio ni cantidad no se puede calcular el ingreso. Se descartó imputar: son 2 de 3.035 (0,07 %), no hay variable de la cual inferirlos y cualquier valor inventado sesgaría el total |
| 3 | `.str.strip()` → 0 cambios | Se ejecutó igualmente porque era una hipótesis razonable; el chequeo previo mostró que no había espacios. Documentar "0 cambios" también es un resultado |
| 4 | `.str.replace("$","").astype(float)` | **Carácter no deseado** exigido por el enunciado: el `$` impide cualquier cálculo. Sólo se quita el símbolo, **no se convierte la moneda** (ver §5) |
| 5 | `.astype(int)` en `cantidad` | El tipo era `float64` únicamente por culpa de los NaN; las cantidades son enteros |
| 6 | `to_datetime(format="%d/%m/%Y")` + columna `mes` | El formato es argentino (día primero); declararlo explícitamente evita que pandas interprete 02/01 como 2 de enero *o* 1 de febrero según el default. La columna `mes` habilita la serie temporal |
| 7 | `ingreso = precio × cantidad` | Métrica base de todas las actividades siguientes |

**Resultado:** 3.035 → **2.998 filas** (−1,2 %), sin nulos, sin duplicados, dtypes correctos.
La verificación post-limpieza (celda siguiente) re-imprime nulos, duplicados, tipos y rango de fechas:
la limpieza se prueba, no se da por hecha.

### 3.2 Actividad 2 · Transformación y filtro de alto rendimiento
**Métrica nueva:** `ingreso` por línea y su agregación por producto.

**Criterio elegido: ingreso total por producto ≥ percentil 75 (P75 = ARS 51.092,96).**

Justificación detallada (tal como queda en el notebook):

1. **Relativo al propio dataset.** No es un número inventado: si el negocio crece, el corte se mueve solo.
2. **Robusto a outliers.** La media se tira con los estrella (Lámpara de mesa 82.276 vs. Candelabro 11.128,
   casi 8×); el percentil sólo mira posición en el ranking.
3. **Equivalente a la intuición de "alto rendimiento".** P75 = cuartil superior = top 25 % del catálogo
   (regla de Pareto adaptada: 8 de 30 productos).
4. **Punto de equilibrio probado con los tres cortes habituales:**

| Corte | Umbral | Productos | % de ingresos | Problema |
|---|---:|---:|---:|---|
| P50 | 48.140 | 15 de 30 | 57,9 % | deja pasar la mitad: no discrimina |
| **P75** | **51.093** | **8 de 30** | **34,1 %** | cuartil estricto y accionable |
| P90 | 60.903 | 3 de 30 | 15,6 % | descarta productos buenos |

5. **Auditable:** cualquier revisor recalcula `quantile(0.75)` y verifica el corte; un umbral "a ojo" no.

**Criterios descartados y por qué:** *cantidad vendida ≥ X* ignora el precio (vender mucho y barato no es
alto ingreso); *corte sobre el total de categoría* desbalancearía tres categorías que hoy están empatadas.

**Resultado:** 8 productos (Lámpara de mesa, Auriculares, Microondas, Cafetera, Cuadro decorativo,
Smartphone, Secadora, Jarrón decorativo) generan **ARS 500.298,53 = 34,1 %** de los ingresos;
la tabla filtrada queda en 972 filas (32,4 % de las ventas).

### 3.3 Actividad 3 · Agregación por categoría
**Qué:** `groupby("categoria")` con transacciones, unidades, ingresos, ticket promedio y participación;
más el top de productos por categoría y tres gráficos.

**Por qué esas métricas:** ingresos (tamaño), unidades (volumen), transacciones (frecuencia) y **ticket
promedio** (precio efectivo) son los cuatro números que permiten decir *por qué* una categoría gana, no
sólo *cuánto* gana.

**Hallazgo y su lectura:** Electrodomésticos 505.300 (34,4 %), Electrónica 482.578 (32,9 %), Decoración
479.216 (32,7 %). Las transacciones están empatadas (1.000 / 998 / 1.000), así que **la diferencia no
viene del volumen sino del ticket** (505,30 vs. 479,22 → +5,4 %). Esa lectura es la que sostiene la
recomendación final (subir ticket en Decoración en lugar de multiplicar ventas).

### 3.4 Actividad 4 · Integración `ventas` × `marketing`
Primero se muestra el **error típico**, después las dos formas correctas. Es deliberado: el TP pide
integrar, y la forma obvia de integrar acá produce un número falso.

**Forma 0 (mostrada como contraejemplo): `merge` directo por `producto`**
* 2.998 → 8.994 filas (**×3**) porque cada producto tiene 3 campañas (relación 1‑a‑muchos).
* Los tres canales quedan con **exactamente los mismos ingresos**: el total se triplicó.
* Se imprime en el notebook para que la trampa sea visible y no se repita.

**Forma 1 (elegida para el análisis por producto): agregar y después unir**
* `groupby("producto")` de ventas y de marketing, y recién ahí `merge` → 30 filas, sin duplicación.
* Da el ROAS por producto (ingreso ÷ costo): promedio 3.322, máximo Lámpara de mesa 5.165, mínimo
  Candelabro 760. Es la comparación correcta de eficiencia publicitaria entre productos.

**Forma 2 (elegida para el análisis por canal): unión por ventana temporal**
* Cada venta se contabiliza sólo si su fecha cae dentro de `[fecha_inicio, fecha_fin]` de la campaña.
* Respeta el tiempo: no atribuye ventas que ocurrieron antes o después de la campaña.
* Resultado: **Email ROAS 1.048 > TV 944 > RRSS 831**; sólo **28,4 %** de los ingresos caen dentro de
  alguna ventana y 2 de 90 campañas no tuvieron ventas.

**Por qué esta doble vía y no una sola:** producto y campaña tienen granularidad distinta (producto =
todo el año; campaña = 38 días promedio). Agregar a nivel producto para el ROAS anual y usar ventana
temporal para el atributo mensual evita mezclar escalas.

**Limitaciones declaradas (obligatorias para no sobre-vender el resultado):**
1. No hay id de cliente ni de canal en la venta → la atribución es **temporal, no causal**.
2. Los 3 canales están activos al mismo tiempo para cada producto → la comparación entre canales es
   indicativa, no un experimento controlado.
3. Costos e ingresos están en escalas distintas → el ROAS sirve para **comparar canales entre sí**.

---

## 4. Análisis complementario de `clientes.csv`

**Decisión: se analiza en forma independiente, sin cruzar con ventas.**

Justificación, en tres verificaciones que quedan impresas en el notebook:

1. **No existe llave de unión.** `clientes.csv` tiene `id_cliente, nombre, edad, ciudad, ingresos`;
   `ventas.csv` tiene `id_venta, producto, categoria, precio, cantidad, fecha, mes, ingreso`.
   Columnas en común: **lista vacía**. No hay `id_cliente`, ni ciudad, ni ningún dato geográfico en la venta.
2. **Cualquier unión obligaría a inventar la relación.** Un cruce por asignación aleatoria *se puede
   programar*, pero mide lo que nosotros fabricamos: el "ingreso por ciudad" resultante no sería más que
   el **número de clientes por ciudad** repartido proporcionalmente (en la prueba esto dio correlación
   0,974 entre ambas participaciones), y el ranking cambiaría con la semilla → no reproducible.
3. **Conclusión metodológica:** sin llave, la única forma honesta de usar el dataset es el **análisis
   descriptivo propio**. Y queda registrada la mejora de captura sugerida: **registrar `id_cliente` en
   cada venta** habilitaría el análisis de comportamiento por cliente en un futuro corte.

**Qué se analizó (y por qué esas variables):** distribución e ingresos (`describe`), perfil por ciudad
(`groupby` con conteo, media, mediana y edad), histogramas de ingreso y edad, y dispersión
edad–ingresos. Son las cuatro lecturas que responden *quiénes son los clientes*.

**Resultados:** 567 clientes, 12 ciudades, sin nulos ni duplicados; ingresos media ARS 34.669 /
mediana 35.067; edad media 37,9 (20–81); **correlación edad–ingresos 0,009 → no hay relación lineal**;
por ciudad el rango entre la mejor y la peor es de sólo **11 %** (no hay mercados diferenciados), pero sí
hay diferencia de volumen (Mar del Plata 63 vs. Buenos Aires 36).

---

## 5. Decisiones transversales

| Decisión | Justificación |
|---|---|
| **Moneda: pesos argentinos (ARS)** | Ningún archivo declara divisa: `Pre.docx` no menciona moneda y no hay columna de moneda; el único símbolo es `$` en `precio`, que por convención local es pesos. Las etiquetas "USD" que aparecían inicialmente eran una suposición sin respaldo y se corrigieron (40 reemplazos) documentando el criterio en el notebook |
| **Formato numérico del texto** | En la prosa se usa formato argentino (51.092,96) porque es ARS; las salidas de código muestran el formato por defecto de pandas/Python. Se documentó que el separador decimal de los CSV es un artefacto de generación, no una indicación de dólares |
| **Idioma y estructura del notebook** | Español, con encabezados por actividad (1 a 5 / 1 a 4) que calzan con el enunciado: al evaluador le resulta directo mapear celda ↔ requisito |
| **Markdown antes de cada bloque** | El TP lo exige explícitamente ("bloques de texto y código") y además fuerza a justificar cada decisión en el momento de tomarla |
| **Celdas de verificación** | Después de cada paso crítico (carga, limpieza, integración) se re-imprime el estado: nulos, duplicados, formas, filas. Permite detectar si una celda anterior cambió de comportamiento al re-ejecutar |
| **Gráficos guardados en `anexos/`** | El bloque Anexo pide "links a todos los archivos adicionales"; si los gráficos sólo vivieran embebidos en el notebook, no habría archivos que enlazar |
| **Notebook ejecutado y commiteado con salidas** | Prueba de que el trabajo corre de punta a punta en una sola tanda y permite revisar resultados sin re-ejecutar |

---

## 6. Resultados clave (resumen ejecutivo)

| Indicador | Valor |
|---|---|
| Filas: crudo → limpio | 3.035 → **2.998** (−35 duplicados, −2 nulos) |
| Impacto de los duplicados | ARS 15.949,41 (1,1 % de los ingresos) |
| Ingresos 2024 | **ARS 1.467.093,52** · 12 meses · ticket medio 489,36 |
| Mes mejor / peor (limpio) | mayo 143.727 / junio 108.480 (CV 8,5 %) |
| Alto rendimiento (P75) | umbral **ARS 51.093** → **8 productos, 34,1 %** de los ingresos |
| Categorías | Electrodomésticos 34,4 % · Electrónica 32,9 % · Decoración 32,7 % (diferencia por ticket, no por volumen) |
| ROAS por producto | promedio 3.322 (máx. Lámpara de mesa 5.165) — alto rendimiento 4.088 vs. resto 3.044 |
| ROAS por canal | **Email 1.048 · TV 944 · RRSS 831** |
| Cobertura de campañas | sólo 28,4 % de los ingresos cae en ventana de campaña |
| Clientes | 567 · 12 ciudades · edad 37,9 · r(edad, ingresos) = 0,009 |

---

## 7. Limitaciones y recomendaciones

**Limitaciones (declaradas en el notebook):**
* Sin `id_cliente` en ventas no hay análisis por cliente ni atribución de canal.
* La atribución de campaña es por ventana temporal: indica coincidencia en el tiempo, no causalidad.
* Costos e ingresos de campaña están en escalas distintas; el ROAS se usa para comparar, no como retorno absoluto.
* Los tres canales corren en paralelo para cada producto → la comparación entre canales no es experimental.

**Recomendaciones de negocio derivadas del análisis:**
1. Priorizar los **8 productos de alto rendimiento** en **Email** (mejor ROAS y menor costo).
2. Ampliar la cobertura de campaña: el 71,6 % de los ingresos ocurre fuera de cualquier ventana activa.
3. Subir el **ticket promedio en Decoración** (packs/accesorios), porque la categoría pierde por precio,
   no por frecuencia de compra.
4. **Registrar `id_cliente` en cada venta** para habilitar el análisis de comportamiento y la atribución
   real por cliente en futuros cortes.

---

## 8. Verificación final

* `TP_Integrador.ipynb`: **62 celdas** (27 Markdown / 35 código), **0 errores de ejecución**, todas con salida.
* Cobertura del enunciado: 4 actividades de la Etapa 1 + 4 de la Etapa 2, más análisis complementario,
  conclusiones y bloque **Anexo**.
* Salidas generadas: 6 gráficos en `anexos/` referenciados en el Anexo.
