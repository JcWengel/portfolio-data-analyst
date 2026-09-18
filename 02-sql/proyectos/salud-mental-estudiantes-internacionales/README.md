# Salud mental y duración de la estancia en estudiantes internacionales
 
> **En una frase:** análisis en SQL de las puntuaciones de depresión, conexión social y estrés
> aculturativo de 201 estudiantes internacionales en Japón. El estrés aculturativo aumenta un
> 20,5 % entre el primer y el cuarto año de estancia, mientras la conexión social se deteriora
> en paralelo.
 
![Puntuaciones medias por duración de la estancia](./img/portada.jpg)
 ---
 ## Contexto y objetivo
 
La Universidad Ritsumeikan Asia Pacífico, un campus internacional situado en Japón, encuestó a
su estudiantado durante 2018 y publicó los resultados al año siguiente en un estudio aprobado
por su comité ético y conforme a la Declaración de Helsinki. Dicho estudio concluyó que el
alumnado internacional presenta un riesgo de dificultades de salud mental superior al de la
población general, con una prevalencia de depresión del 37,81 % frente al 29,85 %
correspondiente al alumnado nacional, y que tanto la conexión social como el estrés
aculturativo actúan como predictores de la depresión.
 
El punto interesante de este análisis reside en el camino que abre hacia la comprensión y la
posterior búsqueda de soluciones sobre la depresión y el aislamiento social del estudiantado,
con el fin de prevenir problemas crónicos y tendencias suicidas, presentes en torno al 20 % de
las respuestas recogidas. Entender estos indicadores y la interrelación que se estudia permite
la anticipación y la rápida intervención a futuro.
 
**Objetivo:** comprobar mediante SQL si los datos respaldan las conclusiones del estudio
original para el alumnado internacional, y determinar si la duración de la estancia constituye
un factor contribuyente.
 
**Preguntas que se pretende responder:**
 
1. ¿Cómo varían las puntuaciones de depresión, conexión social y estrés aculturativo en función de los años de estancia?
2. ¿Se observa una tendencia sostenida o el patrón responde a ruido estadístico?
3. ¿Qué implicaciones tiene la distribución de estudiantes entre duraciones sobre la fiabilidad de las medias?
---
## Datos
 
| | |
|---|---|
| **Fuente** | Encuesta web a estudiantes de la Universidad Ritsumeikan Asia Pacífico (Japón) |
| **Año de recogida** | 2018 |
| **Muestra total** | 268 registros (internacionales y nacionales) |
| **Población analizada** | 201 estudiantes internacionales (`inter_dom = 'Inter'`) |
| **Aprobación ética** | Comité ético de la universidad, conforme a la Declaración de Helsinki (WMA) |
| **Formato** | Tabla `students` en PostgreSQL |
 
### Variables utilizadas
 
| Columna | Descripción | Interpretación |
|---|---|---|
| `inter_dom` | Tipo de estudiante: internacional o nacional | Filtro del análisis |
| `stay` | Años de estancia en el país | Variable de agrupación |
| `todep` | Puntuación total del test PHQ-9 (depresión) | **Más alto = más síntomas depresivos**. Rango 0-27. De 5 a 9 se considera leve; de 10 a 14, moderado |
| `tosc` | Puntuación total del test SCS (conexión social) | **Más alto = mejor conexión social**. Rango 8-48 |
| `toas` | Puntuación total del test ASISS (estrés aculturativo) | **Más alto = más estrés de adaptación**. Rango 36-180 |
 
> Las tres escalas no apuntan en la misma dirección. En PHQ-9 y ASISS, una puntuación
> superior indica un peor estado. En SCS, indica un mejor estado.
---
## Metodología
 
Consulta de agregación sobre la tabla `students`, filtrando al alumnado internacional y
agrupando por años de estancia:
 
```sql
SELECT
    stay,
    COUNT(*) AS count_int,
    ROUND(AVG(todep), 2) AS average_phq,
    ROUND(AVG(tosc), 2) AS average_scs,
    ROUND(AVG(toas), 2) AS average_as
FROM students
WHERE inter_dom = 'Inter'
GROUP BY stay
ORDER BY stay DESC
LIMIT 9;
```
 
### Decisiones relevantes
 
**El filtro por tipo de estudiante se aplica en el `WHERE`, no dentro de un
`COUNT(CASE WHEN ...)`.** El estudio se centra en el alumnado internacional y las cinco
columnas solicitadas se refieren a ese grupo, por lo que el filtro debe alcanzar al conjunto de
la consulta y no únicamente a la columna de recuento.
 
