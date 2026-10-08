# TP Integrador — Data Analytics (Comisión C26251)

Pre-entrega obligatoria del Trabajo Práctico Integrado: recopilación, limpieza, transformación,
agregación e integración de datos con Python y Pandas.

**Integrante:** Franco Oscar Alejandro · **Repositorio:** https://github.com/oscale05/TP-Integrador-C26251

## Estructura

```
TP-Integrador-C26251/
├── TP_Integrador.ipynb      # notebook principal (Etapa 1 y 2, con resultados ejecutados)
├── README.md
├── REPORTE.md                # reporte técnico: pasos y justificación de cada decisión
├── Pre.docx                 # consigna original
├── datos/
│   ├── ventas.csv           # 3.035 registros — originales, sin modificar
│   ├── clientes.csv         #   567 registros — originales, sin modificar
│   └── marketing.csv        #    90 registros — originales, sin modificar
└── anexos/                  # gráficos generados por el notebook
    ├── 01_alto_rendimiento.png
    ├── 02_categorias.png
    ├── 03_top10_productos.png
    ├── 04_roas_productos.png
    ├── 05_canal_marketing.png
    └── 06_clientes.png
```

## Cómo ejecutarlo

**Google Colaboratory (recomendado):** subir la carpeta a Drive y abrir `TP_Integrador.ipynb`
con Colab → `Runtime → Run all`. La celda de carga busca los CSV en la carpeta de Drive o
en el entorno (`/content`).

**Local:** `jupyter notebook TP_Integrador.ipynb` desde la raíz del repositorio
(la carga detecta `./datos/` automáticamente).

## Qué contiene el notebook

| Etapa | Actividades |
|---|---|
| **1 — Recopilación y preparación** | Carga de los 3 CSV · script de ventas mensuales en Python puro (variables y operadores) · estructuras de datos (lista vs diccionario, con decisión justificada) · EDA con Pandas · diagnóstico de calidad (35 duplicados + 2 nulos en `ventas.csv`) |
| **2 — Preprocesamiento y limpieza** | Limpieza 3.035 → 2.998 filas (duplicados, nulos, `$` → numérico, fechas) · filtro de alto rendimiento con criterio **P75 = ARS 51.093** (8 productos, 34,1 % de los ingresos) · agregación por categoría · integración ventas × marketing (agregada y por ventana temporal) |
| **Final** | Análisis complementario de `clientes.csv` · conclusiones generales · bloque **Anexo** |

> El detalle de cada paso y la justificación de cada decisión están en [`REPORTE.md`](REPORTE.md).

## Decisiones metodológicas

- **Alto rendimiento = ingreso total por producto ≥ percentil 75.** Criterio relativo al dataset,
  robusto a outliers, equivalente al cuartil superior y auditable; comparado en el notebook contra
  P50 (no discrimina) y P90 (demasiado exigente).
- **`clientes.csv` no se cruza con ventas.** No existe columna en común (`id_cliente` no figura en
  ventas, ni ciudad ni ningún dato geográfico), por lo que se analiza en forma independiente como
  perfil demográfico del negocio.
- **Integración sin inflación de filas:** un `merge` directo de ventas × marketing multiplica por 3
  las filas (cada producto tiene 3 campañas); se agrega antes de unir y se atribuye por ventana temporal.

## Entrega

Carpeta de Google Drive nombrada `Franco Oscar Alejandro - Comisión C26251 - TPI Data Analytics`
con los datos originales, este notebook (con el bloque **Anexo** al final) y los archivos generados.
