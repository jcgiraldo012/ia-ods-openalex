# Brief · ¿La investigación en IA apunta a los problemas humanos?

## Motivación
¿Estamos haciendo un buen uso de la IA para el avance de la humanidad? Los datos no pueden decidir qué es «buen uso». Por eso este proyecto declara un criterio y lo mide: **buen uso = que la investigación en IA apunte a problemas humanos**, representados por los 17 Objetivos de Desarrollo Sostenible (ODS) de la ONU.

## Pregunta
¿El mundo y Colombia están realmente investigando la IA para responder a las problemáticas humanas (ODS)? ¿Y cómo ha cambiado desde 2016 hasta 2025?

## Quién decide
La **directora de investigación de una universidad colombiana** decide **cada año** cómo repartir el presupuesto de su convocatoria de proyectos de IA entre los ODS, comparando lo que investiga Colombia con lo que investiga el mundo.

## Fuente
- **API de OpenAlex** (`api.openalex.org`, endpoint `/works`): catálogo abierto de publicaciones científicas, con licencia CC0.
- **Filtros:** tema principal en el subcampo «Inteligencia artificial»; años 2016–2025; para Colombia, al menos una institución de autor con país CO.
- **Acceso:** requiere una llave gratuita, que va en un archivo `.env` y nunca se sube al repositorio. Paginación por cursor (hasta 100 registros por página) y presupuesto diario de uso.

## Alcance (qué sí incluye)
- **Colombia, en detalle:** un registro por artículo de IA con al menos un autor de una institución colombiana, 2016–2025 (~2.600 artículos).
- **El mundo, como contexto:** conteos de artículos de IA por año y por ODS, 2016–2025. No se descargan los artículos uno por uno, porque son más de 1,3 millones.

## Indicadores (KPI)
1. **Orientación (indicador principal de buen uso):** % de artículos de IA ligados a al menos un ODS, por año, en Colombia y en el mundo.
2. **Equilibrio:** cómo se reparte la investigación entre los 17 ODS. ¿Se concentra en pocos?
3. **Brecha:** diferencia, por ODS, entre la participación en Colombia y en el mundo.

Equilibrio y brecha **describen y dan contexto, pero no juzgan**: cada país tiene problemas distintos, así que una brecha frente al mundo no significa un mal uso.

## Destino
**BigQuery**, con tres tablas:
- artículos de Colombia;
- conteos del mundo por año y ODS;
- un resumen con los 3 KPI por año, ODS y ámbito (Colombia y mundo), que es lo que consulta la directora.

## Frecuencia
- El flujo corre **el primer día de cada mes**.
- La directora consulta el resumen **una vez al año**, antes de abrir su convocatoria.
- **Por qué mensual si la decisión es anual:** las cifras de cada año se siguen consolidando durante meses, porque los artículos aparecen en la base después de publicados y sus datos (instituciones, ODS) se completan con el tiempo. Actualizar cada mes mejora la precisión de las cifras de años anteriores. Además, si una corrida falla, siempre hay un corte reciente y correcto disponible para la revisión anual, y el problema se detecta con meses de anticipación.

## Cómo sé que terminó bien
1. **Reproducible:** dos corridas seguidas el mismo día dan los mismos conteos.
2. **Cuadra con la fuente:** los artículos de Colombia descargados por año coinciden con el total que reporta OpenAlex para ese año, con una diferencia máxima del 1 %. Ese margen existe porque OpenAlex agrega artículos entre consultas, y se ajustará con las primeras corridas.
3. **Forma esperada:** el resumen tiene 340 filas (10 años × 17 ODS × 2 ámbitos, contando los ceros).
4. **Falla y avisa:** si un año llega vacío o aparece un año mayor que 2025, el flujo se detiene, avisa y no publica el resumen.

## Qué NO incluye
- No mide el impacto real de la IA en los ODS. Mide hacia dónde apunta la investigación, no si mejoró la vida de las personas.
- No analiza los subtemas técnicos de cada artículo, solo los ODS.
- No incluye artículos que usan IA pero cuyo tema principal está en otra área (por ejemplo, medicina): solo cuentan los artículos cuyo tema principal pertenece al subcampo «Inteligencia artificial».
- No incluye 2026, porque es un año incompleto.
- No guarda datos de personas (nombres de autores) ni analiza empresas.
- No juzga el «buen uso» de un país comparándolo con el mundo. Comparar a Colombia con sus propias prioridades requeriría otra fuente.
- No decide por la directora: los datos informan su decisión, la estrategia es suya.
- No incluye otras fuentes, modelos de predicción ni tableros complejos.

## Riesgos de calidad
- **Clasificación automática:** los ODS los asigna un modelo de OpenAlex; no se verifican artículo por artículo.
- **Artículo sin ODS no significa artículo sin problema humano:** el modelo de clasificación automática de OpenAlex puede no haberlo etiquetado. Esto afecta al indicador de orientación.
- **Fechas en el futuro:** hay artículos con año de publicación posterior al actual (por ejemplo, 2027).
- **Artículos con muchos autores:** un artículo internacional con un solo autor colombiano cuenta como «colombiano». Si se cruza sin cuidado con la tabla de autores, se repite y los totales se inflan.
- **Afiliaciones incompletas y retraso de publicación:** algunos autores no tienen institución registrada, y los años recientes siguen sumando artículos que se indexan tarde.

## Uso de IA
Usé Claude (IA) para explorar y verificar fuentes de datos, comparar opciones y estructurar este brief. La pregunta, el usuario, el alcance, los criterios y su justificación son decisiones mías, y puedo explicarlas.
