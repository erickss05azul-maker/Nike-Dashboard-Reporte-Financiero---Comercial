<div align="center">

<img src="https://upload.wikimedia.org/wikipedia/commons/a/a6/Logo_NIKE.svg" width="80px" />

# Nike Sales Dashboard
### Power BI · Análisis Comercial y Financiero 2023–2026

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat-square&logo=powerbi&logoColor=black)
![Power Query](https://img.shields.io/badge/Power%20Query-217346?style=flat-square&logo=microsoft&logoColor=white)
![DAX](https://img.shields.io/badge/DAX-E31837?style=flat-square&logoColor=white)
![Status](https://img.shields.io/badge/Estado-Completado-00A86B?style=flat-square)

</div>

---

> Proyecto de Business Intelligence end-to-end sobre un dataset de ventas de Nike obtenido de Kaggle.  
> Cubre desde la limpieza de datos en Power Query hasta un dashboard interactivo de cuatro páginas  
> con modelo dimensional en estrella, medidas DAX con inteligencia de tiempo y navegación entre páginas.  
> Los datos originales fueron expandidos con apoyo de Inteligencia Artificial para enriquecer el análisis.

---

## 👤 Autor

**Erick Rodrigo Salcca Solorzano**  
Estudiante de Economía 8vo. Ciclo — Área de interés: Planeamiento Financiero · Control de Gestión · Planeamiento Comercial

---

## 🗂️ Estructura del repositorio

```
Nike-Sales-Dashboard-PowerBI/
│
├── 📁 Dataset/
│   ├── Nike_Sales_Original.csv        ← dataset original de Kaggle (2,500 filas)
│   └── Nike_Sales_Expanded.csv        ← dataset ampliado con IA (8,000 filas)
│
├── 📁 Dashboard/
│   └── NIKE_DASHBOARD.pbix
│
├── 📁 Imagenes/
│   ├── Resumen_Ejecutivo.png
│   ├── Rentabilidad.png
│   ├── Geografico.png
│   └── Analisis_Temporal.png
│
└── README.md
```

---

## 📦 Origen de los datos

### Dataset original — Kaggle

| Ítem | Detalle |
|---|---|
| Nombre | Nike Sales (Uncleaned) Dataset |
| Plataforma | Kaggle |
| Autor | nayakganesh007 |
| Enlace | [Ver dataset en Kaggle](https://www.kaggle.com/datasets/nayakganesh007/nike-sales-uncleaned-dataset/data) |
| Licencia | CC0 — Dominio Público (dataset 100% sintético, sin datos reales de Nike) |
| Registros originales | 2,500 filas · 13 columnas |
| Período | 2023 |

El dataset simula transacciones de venta minorista y online de Nike. Fue creado intencionalmente con datos sucios para practicar limpieza, análisis exploratorio y construcción de dashboards. Incluye problemas reales como nulos, errores tipográficos en regiones, formatos de fecha inconsistentes, descuentos mayores al 100% y valores negativos en columnas numéricas.

### Ampliación con Inteligencia Artificial

El dataset original de 2,500 filas fue expandido a **8,000 filas y 20 columnas** con apoyo de Inteligencia Artificial para enriquecer el análisis y construir un proyecto de portafolio más robusto.

**Lo que se amplió:**

| Ampliación | Detalle |
|---|---|
| Cobertura geográfica | De 9 ciudades de India a 28 ciudades en India, Latinoamérica, Europa y USA |
| Rango temporal | Extendido de 2023 a 2023–2026 |
| Nuevas columnas | `Customer_Type`, `Seller_ID`, `Unit_Cost`, `Sales_Budget`, `Payment_Method`, `Return_Flag`, `Customer_Satisfaction` |
| Canal adicional | Se agregó `Wholesale` además de Online y Retail |
| Errores intencionales | Se mantuvieron y ampliaron los errores de calidad para practicar el ETL |

> Los errores de limpieza fueron preservados intencionalmente en el dataset expandido para que todo el proceso ETL se realice dentro de Power Query, demostrando la capacidad de transformación de datos sin necesidad de preprocesamiento externo.

---

## 🎯 Caso de negocio

**Preguntas que responde el dashboard:**

| # | Pregunta |
|---|---|
| 1 | ¿Cuánto se vendió y cuánto se ganó, comparado con el año anterior? |
| 2 | ¿Qué línea de producto y qué zona concentran el margen bruto? |
| 3 | ¿Cómo evoluciona el ingreso acumulado (YTD) año a año? |
| 4 | ¿En qué meses hubo caída de ingresos respecto al mes anterior (MoM)? |
| 5 | ¿Qué canal de ventas genera mayor volumen? |
| 6 | ¿Cómo se distribuyen las ventas por zona geográfica y segmento de cliente? |

---

## ⚙️ Proceso ETL — Power Query

> **Herramienta utilizada:** todo el proceso de limpieza y transformación se realizó exclusivamente en **Power Query** dentro de Power BI Desktop, sin preprocesamiento externo.

### Problemas encontrados y decisiones tomadas

| Problema | Columna afectada | Decisión tomada |
|---|---|---|
| 5 formatos de fecha distintos mezclados | `Order_Date` | Parseo condicional por patrón con `try...otherwise null` en columna personalizada M |
| Fechas imposibles (mes 57, día 30 en nov, mes 16) | `Order_Date` | Convertidas a null — no recuperables sin dato de origen |
| Valores negativos en unidades | `Units_Sold` | Convertidos a positivo (error de captura del operador) |
| Revenue negativo | `Revenue` | Flagueado con columna derivada `Revenue_Flag` |
| Descuentos mayores al 100% | `Discount_Applied` | Convertidos a null — error sin posibilidad de corrección |
| 69 variantes para 28 ciudades | `Region` | Tabla de mapeo con `Record.FieldOrDefault` en columna personalizada |
| Dos formatos de Seller_ID (`VEN-XXX` y `VNDxx`) | `Seller_ID` | Estandarizados todos a formato `VEN-XXX` |
| Valores fuera de rango 1–5 | `Customer_Satisfaction` | Convertidos a null mediante columna condicional |
| Order_ID duplicados (~3% del dataset) | `Order_ID` | Eliminados conservando primera ocurrencia (274 filas eliminadas) |
| Nulos en columnas categóricas | `Customer_Type`, `Payment_Method`, `Return_Flag` | Reemplazados por `"Sin registrar"` / `"Sin dato"` |
| Tallas con valores inválidos (`N/A`, `?`, `-`, vacío) | `Talla` | Reemplazados por null de forma uniforme |

### Columnas derivadas creadas en Power Query

| Columna nueva | Lógica aplicada | Propósito en el modelo |
|---|---|---|
| `Revenue_Flag` | Condicional: Normal / Negativo / Cero / Sin dato | Identificar transacciones problemáticas |
| `Zone` | Mapeo de Region a zona geográfica con `List.Contains` | Nivel de agrupación superior para análisis geográfico |
| `Año` | `Date.Year([Order_Date])` | Filtros temporales y segmentadores |
| `Mes_Número` | `Date.Month([Order_Date])` | Ordenamiento correcto de meses en gráficos |
| `Mes_Nombre` | `Date.MonthName([Order_Date], "es-ES")` | Etiquetas en español en los visuales |
| `Trimestre` | `"T" & Date.QuarterOfYear([Order_Date])` | Agrupación trimestral |
| `Semana` | `Date.WeekOfYear([Order_Date])` | Granularidad semanal opcional |

> **Criterio clave aplicado:** nulo y cero no significan lo mismo. Nulo significa que el dato no existe o no fue registrado; cero significa que ocurrió y el valor fue cero. Tratarlos de forma equivalente destruye promedios y denominadores en DAX.

---

## ⭐ Modelo de datos — Esquema en estrella

```
                      Dim_Tiempo
                          │
  Dim_Cliente ──── Tabla_Hechos ──── Dim_Producto
                          │
              Dim_Ubicación     Dim_Vendedor
                          │
                    Dim_Transacción
```

| Tabla | Tipo | Campos principales |
|---|---|---|
| `Tabla_Hechos` | Hechos | Revenue, Units_Sold, Profit, MRP, Discount_Applied, Sales_Budget, Unit_Cost, Customer_Satisfaction, Talla |
| `Dim_Tiempo` | Dimensión de tiempo | Fecha, Año, Mes, Trimestre, AñoTrimestre, AñoMes, DíaSemana, EsFinDeSemana, NúmeroSemana |
| `Dim_Producto` | Dimensión | Product_Line, Product_Name, Gender_Category |
| `Dim_Ubicación` | Dimensión | Region, Zone |
| `Dim_Cliente` | Dimensión | Customer_Type, Sales_Channel |
| `Dim_Vendedor` | Dimensión | Seller_ID |
| `Dim_Transacción` | Dimensión | Payment_Method, Return_Flag |

**Granularidad:** un registro = una transacción de venta.  
**Dirección de filtro:** de dimensiones hacia tabla de hechos (uno a muchos, filtro simple).  
**Tabla calendario:** continua del 01/01/2023 al 31/12/2026, generada dinámicamente desde el mínimo y máximo de `Order_Date`, marcada como tabla de fechas para habilitar funciones de inteligencia de tiempo.

---

## 📐 Medidas DAX

Todas las medidas están organizadas en una tabla vacía `_Medidas` separada de la tabla de hechos para mantener el modelo ordenado y profesional.

### Métricas base

```dax
Ingresos_Totales = SUM(Tabla_Hechos[Revenue])

Ganancia_Total = SUM(Tabla_Hechos[Profit])

Unidades_Vendidas = SUM(Tabla_Hechos[Units_Sold])

#Transacciones = COUNTROWS(Tabla_Hechos)

Ticket_promedio = DIVIDE([Ingresos_Totales], [Unidades_Vendidas], 0)

Ganancia_bruta = DIVIDE([Ganancia_Total], [Ingresos_Totales], 0)

Costo_Total =
SUMX(
    FILTER(Tabla_Hechos, NOT(ISBLANK(Tabla_Hechos[Unit_Cost]))),
    Tabla_Hechos[Unit_Cost] * Tabla_Hechos[Units_Sold]
)
```

### Inteligencia de tiempo

```dax
Ingresos_YTD =
TOTALYTD([Ingresos_Totales], Dim_Tiempo[Fecha])

Ganancia_YTD =
TOTALYTD([Ganancia_Total], Dim_Tiempo[Fecha])

INGRESO_AÑO_ANTERIOR =
CALCULATE([Ingresos_Totales], DATEADD(Dim_Tiempo[Fecha], -1, YEAR))

Ingreso_YoY% =
VAR INGRESO_ANIO_ACTUAL   = [Ingresos_Totales]
VAR INGRESO_ANIO_ANTERIOR = CALCULATE([Ingresos_Totales], DATEADD(Dim_Tiempo[Fecha], -1, YEAR))
RETURN DIVIDE(INGRESO_ANIO_ACTUAL - INGRESO_ANIO_ANTERIOR, INGRESO_ANIO_ANTERIOR, 0)
```

### Medidas de texto para KPIs (variación con flecha)

```dax
Variacion_Ingresos_Texto =
VAR Actual    = [Ingresos_Totales]
VAR Anterior  = CALCULATE([Ingresos_Totales], DATEADD(Dim_Tiempo[Fecha], -1, YEAR))
VAR Variacion = DIVIDE(Actual - Anterior, Anterior, 0)
RETURN
    SWITCH(
        TRUE(),
        ISBLANK(Anterior), "Sin año anterior",
        Variacion >= 0,    " ▲ +" & FORMAT(Variacion, "0.00%"),
                           " ▼ "  & FORMAT(Variacion, "0.00%")
    )

Variacion_Ganancias_Texto =
VAR Actual    = [Ganancia_Total]
VAR Anterior  = CALCULATE([Ganancia_Total], DATEADD(Dim_Tiempo[Fecha], -1, YEAR))
VAR Variacion = DIVIDE(Actual - Anterior, Anterior, 0)
RETURN
    SWITCH(
        TRUE(),
        ISBLANK(Anterior), "Sin año anterior",
        Variacion >= 0,    " ▲ +" & FORMAT(Variacion, "0.00%"),
                           " ▼ "  & FORMAT(Variacion, "0.00%")
    )

Variacion_GananciaBruta_Texto =
VAR Actual    = [Ganancia_bruta]
VAR Anterior  = CALCULATE([Ganancia_bruta], DATEADD(Dim_Tiempo[Fecha], -1, YEAR))
VAR Variacion = Actual - Anterior
RETURN
    SWITCH(
        TRUE(),
        ISBLANK(Anterior), "Sin año anterior",
        Variacion >= 0,    " ▲ +" & FORMAT(Variacion * 100, "0.00") & " PT",
                           " ▼ "  & FORMAT(ABS(Variacion) * 100, "0.00") & " PT"
    )

Variacion_TICKETPRO_Texto =
VAR Actual    = [Ticket_promedio]
VAR Anterior  = CALCULATE([Ticket_promedio], DATEADD(Dim_Tiempo[Fecha], -1, YEAR))
VAR Variacion = DIVIDE(Actual - Anterior, Anterior, 0)
RETURN
    SWITCH(
        TRUE(),
        ISBLANK(Anterior), "Sin año anterior",
        Variacion >= 0,    " ▲ +" & FORMAT(Variacion, "0.00%"),
                           " ▼ "  & FORMAT(Variacion, "0.00%")
    )

Variacion_Transacciones_Texto =
VAR Actual    = [#Transacciones]
VAR Anterior  = CALCULATE([#Transacciones], DATEADD(Dim_Tiempo[Fecha], -1, YEAR))
VAR Variacion = DIVIDE(Actual - Anterior, Anterior, 0)
RETURN
    SWITCH(
        TRUE(),
        ISBLANK(Anterior), "Sin año anterior",
        Variacion >= 0,    " ▲ +" & FORMAT(Variacion, "0.00%"),
                           " ▼ "  & FORMAT(Variacion, "0.00%")
    )

Variacion_IngresosYTD_Texto =
VAR Actual    = [Ingresos_YTD]
VAR Anterior  = CALCULATE([Ingresos_YTD], DATEADD(Dim_Tiempo[Fecha], -1, YEAR))
VAR Variacion = DIVIDE(Actual - Anterior, Anterior, 0)
RETURN
    SWITCH(
        TRUE(),
        ISBLANK(Anterior), "Sin año anterior",
        Variacion >= 0,    " ▲ +" & FORMAT(Variacion, "0.00%"),
                           " ▼ "  & FORMAT(Variacion, "0.00%")
    )

Variacion_Ingreso_YoY%_Texto =
VAR Actual    = [Ingreso_YoY%]
VAR Anterior  = CALCULATE([Ingreso_YoY%], DATEADD(Dim_Tiempo[Fecha], -1, YEAR))
VAR Variacion = Actual - Anterior
RETURN
    SWITCH(
        TRUE(),
        ISBLANK(Anterior), "Sin año anterior",
        Variacion >= 0,    " ▲ +" & FORMAT(Variacion * 100, "0.00") & " PT",
                           " ▼ "  & FORMAT(ABS(Variacion) * 100, "0.00") & " PT"
    )
```

> **Nota sobre PT vs %:** las medidas de variación de `Ganancia_bruta` e `Ingreso_YoY%` expresan la variación en **puntos porcentuales (PT)** porque comparan dos ratios — la diferencia entre porcentajes no es un porcentaje sino una resta directa. Las demás medidas usan `DIVIDE` para obtener el crecimiento proporcional expresado en %.

### Medidas de color para formato condicional

```dax
Color_Variacion_Ingresos =
VAR Actual    = [Ingresos_Totales]
VAR Anterior  = CALCULATE([Ingresos_Totales], DATEADD(Dim_Tiempo[Fecha], -1, YEAR))
VAR Variacion = DIVIDE(Actual - Anterior, Anterior, 0)
RETURN
    IF(ISBLANK(Anterior), "#AAAAAA",
        IF(Variacion >= 0, "#1DB954", "#FF4444"))

Color_Gnancia_bruta =
VAR Actual    = [Ganancia_bruta]
VAR Anterior  = CALCULATE([Ganancia_bruta], DATEADD(Dim_Tiempo[Fecha], -1, YEAR))
VAR Variacion = DIVIDE(Actual - Anterior, Anterior, 0)
RETURN
    IF(ISBLANK(Anterior), "#AAAAAA",
        IF(Variacion >= 0, "#1DB954", "#FF4444"))

Color_Ticket_promedio =
VAR Actual    = [Ticket_promedio]
VAR Anterior  = CALCULATE([Ticket_promedio], DATEADD(Dim_Tiempo[Fecha], -1, YEAR))
VAR Variacion = Actual - Anterior
RETURN
    IF(ISBLANK(Anterior), "#AAAAAA",
        IF(Variacion >= 0, "#1DB954", "#FF4444"))

Color_#Transacciones =
VAR Actual    = [#Transacciones]
VAR Anterior  = CALCULATE([#Transacciones], DATEADD(Dim_Tiempo[Fecha], -1, YEAR))
VAR Variacion = Actual - Anterior
RETURN
    IF(ISBLANK(Anterior), "#AAAAAA",
        IF(Variacion >= 0, "#1DB954", "#FF4444"))

Color_Ingreso_YoY% =
VAR Actual    = [Ingreso_YoY%]
VAR Anterior  = CALCULATE([Ingreso_YoY%], DATEADD(Dim_Tiempo[Fecha], -1, YEAR))
VAR Variacion = Actual - Anterior
RETURN
    IF(ISBLANK(Anterior), "#AAAAAA",
        IF(Variacion >= 0, "#1DB954", "#FF4444"))

Color_Variacion_IngresosYTD =
VAR Actual    = [Ingresos_YTD]
VAR Anterior  = CALCULATE([Ingresos_YTD], DATEADD(Dim_Tiempo[Fecha], -1, YEAR))
VAR Variacion = DIVIDE(Actual - Anterior, Anterior, 0)
RETURN
    IF(ISBLANK(Anterior), "#AAAAAA",
        IF(Variacion >= 0, "#1DB954", "#FF4444"))
```

> **Lógica de colores:** verde `#1DB954` cuando la variación es positiva, rojo `#FF4444` cuando es negativa, gris `#AAAAAA` cuando no existe año anterior para comparar. Estas medidas se aplican en **Formato → Valor → fx → Campo** dentro de cada tarjeta KPI.

---

## 📊 Páginas del dashboard

### Página 1 — Resumen Ejecutivo
KPIs con variación YoY y color condicional · Tendencia de ingresos por trimestre · Unidades vendidas por línea de producto · Distribución por canal de ventas  
**Segmentadores:** Año · Trimestre · Canal de Ventas

![Resumen Ejecutivo](Imagenes/Resumen_Ejecutivo.png)

---

### Página 2 — Rentabilidad
Ganancia y margen bruto por trimestre · Ingresos por línea y segmento de género · Margen por tipo de cliente y canal de ventas  
**Segmentadores:** Año · Línea de Producto · Zona

![Rentabilidad](Imagenes/Rentabilidad.png)

---

### Página 3 — Análisis Geográfico
Mapa de burbujas por región · Tabla comparativa por zona · Dispersión Ingresos vs Margen Bruto por zona  
**Segmentadores:** Año · Zona · Género

![Geográfico](Imagenes/Geografico.png)

---

### Página 4 — Análisis Temporal
Ingresos YTD acumulado comparativo 2023–2026 · Comparativo YoY mensual · Variación MoM con barras positivo/negativo · Tabla resumen trimestral con variaciones  
**Segmentadores:** Año · Línea de Producto

![Análisis Temporal](Imagenes/Analisis_Temporal.png)

---

## 🛠️ Herramientas utilizadas

| Herramienta | Uso en el proyecto |
|---|---|
| **Power BI Desktop** | Modelado dimensional, creación de medidas DAX y diseño del dashboard |
| **Power Query (M)** | Todo el proceso ETL: limpieza, transformación y carga de datos |
| **DAX** | Medidas de KPIs, inteligencia de tiempo, colores condicionales y textos de variación |
| **Inteligencia Artificial** | Ampliación del dataset original de 2,500 a 8,000 filas con nuevas columnas y regiones |

> No se utilizó Python ni ninguna herramienta externa para la limpieza de datos. Todo el ETL fue realizado íntegramente en Power Query dentro de Power BI Desktop.

---

## 💡 Aprendizajes clave

- El parseo de fechas con múltiples formatos requiere lógica condicional explícita en M — Power Query no puede inferir el formato cuando hay ambigüedad entre `DD/MM` y `MM/DD` en la misma columna.
- Nulo y cero no son equivalentes: tratarlos igual destruye promedios y denominadores en DAX. Cada caso requiere una decisión analítica documentada.
- La variación de un ratio (como margen %) se expresa en puntos porcentuales (PT), no en porcentaje — la diferencia entre dos porcentajes es una resta directa, no un cociente.
- Un modelo en estrella bien diseñado permite agregar nuevas medidas DAX sin tocar la estructura de datos — la inversión en modelado paga cada vez que se agrega un KPI.
- Las funciones de inteligencia de tiempo (`TOTALYTD`, `DATEADD`, `SAMEPERIODLASTYEAR`) solo funcionan correctamente con una tabla calendario continua sin gaps, marcada explícitamente como tabla de fechas.
- Separar las medidas en una tabla vacía `_Medidas` hace el modelo más mantenible y profesional — evita que las medidas queden mezcladas con los campos de la tabla de hechos.

---

## 📎 Referencias

- Nayak, G. (2024). *Nike Sales (Uncleaned) Dataset*. Kaggle. [https://www.kaggle.com/datasets/nayakganesh007/nike-sales-uncleaned-dataset/data](https://www.kaggle.com/datasets/nayakganesh007/nike-sales-uncleaned-dataset/data)

---

<div align="center">

*Proyecto desarrollado como parte de un portafolio orientado a prácticas preprofesionales*  
*en planeamiento financiero, control de gestión y planeamiento comercial.*

</div>
