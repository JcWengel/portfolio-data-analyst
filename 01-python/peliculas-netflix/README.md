# Análisis exploratorio del cine de los 90 en el catálogo de Netflix
 
> **En una frase:** análisis en Python del catálogo de Netflix correspondiente a la década de
> 1990. La duración más frecuente se sitúa en 94 minutos, si bien el género de acción promedia
> 120,1 y únicamente 7 de sus 48 títulos descienden de los 90 minutos.
 
![Cine de los 90](./img/portada.jpg)
 
---
 
## Contexto y objetivo
 
El análisis se plantea desde el supuesto de una productora especializada en estilos nostálgicos
con interés en el cine estrenado durante la década de 1990. El conjunto de datos disponible
recoge el catálogo de Netflix, del que se extraen los títulos correspondientes a dicho periodo.
 
El interés de conocer la duración característica de aquellas películas reside en la posibilidad
de observar cómo ha evolucionado esta variable en relación con la atención del público, y de
determinar de qué manera se estructuraba la retención de dicha atención a lo largo del metraje.
El objetivo último consiste en valorar si aquel formato resulta adaptable a las exigencias del
mercado actual o si, por el contrario, requiere una reformulación para ajustarse a los
requerimientos del consumo audiovisual contemporáneo.
 
**Objetivo técnico:** determinar la duración más frecuente de las películas estrenadas entre
1990 y 1999, y cuantificar cuántas de las pertenecientes al género de acción no superan los 90
minutos.
 
**Preguntas que se pretende responder:**
 
1. ¿Cuál es la duración más frecuente entre las películas de la década de 1990 recogidas en el catálogo?
2. ¿Cuántas películas de acción de ese periodo tienen una duración igual o inferior a 90 minutos?
3. ¿Resulta la duración una característica homogénea de la década, o depende del género?
---
 
##  Datos
 
| | |
|---|---|
| **Fuente** | `netflix_data.csv` — catálogo de títulos disponibles en la plataforma |
| **Registros totales** | 4.812 títulos |
| **Subconjunto analizado** | 183 películas estrenadas entre 1990 y 1999 |
| **Formato** | CSV |
 
### Variables utilizadas
 
| Columna | Descripción |
|---|---|
| `type` | Tipo de título: película o serie |
| `title` | Título de la obra |
| `release_year` | Año de estreno |
| `duration` | Duración en minutos |
| `genre` | Género asignado al título |
| `country` | País de origen |
 
>  La columna `duration` expresa minutos únicamente en el caso de las películas. Para las
> series recoge el número de temporadas, motivo por el cual el filtrado por `type == 'Movie'`
> resulta imprescindible antes de cualquier cálculo sobre esta variable.
 
---
 
## Metodología
 
El análisis se estructura en tres pasos: filtrado del subconjunto correspondiente a la década,
obtención de la duración más frecuente y recuento de títulos de acción por debajo de un umbral
de metraje.
 
```python
import pandas as pd
import matplotlib.pyplot as plt
 
netflix_df = pd.read_csv("netflix_data.csv")
 
# Filtrado: películas estrenadas entre 1990 y 1999
filtered_netflix = netflix_df.query("1990 <= release_year < 2000 and type == 'Movie'")
 
# Duración más frecuente
duration = int(filtered_netflix['duration'].mode()[0])
 
# Recuento de películas de acción de 90 minutos o menos
action_movies = filtered_netflix.query("genre == 'Action'")
short_movie_count = (action_movies['duration'] <= 90).sum()
```
 
### Decisiones relevantes
 
**El filtrado se resuelve mediante `query` en lugar de encadenar máscaras booleanas.** La
sintaxis de `query` permite expresar la condición compuesta de año y tipo en una única expresión
legible, lo que facilita la comprensión del criterio aplicado sin necesidad de descomponerlo en
varias líneas.
 
**Sustitución de un bucle por una operación vectorizada.** El recuento de películas de acción de
90 minutos o menos se resolvió inicialmente mediante un bucle `for` que recorría la columna de
duraciones incrementando un contador, aproximación procedente de una lógica de programación
imperativa previa. El resultado obtenido es correcto, pero pandas permite resolver la misma
operación mediante una máscara booleana, `(action_movies['duration'] <= 90).sum()`, que evalúa
la condición sobre la totalidad de la columna en una sola operación. La versión vectorizada
resulta más concisa y escala mejor ante conjuntos de datos de mayor tamaño. Se conserva aquí la
mención al planteamiento original por constituir uno de los aprendizajes derivados del
ejercicio: el tránsito desde el razonamiento iterativo hacia el pensamiento en términos de
columnas completas.
 
---
 
## Resultados
 
### Medidas de posición del conjunto
 
| Medida | Valor (minutos) |
|---|---:|
| Moda | 94 |
| Mediana | 108 |
| Media | 115,1 |
| Mínimo | 28 |
| Máximo | 195 |
 
### Hallazgo 1: la moda y la media difieren en 21 minutos
 
La duración más frecuente asciende a 94 minutos, mientras que la media se sitúa en 115,1. La
relación entre ambas medidas, junto con una mediana intermedia de 108 minutos, revela una
distribución sesgada hacia la derecha: existe una cola de títulos de metraje prolongado, con un
máximo de 195 minutos, que desplaza la media al alza sin que ello refleje el comportamiento
mayoritario del conjunto.
 
La consecuencia práctica es que ambas medidas responden a preguntas distintas. La moda indica
la duración más habitual; la media, el promedio de una distribución desequilibrada. Adoptar una
u otra como referencia conduce a decisiones de producción divergentes.
 
### Hallazgo 2: únicamente 7 de 48 películas de acción descienden de los 90 minutos
 
