# 🚇 ¿Cómo afectan los tarifazos a la cantidad de pasajeros del Subte?

**Trabajo Práctico Final — Introducción a la Ciencia de Datos**  
**Carrera:** Licenciatura en Ciencia de Datos  
**Grupo 7:** Apicella, De Carli, Stacchino

---

## 📌 Descripción

Este proyecto analiza el impacto de los aumentos tarifarios del Subte de Buenos Aires sobre la cantidad de pasajeros, usando datos desde enero de 2020 hasta septiembre de 2025. A través de análisis exploratorio, limpieza de datos y modelos de regresión lineal, buscamos responder:

> **¿Los aumentos en el precio del pasaje reducen efectivamente el uso del subte?**

📊 **[Ver presentación completa](presentacion/presentacion_final.pdf)**

---

## 🗂️ Estructura del repositorio

```
📦 subte-tarifazos/
├── 📄 README.md
├── 📊 analisis_subte_tarifazos.Rmd   # Código principal del análisis
├── 📁 data/
│   ├── tarifas_ipc.csv               # Tarifas históricas + IPC + RIPTE
│   ├── tabladias.csv                 # Tabla auxiliar de fechas y días
│   ├── dat-ab-usos-2020.csv          # Datos SUBE 2020
│   ├── dat-ab-usos-2021.csv          # Datos SUBE 2021
│   ├── dat-ab-usos-2022.csv          # Datos SUBE 2022
│   ├── dat-ab-usos-2023.csv          # Datos SUBE 2023
│   ├── dat-ab-usos-2024.csv          # Datos SUBE 2024
│   └── dat-ab-usos-2025.csv          # Datos SUBE 2025
└── 📁 presentacion/
    └── presentacion_final.pdf        # Slides de la presentación
```

