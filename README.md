# Spotify Listening Behavior Analysis 🎧📊

> Proyecto de ETL → MongoDB → Power BI que analiza **756 k** reproducciones de Spotify.
> Paleta, tipografía y UX alineadas a la marca - cumple criterios de claridad (Yusef Hassan) y accesibilidad (WCAG AA).

---

## 📂 Estructura del repositorio

```
.
├─ data/                       # JSON originales (Spotify Extended Streaming History)
├─ powerbi/                    # Todos los archivos correspondientes al desarrollo de dashboard
├─ 01_etl_spotify.ipynb        # Flujo E-T-L → MongoDB
├─ spotify_theme.json          # Tema Power BI (colores + Segoe UI)
├─ requirements.txt
├─ .env.example                # Plantilla credenciales
├─ .gitignore
└─ README.md
```

---

## ⚙️ Instalación rápida

```bash
# 1. Clona y entra
git clone https://github.com/tu_usuario/spotify.git
cd spotify

# 2. Crea entorno virtual
python -m venv .venv
# PowerShell
.venv\Scripts\Activate.ps1
# Bash
source .venv/bin/activate

# 3. Instala dependencias
pip install -r requirements.txt

# 4. Copia .env y ajusta tu URI
cp .env.example .env
```

---

## 🔑 Variables de entorno

| Variable    | Ejemplo                     | Uso                      |
| ----------- | --------------------------- | ------------------------ |
| `MONGO_URI` | `mongodb://localhost:27017` | Conexión local o Atlas   |
| `DB_NAME`   | `spotify`                   | Base Mongo               |
| `COLL_NAME` | `extended_streams`          | Colección destino        |
| `DATA_ROOT` | `data`                      | Carpeta raíz de los JSON |

---

## 🔄 Flujo ETL (notebook)

1. **`notebooks/01_etl_spotify.ipynb`**
2. Ejecuta celdas → une JSON → convierte `ts` a Date → agrega `user` → inserta 756 k docs.
3. Índice recomendado:

```python
coll.create_index([("user", 1), ("ts", 1)], name="idx_user_ts")
```

4. **Vista sin IP**

```javascript
db.createView(
  "extended_streams_public",
  "extended_streams",
  [{ $project: { ip_addr: 0 } }]
)
```

---

## 🗄️ Conexión Power BI (Mongo local)

1. Instala **MongoDB ODBC Driver** 64-bit → crea DSN **MongoLocal**

   * Server: `localhost` • Port: `27017` • Database: `spotify`
2. **Inicio → Obtener datos → ODBC** → DSN *MongoLocal*
3. Carga vista **extended\_streams\_public** (modo *Import*).
4. Power Query:

   * `ts` → Date/Time
   * `minutes` = `ms_played / 60000`
   * `HourSlot` (franjas 2 h) – ver M-code en notebook.


---

## 📐 Medidas DAX utilizadas

| Medida         | Cálculo simplificado                        | Descripción                                                               |
| -------------- | ------------------------------------------- | ------------------------------------------------------------------------- |
| `total_plays`  | `COUNTROWS(streams)`                        | Total de reproducciones registradas.                                      |
| `minutes`      | `SUM(ms_played)`                            | Total de minutos escuchados (originalmente en milisegundos).              |
| `tracks`       | `DISTINCTCOUNTNOBLANK(track_name)`          | Número de canciones únicas escuchadas.                                    |
| `artists`      | `DISTINCTCOUNTNOBLANK(album_artist_name)`   | Número de artistas únicos.                                                |
| `albums`       | `DISTINCTCOUNTNOBLANK(album_name)`          | Número de álbumes únicos.                                                 |
| `shuffle_rate` | `DIVIDE(repros en shuffle, total_plays)`    | Porcentaje de canciones escuchadas en modo aleatorio (shuffle).           |
| `skip_rate`    | `DIVIDE(repros saltadas, total_plays)`      | Porcentaje de canciones que fueron saltadas por el usuario.               |
| `min per play` | `DIVIDE(minutes, total_plays)`              | Promedio de minutos escuchados por reproducción.                          |
| `Title`        | `"⏳ Monthly Listening Time per User (Año)"` | Texto dinámico que muestra el título del gráfico con el año seleccionado. |

---

---

## 🎨 Visuales y CX

| Visual                  | Campos                                                          | Insight / Criterio UX          |
| ----------------------- | --------------------------------------------------------------- | ------------------------------ |
| **Tarjetas KPI**        | 5 medidas (Plays, Horas, Usuarios, Shuffle %, Skip %)           | 5 s snapshot; jerarquía visual |
| **Columnas apiladas**   | Eje `YearMonth`, Leyenda `user`, Valor `[Total Minutes]`        | Tendencia mensual por usuario  |
| **Heat-map Matrix**     | Rows `HourSlot`, Columns `Weekday`, Values `[Total Minutes]`    | Hábitos circadianos            |
| **Tabla Top 10 Tracks** | `master_metadata_track_name`, `[Total Plays]` + barras internas | Descubrimiento de canciones    |
| **Barras horizontales** | Motivos `reason_end`, `[Total Plays]`                           | Por qué se detienen las pistas |

*Tema*: importa `spotify_theme.json` (verde Spotify, Segoe UI, fondos gris claro).


```

