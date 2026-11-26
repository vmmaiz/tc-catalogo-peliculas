# Catálogo de películas · Team Challenge

**Equipo:** Jon Gojenola Nolte, Luz Mery Clavijo Florez, Victor Eusebio Moreno Maiz

Catálogo de películas enriquecido a partir del dataset MovieLens: análisis con Pandas, sinopsis y web oficial desde la API de TMDB, y sinopsis en español generadas con Gemini.

## Reparto de funciones 

| Persona                     | Rol |  Notebook (nb) / tarea             | Rol transversal                         | Revisa las PR de     |
|---                          |---  |  ----------------                  | ---------------                         | ---------------      |
| Jon Gojenola Nolte          | B   | notebook 02 (apartados 3-4)        | Calidad del código y prueba de reproducibilidad | PRs de A / C |
| Luz Mery Clavijo Florez     | C   | notebooks 03 (TMDB) y 04 (Gemini)  | gestión de claves y .env.example.               | PRs de A / B |
| Victor Eusebio Moreno Maiz  | A   | nb 01 + nb 02 (apart 5) + bonus 4A | repo, ramas, README y PR final develop → main   | PRs de B / c |


## Estructura del Repositorio

```
tc-catalogo-peliculas/
├── 00_enunciado_general.ipynb     # enunciado (no se edita)
├── 01_ingesta_limpieza.ipynb
├── 02_analisis.ipynb
├── 03_tmdb.ipynb
├── 04_gemini.ipynb
├── data/
│   ├── movies.csv, ratings.csv, tags.csv, links.csv   # datos originales
│   └── processed/                                     # CSV generados por los notebooks
│       ├── movies_clean.csv
│       ├── movies10.csv
│       └── movies10_es.csv
├── README.md
├── requirements.txt
├── .env.example                    # plantilla de claves (sin valores)
└── .gitignore
```


## Instalación

Se intalarán las librerías indicadas en requirements.txt

```bash
git clone https://github.com/vmmaiz/tc-catalogo-peliculas.git
cd tc-catalogo-peliculas
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env            # y rellenad las claves
```
Después, editad .env con vuestras claves (ver siguiente apartado).
Al abrir los notebooks en Jupyter o VS Code, seleccionad como kernel el Python del entorno venv.


## Claves necesarias

| Variable                                  | Dónde se obtiene                |
|---                                        |---                              |
| `TMDB_READ_ACCESS_TOKEN` o `TMDB_API_KEY` | themoviedb.org → Settings → API |
| `GEMINI_API_KEY`                          | aistudio.google.com             |

Las claves van solo en el fichero .env, que está en el .gitignore y nunca se sube al repositorio.


## Ejecución

Ejecutad los notebooks **en este orden** (cada uno genera un CSV en `data/processed/` que usa el siguiente):

1. `01_ingesta_limpieza.ipynb` → `movies_clean.csv`
2. `02_analisis.ipynb`
3. `03_tmdb.ipynb` → `movies10.csv`
4. `04_gemini.ipynb` → `movies10_es.csv`

Si Gemini devuelve un error 404 / model not found, cambiad la constante MODEL al principio de 04_gemini.ipynb por un modelo vigente de la lista oficial.


## Flujo de Trabajo

- **main:** versión estable. Solo recibe la PR final desde develop (merge commit).
- **develop:** integración. Recibe cada tarea por pull request revisada por otro miembro (squash and merge).
- **feature/...:** Cada Persona crea su rama feature/... desde develop y empieza su notebook   

|Tarea	                     |Rama	                 |Responsable (equipo de 3)  |
|----                        |----                   |----------                 |
|Setup inicial del repo      |	feature/repo-setup   |	A                        |
|Notebook 01 · limpieza      |	feature/01-limpieza  |	A                        |
|Notebook 02 · apartados 3–4 |	feature/02-analisis  |	B                        |
|Notebook 02 · apartado 5    |	feature/02-preguntas |	A (tras mergear la de B) |
|Notebook 03 · TMDB          |	feature/03-tmdb      |	C                        |
|Notebook 04 · Gemini        |	feature/04-gemini    |	C                        |
|Bonus 4A                    |	feature/4a-catalogo  |	A (tras mergear la de C) |
|README final                | 	docs/readme-final    |	A                        |


## AVISOS PRUEBA REPRODUCIBILIDAD
Por el camino ha encontrado cuatro cosas:

1	VS Code abrió la **carpeta en modo restringido y no dejaba elegir el kernel**	Aviso de Restricted Mode → Trust 
2	Sin .env, el 03 pide la clave en una ventana   	
3	Sin .env, el 04 da No API key was provided	**fix: Pedir la clave con getpass**



## Resultados principales

### Notebook 01 · Ingesta y limpieza
- El dataset MovieLens contiene **9.742 películas**, **100.836 valoraciones**, **3.683 etiquetas** y **9.742 links**. Ninguna tabla tiene nulos salvo `links`.
- **8 películas no tienen `tmdbId`** en `links.csv`, así que **no se podrán consultar en TMDB** (notebook 03).
- El año se ha extraído del título con la regex `\((\d{4})\)\s*$`. Hay **13 películas sin año reconocible**, incluido el rango *Death Note (2006–2007)*. Siguiendo el enunciado, su `year` queda como `NaN`.
- **Esas 13 películas quedan fuera de cualquier análisis o filtro por año** (`groupby("year")`, décadas, `year >= 2000`…). Por eso los totales por año suman **9.729**, no 9.742.
- Para conservar el tipo entero con nulos, leed el fichero con `pd.read_csv("data/processed/movies_clean.csv", dtype={"year": "Int64"})`.

### Notebook 02 · Análisis con pandas (apartados 3-4)
 -  **Merge de tablas:** Cruce por `movieId` (`how="left"`, `validate="many_to_one"`) para construir `ratings_movies` y `tags_movies`, además de identificar las películas sin valoraciones
 -  **Agregaciones y segmentos:** Resumen por `userId` (cantidad de valoraciones y nota media) y segmentación en `"Muchas valoraciones"` y `"Pocas valoraciones"` a partir de la mediana de valoraciones por usuario

### 03 · TMDB → `movies10.csv`
- Necesita `TMDB_READ_ACCESS_TOKEN` o `TMDB_API_KEY` en el `.env`. Si no hay ninguna, la pide por teclado.
- Solo 4 de las 10 películas tienen `homepage`. Es normal: las que no tienen se guardan como cadena vacía.
- Si una petición falla, el notebook avisa y sigue. Revisad que no haya avisos en la salida antes de subir el CSV.
- El 04 lee este CSV, así que no necesita clave de TMDB.


### 04 · Gemini → `movies10_es.csv`
- Necesita `GEMINI_API_KEY` en el `.env`.
- Si sale `404 / model not found`, cambiad solo la constante `MODEL`.
- No controla los errores de la API: si Gemini falla o se supera el límite gratuito (error 429), el notebook se para. Esperad un poco y volved a ejecutarlo.
- El bonus 4A está sin hacer.