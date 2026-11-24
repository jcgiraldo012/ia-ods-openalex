# Modelo de datos · IA y ODS (OpenAlex)

Es un **dataset**, no una base de datos: aquí se describe cómo llegan y cómo se organizan los datos que se traen de la API de OpenAlex. Todos los datos son públicos (licencia CC0) y **no se guardan nombres de autores**.

## Vista general

| Tabla | Qué representa una fila | Filas aprox. | De dónde sale |
|---|---|---|---|
| `articulos_colombia` | Un artículo de IA con al menos un autor de una institución colombiana | ~2.600 (2016–2025) | `/works`, con filtros |
| `articulo_ods` | Un par artículo–ODS (un artículo puede tener 0, 1 o varios ODS) | Por medir | La lista de ODS dentro de cada artículo |
| `ods` | Uno de los 17 Objetivos de Desarrollo Sostenible | 17 | Catálogo fijo de la ONU |
| `conteos_mundo` | Cuántos artículos de IA del mundo hay por año y por ODS | 170 (10 años × 17 ODS) | Consultas de conteo a `/works` |
| `totales_mundo` | Totales de artículos de IA del mundo por año | 10 | Consultas de conteo a `/works` |

## Cómo se relacionan

```
articulos_colombia ──(id_articulo)──< articulo_ods >──(id_ods)── ods ──(id_ods)──< conteos_mundo
                                                                                     │
                                                              totales_mundo ───(anio)┘
```

- `articulo_ods` conecta cada artículo colombiano con sus ODS, a través de `id_articulo`.
- `ods` es la pieza común: permite comparar Colombia y el mundo con los mismos 17 ODS.
- `conteos_mundo` y `totales_mundo` se unen por `anio`.
- **Ojo con el JOIN:** si un artículo tiene 3 ODS, al unir `articulos_colombia` con `articulo_ods` aparece 3 veces. Para contar artículos hay que contar `id_articulo` distintos, no filas.
- Los totales van en su propia tabla (`totales_mundo`) y no mezclados con el detalle por ODS. Así se evita que alguien los sume dos veces.

---

## 1. `articulos_colombia`
**Llave:** `id_articulo` (no se repite).

| Columna | Tipo | Qué significa | ¿Puede faltar? | Ejemplo |
|---|---|---|---|---|
| id_articulo | texto | Identificador único del artículo en OpenAlex | No | W4409746746 |
| titulo | texto | Título del artículo | No | Intelligent learning systems in primary and secondary education… |
| anio_publicacion | entero | Año de publicación (2016–2025) | No | 2025 |
| fecha_publicacion | fecha (AAAA-MM-DD) | Fecha de publicación | No | 2025-06-30 *(formato)* |
| tipo | texto | Tipo de publicación | No | article, review |
| idioma | texto | Código del idioma | Sí | en |
| es_acceso_abierto | booleano | Si se puede leer gratis | No | true |
| tipo_acceso_abierto | texto | Clase de acceso abierto (gold, green, hybrid, bronze, closed) | Sí | gold |
| num_autores | entero | Cuántos autores tiene (se cuentan; no se guardan los nombres) | No | 2 |
| tema_principal | texto | Tema principal asignado por OpenAlex | Sí | Intelligent Tutoring Systems and Adaptive Learning |

## 2. `articulo_ods`
**Llave:** `id_articulo` + `id_ods` juntos (el mismo par no se repite).
Los artículos sin ODS **no tienen filas** en esta tabla. Así se calcula la orientación: artículos que aparecen aquí ÷ total de artículos.

| Columna | Tipo | Qué significa | ¿Puede faltar? | Ejemplo |
|---|---|---|---|---|
| id_articulo | texto | Artículo al que pertenece (conecta con `articulos_colombia`) | No | W4409746746 |
| id_ods | entero (1–17) | Número del ODS; se toma del final de la dirección `https://metadata.un.org/sdg/3` | No | 3 |
| puntaje_ods | número decimal (0–1) | Qué tan seguro está el modelo de OpenAlex de esa clasificación | No | 0.624 |

## 3. `ods`
**Llave:** `id_ods`.

| Columna | Tipo | Qué significa | ¿Puede faltar? | Ejemplo |
|---|---|---|---|---|
| id_ods | entero (1–17) | Número del ODS | No | 3 |
| nombre_ods | texto | Nombre del ODS | No | Salud y bienestar |

## 4. `conteos_mundo`
**Llave:** `anio` + `id_ods` juntos.

| Columna | Tipo | Qué significa | ¿Puede faltar? | Ejemplo |
|---|---|---|---|---|
| anio | entero | Año de publicación (2016–2025) | No | 2025 |
| id_ods | entero (1–17) | Número del ODS (conecta con `ods`) | No | 3 |
| num_articulos | entero | Artículos de IA del mundo ligados a ese ODS ese año | No (0 si no hay) | 24848 |

## 5. `totales_mundo`
**Llave:** `anio`.

| Columna | Tipo | Qué significa | ¿Puede faltar? | Ejemplo |
|---|---|---|---|---|
| anio | entero | Año de publicación (2016–2025) | No | 2025 |
| total_articulos_ia | entero | Todos los artículos de IA del mundo ese año | No | 255740 |
| articulos_con_algun_ods | entero | Artículos con al menos un ODS (cada artículo cuenta una vez) | No | 64158 |

`articulos_con_algun_ods` no se obtiene sumando `conteos_mundo`, porque un artículo puede tener varios ODS y se contaría varias veces. Se pide aparte a la API.

---

## Salida para la directora (se calcula a partir de las tablas anteriores)
`resumen_kpi`: una fila por **año × ODS × ámbito** (Colombia o mundo) = 10 × 17 × 2 = **340 filas**, con los 3 KPI del brief (orientación, equilibrio y brecha).

## Lo esencial del contrato de datos
- **Frecuencia:** se actualiza el primer día de cada mes.
- **Responsable de la fuente:** OpenAlex (organización sin ánimo de lucro OurResearch).
- **Qué pasa si la fuente cambia** (por ejemplo, si OpenAlex cambia la clasificación de ODS o el formato de la API): el flujo compara las columnas recibidas con las esperadas y los conteos con los del mes anterior. Si algo no cuadra, falla, avisa y no publica el resumen.
- **Datos personales:** no se guardan nombres de autores, solo cuántos son.