Una primera aproximación consistió en contar los estudiantes internacionales mediante
`COUNT(CASE WHEN inter_dom = 'Inter' THEN 1 END)` sin filtrar la tabla. Esa construcción
devuelve un recuento correcto, pero las tres funciones `AVG` continúan operando sobre la
totalidad de registros, incluido el alumnado nacional. El resultado sería una tabla en la que
`count_int` corresponde a un subconjunto y las medias a otro distinto, sin que la consulta
arroje error alguno. Aplicar el filtro en el `WHERE` garantiza que las cinco columnas describan
la misma población.
 
**La columna `count_int` se incorpora como criterio de validez, no como dato accesorio.** Es la
variable que determina qué filas del resultado admiten interpretación y cuáles corresponden a
observaciones individuales.
 
---
## Resultados
 
| stay | count_int | average_phq | average_scs | average_as |
|---:|---:|---:|---:|---:|
| 10 | 1 | 13.00 | 32.00 | 50.00 |
| 8 | 1 | 10.00 | 44.00 | 65.00 |
| 7 | 1 | 4.00 | 48.00 | 45.00 |
| 6 | 3 | 6.00 | 38.00 | 58.67 |
| 5 | 1 | 0.00 | 34.00 | 91.00 |
| 4 | 14 | 8.57 | 33.93 | 87.71 |
| 3 | 46 | 9.09 | 37.13 | 78.00 |
| 2 | 39 | 8.28 | 37.08 | 77.67 |
| 1 | 95 | 7.48 | 38.11 | 72.80 |
 
### Hallazgo 1: la muestra se concentra casi por completo en los cuatro primeros años
 
De los 201 estudiantes internacionales, **194 (el 96,5 %) acumulan entre uno y cuatro años** de
estancia. Las duraciones de 5, 7, 8 y 10 años corresponden a **un único estudiante cada una**, y
la de 6 años a tres.
 
No se trata de un detalle metodológico menor: implica que las cinco primeras filas de la tabla
no constituyen promedios, sino respuestas individuales. El valor de 13,00 en PHQ-9 asociado a
la estancia de 10 años, que a primera vista sugeriría un deterioro grave a largo plazo,
corresponde a la puntuación de una sola persona. El valor de 0,00 a los 5 años, igualmente.
 
**Cualquier lectura de tendencia debe restringirse a los grupos de 1 a 4 años.**
 
### Hallazgo 2: el estrés aculturativo crece de forma sostenida con la estancia
 
Dentro de los grupos con tamaño muestral suficiente, el ASISS es la única de las tres escalas
que evoluciona de manera monótona:
 
| Años | ASISS medio | Variación acumulada |
|---:|---:|---:|
| 1 | 72.80 | — |
| 2 | 77.67 | +4.87 |
| 3 | 78.00 | +5.20 |
| 4 | 87.71 | **+14.91 (+20,5 %)** |
 
El estrés de adaptación no se resuelve con el transcurso del tiempo, sino que se acumula. El
incremento más pronunciado se produce en el cuarto año.
 
### Hallazgo 3: la conexión social se deteriora en el mismo periodo
 
El SCS sigue la trayectoria inversa, descendiendo de 38,11 en el primer año a 33,93 en el
cuarto, lo que supone 4,18 puntos menos y una caída del 11 %. El alumnado con estancias más
prolongadas reporta sentirse **menos** conectado socialmente, no más.
 
### Hallazgo 4: el máximo de sintomatología depresiva en el tercer año coincide con el estudio publicado
 
La puntuación PHQ-9 se mantiene en la franja leve en todos los grupos, entre 7,48 y 9,09, pero
su máximo entre los grupos con tamaño suficiente se sitúa en el **tercer año**, con 9,09.
 
El estudio original identificó precisamente la estancia de tres años como la única duración con
correlación estadísticamente significativa con la depresión entre el alumnado internacional
(β = 1,08, p = 0,032). El análisis descriptivo realizado aquí reproduce ese patrón de forma
independiente, lo que respalda que la agregación está capturando una señal real y no un
artefacto del muestreo.
 
---
## Conclusiones y recomendaciones
 