> **Nota:** Los archivos de datos SUBE son de gran tamaño. Se recomienda descargarlos directamente desde [datos.gob.ar](https://datos.gob.ar) (ver sección Fuentes).

---

## 📊 Dataset

El dataset final combina información de múltiples fuentes y contiene:

| Característica | Detalle |
|---|---|
| **Observaciones** | 850.643 |
| **Variables** | 29 (5 categóricas, 24 numéricas) |
| **Período** | Enero 2020 – Septiembre 2025 |
| **Medios de transporte** | Subte, Colectivo, Tren |
| **Granularidad original** | Cantidad de viajes por día por línea |

### Variables clave

- `cantidad`: número de viajes diarios por línea
- `tarifa_subte_registrada`: precio del pasaje para usuario con SUBE registrada
- `sueldo_tarifa_subte_priv`: **Sueldo/Tarifa** — cuántos pasajes puede comprar un trabajador con su salario mensual promedio
- `LINEA`: línea de subte (A, B, C, D, E, H)
- `periodomes`: año-mes (clave de unión entre datasets)

---

## 🧹 Limpieza y preprocesamiento

El análisis requirió una limpieza exhaustiva del dataset crudo:

1. **Datos duplicados Nov/Dic 2021**: al cambiar la concesión del subte de Metrovías a Emova el 30/11/2021, los registros de esos meses aparecían duplicados. Se resolvió conservando Metrovías para noviembre y Emova para diciembre.

2. **Exclusión de líneas sin significancia**: Premetro, Lin_Amarilla_C y Lin_Verde_D presentaban valores muy bajos e introducían dispersión innecesaria.

3. **Exclusión de fines de semana**: sábados y domingos tienen patrones de viaje muy distintos a los días hábiles.

4. **Exclusión de meses de verano**: enero, febrero y diciembre presentan caídas sistemáticas por vacaciones, no relacionadas con las tarifas.

5. **Filtro post-pandemia**: se excluyeron los períodos 2020/03 – 2022/03 por el impacto del COVID-19 en el transporte público.

---

## 📈 Análisis exploratorio

Se identificaron varios patrones relevantes:

- **Caída sostenida desde mediados de 2024**, coincidiendo con los mayores aumentos tarifarios.
- **Relación positiva entre Sueldo/Tarifa y cantidad de viajes**: a mayor poder adquisitivo en términos de pasajes, más viajes se realizan.
- **La caída del subte no se compensa con otros transportes**: colectivo y tren muestran patrones proporcionales al subte, lo que descarta una migración hacia otros medios.

La tarifa del subte pasó de **$30 (enero 2020) a $1.112 (septiembre 2025)**, un aumento del **3.706%**, superando incluso la inflación acumulada del período (3.209%).

---

## 🤖 Modelado

Se entrenaron modelos de regresión lineal sobre un dataframe agrupado por **año-mes y línea**, donde cada observación representa la cantidad total de viajes de una línea en un mes dado.

### Modelos evaluados

| Modelo | Fórmula | R² | Res. Df |
|--------|---------|-----|---------|
| Modelo 1 | `cantidad ~ Año` | — | 184 |
| Modelo 2 | `cantidad ~ Año + Sueldo/tarifa` | — | 183 |
| Modelo 3 | `cantidad ~ Año + Sueldo/tarifa + Linea` | — | 178 |
| Modelo 4 | `cantidad ~ Año * Sueldo/tarifa + Linea` | — | 177 |
| **Modelo 5** | **`cantidad ~ (Año + Sueldo/tarifa) * Linea`** | **0.9195** | **168** |
| Modelo 6 | `cantidad ~ (Año * Sueldo/tarifa) * Linea` | 0.9349 | 162 |
| **Modelo 5 polinómico** | **`cantidad ~ (poly(Año,2) + poly(Sueldo/tarifa,2)) * Linea`** | **0.9409** | **156** |

La comparación mediante **ANOVA** confirmó que cada incremento en complejidad fue estadísticamente significativo (p < 0.05).

### Modelo final seleccionado

El **Modelo 5 polinómico** fue elegido por su balance entre ajuste (R² = 0.9409) y parsimonia. Al incluir términos de segundo grado, captura la curvatura observada en los residuos del Modelo 5 lineal.

---

## 🔮 Predicciones

Estimación de viajes para **noviembre de 2025**, considerando:
- Aumento de tarifa: $1.112 → $1.157 (+4%)
- Variación de Sueldo/Tarifa: 1.459 → 1.452 pasajes (-0.5%)

| Línea | Variación estimada |
|-------|--------------------|
| A | -0.010% |
| B | -0.033% |
| C | -0.009% |
| D | -0.033% |
| E | +0.008% |
| H | -0.012% |
| **Total** | **-0.029%** |

---

## 🧾 Conclusiones

- Los aumentos tarifarios **sí reducen** la cantidad de viajes en subte, pero el impacto es moderado: los usuarios reducen el uso, pero no abandonan el servicio.
- La variable **Sueldo/Tarifa** (poder adquisitivo en términos de pasajes) es el predictor más relevante para explicar las variaciones en el uso.
- Las **políticas de descuento** mediante bancos y billeteras virtuales probablemente mitigaron una caída más abrupta.
- La caída en el uso del subte **no se transfirió a otros medios de transporte**, lo que sugiere que parte de los viajes simplemente se dejaron de realizar.

---

## 🛠️ Tecnologías utilizadas

- **Lenguaje:** R
- **Paquetes principales:** `tidyverse`, `lubridate`, `ggplot2`, `modelr`, `scales`, `ggridges`, `zoo`

---

## 📥 Fuentes de datos

| Fuente | Descripción | Link |
|--------|-------------|------|
| datos.gob.ar | Dataset SUBE (usos diarios por línea) | [datos.gob.ar](https://datos.gob.ar/dataset/transporte-usos-sube) |
| INDEC | IPC (Índice de Precios al Consumidor) | [indec.gob.ar](https://www.indec.gob.ar/indec/web/Nivel4-Tema-3-5-31) |
| MTEySS | RIPTE (Remuneración Imponible Promedio) | [argentina.gob.ar](https://www.argentina.gob.ar/trabajo/seguridadsocial/ripte) |
| Medios periodísticos y boletines oficiales | Histórico de tarifas del subte | — |

---

## ▶️ Cómo ejecutar el análisis

1. Clonar el repositorio:
   ```bash
   git clone https://github.com/tu-usuario/subte-tarifazos-icd.git
   cd subte-tarifazos-icd
   ```

2. Descargar los archivos de datos SUBE desde [datos.gob.ar](https://datos.gob.ar/dataset/transporte-usos-sube) y colocarlos en la carpeta `data/`.

3. Abrir `Exploracion_clean_original.R` en RStudio e instalar las dependencias:
   ```r
   install.packages(c("tidyverse", "lubridate", "scales", "ggridges", "modelr", "zoo"))
   ```

4. Ajustar las rutas de lectura de archivos en las primeras líneas del script si es necesario y ejecutar.

---

*Trabajo realizado para la materia Introducción a la Ciencia de Datos — Facultad de Ciencias Exactas y Naturales, UBA.*
