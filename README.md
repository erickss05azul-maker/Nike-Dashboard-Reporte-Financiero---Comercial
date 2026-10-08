Nike Dashboard Reporte Financiero Comercial
### Análisis Comercial y Financiero 2023–2026 · Proceso ETL, Modelo Dimensional y Medidas DAX

---

## Autor

**Erick Rodrigo Salcca Solorzano** — Estudiante de Economía  
Área de interés: Planeamiento Financiero · Control de Gestión · Planeamiento Comercial

---

## Resumen del proyecto

Proyecto de Business Intelligence end-to-end construido sobre un dataset sintético de ventas de Nike.  
Cubre desde la limpieza de datos en Power Query hasta la publicación de un dashboard interactivo de cuatro páginas con navegación, segmentadores dinámicos y medidas DAX con inteligencia de tiempo.

El proyecto simula el flujo de trabajo real de un analista en un área de planeamiento:  
extracción y limpieza del dato → modelado dimensional → definición de KPIs → visualización ejecutiva.

---

## Caso de negocio

Nike genera miles de transacciones de venta en múltiples canales, regiones y líneas de producto.  
El área comercial necesita una vista única que responda preguntas de gestión sin depender de extracciones manuales.

**Preguntas de negocio que responde el dashboard:**

- ¿Cuánto se vendió y cuánto se ganó en el período, comparado con el año anterior?
- ¿Qué línea de producto y qué zona geográfica concentran el margen?
- ¿Cómo evoluciona el ingreso acumulado (YTD) año a año?
- ¿En qué meses hubo caída de ventas respecto al mes anterior (MoM)?
- ¿Qué canal de ventas genera mayor rentabilidad?
- ¿Qué vendedor cumple mejor su cuota y cuál necesita seguimiento?

---

## Origen de los datos

| Ítem | Detalle |
|---|---|
| Dataset base | Nike Sales Uncleaned — Kaggle |
| Tipo | Datos sintéticos de transacciones minoristas y online |
| Registros originales | 2,500 filas · 13 columnas |
| Registros expandidos | 8,000 filas · 20 columnas |
| Rango temporal | Enero 2023 — Marzo 2026 |
| Regiones | 28 ciudades en India, Latinoamérica, Europa y USA |

El dataset fue ampliado con Python para agregar cobertura geográfica internacional, nuevas columnas de análisis financiero y errores intencionales de calidad de datos para practicar la limpieza en Power Query.

---

## Columnas del dataset expandido

| Columna | Descripción |
|---|---|
| Order_ID | Identificador de transacción (con duplicados intencionales) |
| Gender_Category | Segmento del producto: Men, Women, Kids |
| Product_Line | Familia del producto: Running, Basketball, Lifestyle, Training, Soccer |
| Product_Name | Modelo específico vendido |
| Talla | Talla del producto (numérica o alfabética según tipo) |
| Units_Sold | Unidades vendidas (con negativos y nulos como errores) |
| MRP | Precio de lista antes del descuento |
| Discount_Applied | Descuento aplicado en decimales (algunos > 1.0 como error) |
| Revenue | Ingreso final después del descuento |
| Order_Date | Fecha de transacción (5 formatos distintos mezclados) |
| Sales_Channel | Online / Retail / Wholesale |
| Region | Ciudad de venta (con errores tipográficos) |
| Profit | Ganancia obtenida |
| Customer_Type | B2C / B2B / Distributor |
| Seller_ID | Código de vendedor (formato inconsistente y nulos) |
| Unit_Cost | Costo unitario del producto |
| Sales_Budget | Presupuesto de ventas por transacción |
| Payment_Method | Medio de pago utilizado |
| Return_Flag | Estado de devolución: No / Sí / Pendiente |
| Customer_Satisfaction | Calificación 1–5 (con valores fuera de rango como error) |

---

## Proceso ETL — Power Query

### Problemas de calidad encontrados y decisiones tomadas

| Problema | Columna | Decisión |
|---|---|---|
| 5 formatos de fecha distintos | Order_Date | Parseo condicional por formato con `try...otherwise null` |
| Fechas imposibles (mes 57, día 30 en nov) | Order_Date | Convertidas a null — no se puede recuperar la fecha real |
| Valores negativos | Units_Sold, Revenue | Units_Sold: convertidos a positivo. Revenue: flagueado con Revenue_Flag |
| Descuentos > 100% | Discount_Applied | Convertidos a null — error de captura sin posibilidad de corrección |
| 69 variantes para 28 ciudades | Region | Tabla de mapeo con `Record.FieldOrDefault` en columna personalizada |
| Dos formatos de Seller_ID | Seller_ID | Estandarizados a formato VEN-XXX |
| Valores fuera de rango 1–5 | Customer_Satisfaction | Convertidos a null con columna condicional |
| Duplicados en Order_ID | Order_ID | Eliminados conservando primera ocurrencia (274 filas eliminadas) |
| Nulos en columnas categóricas | Customer_Type, Payment_Method, Return_Flag | Reemplazados por "Sin registrar" / "Sin dato" |

### Columnas derivadas creadas en Power Query

- `Revenue_Flag` — Normal / Negativo / Cero / Sin dato
- `Zone` — Clasificación geográfica: India / Latinoamérica / Europa / USA
- `Año`, `Mes_Número`, `Mes_Nombre`, `Trimestre`, `Semana` — derivadas de Order_Date limpia

---

## Modelado de datos — Esquema en estrella

```
                    Dim_Tiempo
                        │
Dim_Cliente ──── Tabla_Hechos ──── Dim_Producto
                        │
              Dim_Ubicación    Dim_Vendedor
                        │
                  Dim_Transacción
```