| Hallazgo | Implicación práctica |
|---|---|
| El estrés aculturativo aumenta un 20,5 % entre el primer y el cuarto año | El apoyo psicológico no debería concentrarse únicamente en la acogida inicial |
| La conexión social disminuye conforme se prolonga la estancia | Los programas de integración, dirigidos habitualmente a recién llegados, dejan fuera al alumnado veterano |
| El tercer año concentra el máximo de sintomatología depresiva | Constituye un punto de control natural para un cribado proactivo |
| El 96,5 % de la muestra se sitúa entre uno y cuatro años de estancia | Las conclusiones relativas a estancias largas carecen de respaldo suficiente en este estudio |
 
**Respuesta a la pregunta de partida:** los datos respaldan las conclusiones del estudio
original. La conexión social y el estrés aculturativo evolucionan de forma coherente con su
papel predictor de la depresión, y la duración de la estancia se confirma como factor
contribuyente, con el tercer año como punto crítico.
 
**Recomendación principal:** establecer un programa de seguimiento del alumnado internacional
que incorpore indicadores adicionales a los tres recogidos aquí, con el fin de comprender el
origen de estas puntuaciones y no únicamente su evolución. El objetivo sería brindar apoyo
psicológico en las etapas tempranas de estos problemas, de modo que las cifras no se agraven a
medida que la estancia se prolonga.
 
**Observación que merece investigación adicional:** resulta contraintuitivo que la conexión
social disminuya conforme aumenta la estancia, cuando lo esperable sería una consolidación
progresiva de las relaciones sociales. Una explicación plausible es que el grupo de referencia
del primer año lo constituye la propia cohorte de recién llegados, que comparte situación y se
apoya mutuamente; conforme dicha cohorte se dispersa, quien permanece pierde ese círculo sin
haberlo sustituido por vínculos locales. Los datos disponibles no permiten contrastar esta
hipótesis, pero sí señalan la dirección en la que convendría profundizar.
 
---
 
## Limitaciones
 
**Diseño transversal.** Los datos comparan a personas distintas en un mismo momento temporal,
no realizan un seguimiento de los mismos estudiantes a lo largo de su estancia. Para obtener un
análisis más fiable convendría repetir el estudio sobre una población continua en el tiempo,
con el fin de observar cómo evolucionan realmente estos indicadores en cada trayectoria
individual. Con el diseño actual no puede descartarse que las diferencias observadas entre
grupos respondan a un efecto de cohorte en lugar de a un deterioro progresivo.
 
**Sesgo de supervivencia.** Quienes peor se adaptan tienen mayor probabilidad de abandonar el
país antes de completar estancias prolongadas. El alumnado de mayor permanencia está compuesto,
por definición, por quienes permanecieron, circunstancia que podría estar atenuando la magnitud
real del efecto.
 
**Tamaños de grupo notablemente desiguales.** El primer año agrupa a 95 estudiantes frente a los
14 del cuarto. Los intervalos de confianza asociados a los grupos de estancia prolongada serían
considerablemente más amplios, lo que exige prudencia al comparar sus medias.
 
**Ausencia de contraste estadístico.** Las diferencias descritas son de naturaleza descriptiva.
Determinar su significación requeriría un ANOVA o pruebas por pares con corrección por
comparaciones múltiples, análisis que excede el alcance de esta consulta.
 
---
 
## 🛠️ Stack técnico
 
`SQL` · `PostgreSQL` · `Jupyter Notebook`
 
---
 
## Estructura
 
```
salud-mental-estudiantes-internacionales/
├── README.md
├── mental_health_analysis.ipynb
├── data/
│   └── students.csv
└── img/
    └── portada.jpg
```
 
---
 
## Referencias
 
**Estudio original**
 
- Nguyen, M. H., Le, T. T. & Meirmanov, S. (2019). *Depression, Acculturative Stress, and Social Connectedness among International University Students in Japan: A Statistical Investigation*. Sustainability, 11(3), artículo 878. https://doi.org/10.3390/su11030878
**Dataset**
 
- *A Dataset of Students' Mental Health and Help-Seeking Behaviors in a Multicultural Environment*. Data, 4(3), 124 (2019). https://www.mdpi.com/2306-5729/4/3/124
**Instrumentos de medida**
 
- PHQ-9: Kroenke, Spitzer & Williams (2001), *Patient Health Questionnaire-9*
- SCS: Lee & Robbins (1995), *Social Connectedness Scale*
- ASISS: Sandhu & Asrabadi (1994), *Acculturative Stress Scale for International Students*. Adaptado por los autores al contexto de un campus anglófono en Japón, manteniendo el rango total de 36 a 180
---