De los 48 títulos del género de acción estrenados durante la década, **7 presentan una duración
igual o inferior a 90 minutos**, lo que representa un 14,6 % del total del género. La duración
media del género asciende a 120,1 minutos.
 
El dato resulta contraintuitivo en la medida en que la acción constituye el género asociado a un
ritmo narrativo más acelerado, circunstancia que no se traduce en un metraje reducido.
 
### Hallazgo 3: la duración depende del género antes que de la década
 
| Género | Duración media | Títulos |
|---|---:|---:|
| Stand-Up | 53,2 | 8 |
| Children | 93,1 | 15 |
| Comedies | 110,7 | 40 |
| Action | 120,1 | 48 |
| Classic Movies | 128,7 | 15 |
| Dramas | 132,0 | 44 |
 
La diferencia entre el género de menor y mayor duración media alcanza los 79 minutos. La
pregunta relativa a la duración del cine de los noventa carece, por tanto, de una respuesta
única: la variable determinante es el género y no el periodo.
 
Conviene señalar que el género de acción no ocupa la primera posición en cuanto a extensión.
Tanto Dramas como Classic Movies lo superan. Lo destacable no reside en que la acción sea el
género más largo, sino en que, siendo el de ritmo más acelerado, tampoco resulte breve.
 
### Distribución por país de origen
 
| País | Títulos |
|---|---:|
| Estados Unidos | 99 |
| India | 34 |
| Reino Unido | 17 |
| Hong Kong | 11 |
| Francia | 5 |
| Australia | 5 |
 
---
 
## Conclusiones y recomendaciones
 
| Hallazgo | Implicación práctica |
|---|---|
| La duración más frecuente se sitúa en 94 minutos, frente a una media de 115,1 | La distribución presenta sesgo hacia la derecha. La moda representa mejor el metraje habitual que la media |
| El género de acción promedia 120,1 minutos, y únicamente 7 de 48 títulos no superan los 90 | El ritmo narrativo acelerado propio del género no se traduce en un metraje reducido |
| La diferencia entre géneros alcanza los 79 minutos | La duración no constituye una característica homogénea de la década, sino que depende del género |
 
**Respuesta a las preguntas planteadas:** la duración más frecuente entre las películas de la
década de 1990 recogidas en el catálogo es de **94 minutos**, y **7 películas** del género de
acción presentan una duración igual o inferior a 90 minutos.
 
**Observación de mayor interés:** el resultado más llamativo del análisis reside en el
comportamiento del género de acción. Pese a tratarse del género asociado a un ritmo narrativo
más acelerado, sus títulos no son breves: promedian 120,1 minutos y apenas un 15 % de ellos
desciende de los 90. La rapidez del género se manifiesta en la cadencia interna de las escenas y
no en la extensión total de la obra, circunstancia que conviene tener presente a la hora de
plantear una producción que aspire a reproducir aquella estética.
 
**Recomendación para una producción de estilo noventero:** tomar la duración como una decisión
derivada del género y no de la década. Un proyecto de acción ambientado en aquel periodo debería
situarse en el entorno de los 120 minutos, mientras que una comedia se ajustaría mejor a los 110
y una propuesta infantil a los 93. Adoptar la moda global de 94 minutos como referencia
universal supondría acortar en exceso los géneros de metraje más extenso.
 
---
 
## Limitaciones
 
**El catálogo de Netflix no equivale al cine de la década.** Los 183 títulos analizados
corresponden a las películas de los años noventa que la plataforma mantiene licenciadas en la
actualidad, no a la producción cinematográfica de aquel periodo. El catálogo responde a
criterios comerciales y de disponibilidad de derechos, de modo que tiende a conservar los
títulos que han mantenido demanda con el paso del tiempo. Las conclusiones describen, en rigor,
el cine noventero que ha sobrevivido en el catálogo.
 
**Ausencia de indicadores de atención o recepción.** El conjunto de datos recoge duración,
género, año y país, pero no incorpora variable alguna relativa a audiencia, retención,
valoración o comportamiento del espectador. La relación entre metraje y capacidad de retención
de la atención, planteada en el contexto del análisis, constituye una hipótesis de partida y no
un resultado que estos datos permitan contrastar.
 
**Asignación de un único género por título.** Cada registro tiene asignado un solo género,
cuando la mayoría de las obras admite varias clasificaciones simultáneas. Las medias por género
resultan sensibles a este criterio de asignación.
 
**Tamaños muestrales reducidos en determinados géneros.** Los géneros minoritarios cuentan con
un número de títulos escaso, como los 8 de Stand-Up o los 4 de Horror Movies, lo que resta
fiabilidad a sus medias. La categoría Stand-Up, además, agrupa espectáculos grabados cuya
naturaleza difiere de la del largometraje convencional, circunstancia que explica su media de
53,2 minutos y aconseja tratarla al margen.
 
---
 
## Próximos pasos
 
- Profundizar en el género de acción de manera específica, dado que constituye el hallazgo de mayor interés del análisis y el punto de partida idóneo para una producción nostálgica ambientada en la época.
- Incorporar datos de recepción y audiencia procedentes de fuentes externas, con el fin de contrastar la hipótesis relativa a la relación entre duración y retención de la atención.
- Comparar la duración del cine de los noventa con la de décadas posteriores, con objeto de determinar la dirección y magnitud de la evolución del metraje.
---
 
## Stack técnico
 
`Python` · `pandas` · `Matplotlib` · `Jupyter Notebook`
 
---
 
## Estructura
 
```
peliculas-netflix/
├── README.md
├── notebooks/
│   └── analisis-netflix.ipynb
├── data/
│   └── netflix_data.csv
└── img/
```
 