| Tabla | Tipo | Campos clave |
|---|---|---|
| Tabla_Hechos | Hechos | Order_ID, Revenue, Units_Sold, Profit, MRP, Discount_Applied, Sales_Budget, Unit_Cost, Customer_Satisfaction, Talla |
| Dim_Tiempo | Dimensión de tiempo | Fecha, Año, Mes, Trimestre, AñoTrimestre, AñoMes, DíaSemana, EsFinDeSemana |
| Dim_Producto | Dimensión | Product_Line, Product_Name, Gender_Category |
| Dim_Ubicación | Dimensión | Region, Zone |
| Dim_Cliente | Dimensión | Customer_Type, Sales_Channel |
| Dim_Vendedor | Dimensión | Seller_ID |
| Dim_Transacción | Dimensión | Payment_Method, Return_Flag |

**Granularidad:** un registro = una transacción de venta.  
**Dirección de filtro:** de dimensiones hacia tabla de hechos (uno a muchos).

---

## Medidas DAX

### Métricas base
```dax
Ingresos Totales = SUM(Tabla_Hechos[Revenue])
Ganancia Total = SUM(Tabla_Hechos[Profit])
Unidades Vendidas = SUM(Tabla_Hechos[Units_Sold])
Num Transacciones = COUNTROWS(Tabla_Hechos)
Ticket Promedio = DIVIDE([Ingresos Totales], [Num Transacciones], 0)
Margen Bruto % = DIVIDE([Ganancia Total], [Ingresos Totales], 0)
```

### Inteligencia de tiempo
```dax
Ingresos YTD = TOTALYTD([Ingresos Totales], Dim_Tiempo[Fecha])

Ingresos YoY % =
VAR Actual = [Ingresos Totales]
VAR Anterior = CALCULATE([Ingresos Totales], SAMEPERIODLASTYEAR(Dim_Tiempo[Fecha]))
RETURN DIVIDE(Actual - Anterior, Anterior, 0)

Ingresos MoM % =
VAR Actual = [Ingresos Totales]
VAR Anterior = CALCULATE([Ingresos Totales], DATEADD(Dim_Tiempo[Fecha], -1, MONTH))
RETURN DIVIDE(Actual - Anterior, Anterior, 0)

Ingresos Promedio Movil 3M =
AVERAGEX(
    DATESINPERIOD(Dim_Tiempo[Fecha], LASTDATE(Dim_Tiempo[Fecha]), -3, MONTH),
    [Ingresos Totales]
)
```

### Rendimiento comercial
```dax
Tasa Devolución % =
DIVIDE(
    COUNTROWS(FILTER(Tabla_Hechos, Tabla_Hechos[Return_Flag] = "Sí")),
    [Num Transacciones], 0
)

Ranking Vendedor =
RANKX(ALL(Dim_Vendedor[Seller_ID]), [Ingresos Totales], , DESC, DENSE)
```

---

## Estructura del dashboard

### Página 1 — Resumen Ejecutivo
KPIs con variación YoY · Tendencia de ingresos por trimestre · Unidades por línea de producto · Distribución por canal de ventas

### Página 2 — Rentabilidad
Ganancia y margen bruto por trimestre · Ingresos por línea y segmento de género · Margen por tipo de cliente y canal

### Página 3 — Análisis Geográfico
Mapa de burbujas por región · Tabla comparativa por zona · Dispersión Ingresos vs Margen Bruto por zona

### Página 4 — Análisis Temporal
Ingresos YTD acumulado por año (comparativo 2023–2026) · Comparativo YoY mensual · Tabla de resumen trimestral con variaciones

---

## Vista previa del dashboard

### Resumen Ejecutivo
![Resumen Ejecutivo](Imagenes/Resumen_Ejecutivo.png)

### Rentabilidad
![Rentabilidad](Imagenes/Rentabilidad.png)

### Análisis Geográfico
![Geográfico](Imagenes/Geografico.png)

### Análisis Temporal
![Análisis Temporal](Imagenes/Analisis_Temporal.png)

---

## Herramientas utilizadas

| Herramienta | Uso |
|---|---|
| Python (pandas, numpy) | Generación y expansión del dataset sintético |
| Power BI Desktop | ETL en Power Query, modelado dimensional, DAX, visualización |
| DAX | Medidas de KPIs, inteligencia de tiempo, rankings |
| GitHub | Control de versiones y publicación del portafolio |

---

## Aprendizajes clave

- El parseo de fechas con múltiples formatos requiere lógica condicional explícita — Power Query no puede inferir el formato cuando hay ambigüedad entre DD/MM y MM/DD.
- La decisión sobre nulos no es técnica sino analítica: nulo y cero no significan lo mismo y tratarlos igual destruye métricas de promedio y denominador.
- Un modelo en estrella bien diseñado permite agregar nuevas métricas DAX sin tocar la estructura de datos — la inversión en modelado paga cada vez que se agrega un KPI.
- Las medidas de inteligencia de tiempo solo funcionan correctamente con una tabla calendario continua (sin gaps) conectada a la tabla de hechos.

---

## Estructura del repositorio

```
Nike-Sales-Dashboard-PowerBI/
├── Dataset/
│   ├── Nike_Sales_Expanded.csv
│   └── Nike_Sales_Cleaned.csv
├── Dashboard/
│   └── NIKE_DASHBOARD.pbix
├── Imagenes/
│   ├── Resumen_Ejecutivo.png
│   ├── Rentabilidad.png
│   ├── Geografico.png
│   └── Analisis_Temporal.png
└── README.md
```

---

*Proyecto desarrollado como parte de un portafolio de análisis de datos orientado a prácticas preprofesionales en planeamiento financiero y comercial.*
