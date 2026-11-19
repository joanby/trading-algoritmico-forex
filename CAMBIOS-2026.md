# Cambios de la rama `update-2026`

> Esta rama es el mismo curso —Estrategias Avanzadas de Trading Algorítmico aplicadas al mundo Forex 2023—, con el código adaptado a las librerías de hoy
> (matplotlib 3.11.2, pandas 3.0.6, yfinance 1.7.0; octubre de 2026). La rama principal sigue exactamente como en el vídeo.

> **Qué está comprobado y qué no.** Se ha ejecutado cada notebook entero con las versiones de
> `requirements.txt`: **7 correctos · 0 con fallo · 0 con timeout · 1 omitidos**. Comprobado: que cada celda se ejecuta sin error y que los datos
> tienen la forma del vídeo (histórico completo, mismas columnas, mismo orden). **No comprobado:** la
> descarga real desde Yahoo Finance, que no era accesible desde el entorno de prueba; se usó una réplica
> de su respuesta con precios inventados. **Tus números saldrán distintos** a los del vídeo, porque los
> precios reales han seguido moviéndose.
> Sin ejecutar: `ES_FX_Capítulo_07_MT5_Live_Trading` (MetaTrader 5 solo funciona en Windows, con el terminal y una cuenta).
> En noviembre de 2026 se corrigieron además errores del propio código del curso, que dan resultados
> distintos a los del vídeo: ver [Correcciones de errores del curso](#correcciones-de-errores-del-curso-noviembre-de-2026).

## Cómo usarla

- **En Google Colab (como en el vídeo):** abre el notebook de esta rama y, cuando el código lea un CSV,
  súbelo al panel de archivos igual que en el vídeo.
- **En tu ordenador:** `git clone -b update-2026 https://github.com/joanby/trading-algoritmico-forex`, instala `pip install -r requirements.txt`
  (versiones **fijadas**: las mismas con las que se ha comprobado, para que un cambio futuro de las
  librerías no lo vuelva a romper) y deja junto al notebook los CSV que use.

## Qué ha cambiado y por qué

### 1. Descargar precios con yfinance

Notebooks: `BONUS_ES_ML_Capítulo_07_SVR_para_predecir_stocks`, `ES_FX_Capítulo_04_Backtesting_Vectorizado`, `ES_FX_Capítulo_05_Estrategias_avanzadas_para_Forex`.

yfinance cambió tres comportamientos por defecto de `yf.download` desde que se grabó el curso
(comprobado leyendo yfinance 0.1.70, la versión de entonces, y la 1.7.0 de hoy):

| En el vídeo | Hoy, si no dices nada | Qué pasa con el código del curso |
|---|---|---|
| Sin fechas, descarga **todo el histórico** | Descarga **solo el último mes** | Medias largas vacías, `.loc["2020"]` da `KeyError`, backtests de un mes |
| Con solo `end="2021-01-01"`, desde el principio hasta esa fecha | **Solo el mes anterior** a esa fecha | El mismo problema, sin ningún error |
| Columnas `Open, High, Low, Close, Adj Close, Volume` | Sin `Adj Close` (`auto_adjust=True`) | `KeyError: 'Adj Close'` y *Length mismatch* al renombrar |
| Columnas simples, en ese orden | Dos niveles (precio, ticker) y en **orden alfabético** | Aunque arregles lo anterior, al renombrar por posición `open` acabaría siendo `Adj Close` |

Cada `yf.download(...)` lleva ahora los argumentos que devuelven el comportamiento del vídeo:

```python
yf.download("EURUSD=X", period="max", auto_adjust=False, multi_level_index=False)
```

`period="max"` se añade siempre que la llamada no tenga fecha de inicio (`start`). Donde el código renombra las columnas por
posición (`df.columns = ["open", "high", ...]`), antes se reordenan como estaban:
`[["Open", "High", "Low", "Close", "Adj Close", "Volume"]]`.

**Si escribes el código siguiendo el vídeo**, añade esos argumentos en tu `yf.download`.

`yf.Ticker(...).history()` no ha cambiado (ya ajustaba precios y bajaba un mes por defecto). Y los dobles
corchetes de `df[["Close"]].rolling(15).mean()` **no son un fallo**: funcionan igual en pandas 3.

### 2. matplotlib: el estilo `seaborn` cambió de nombre

Notebooks: `BONUS_ES_ML_Capítulo_07_SVR_para_predecir_stocks`.

`plt.style.use('seaborn')` → `plt.style.use('seaborn-v0_8')`. Es el mismo estilo de gráficos.

### 3. Rutas de Colab

Notebooks: `ES_FX_Capítulo_02_Python_para_Data_Science`, `ES_FX_Capítulo_05_Estrategias_avanzadas_para_Forex`.

`pd.read_csv("/content/fichero.csv")` → `pd.read_csv("fichero.csv")`. En Colab es lo mismo (la carpeta de trabajo es `/content`) y además funciona en tu ordenador.

### 4. Errores que el vídeo provoca a propósito

Notebooks: `ES_FX_Capítulo_01_Los_fundamentos_de_Python`, `ES_FX_Capítulo_07_MT5_Live_Trading`.

Algunas celdas dan un error a propósito para explicar algo (por ejemplo, qué es una variable local). Siguen dándolo; solo se han marcado (`raises-exception`) para que *Ejecutar todo* no se pare ahí.

### Otros cambios

- **Bonus, capítulo 7, celda de estandarización:** `X_test = sc.transform(X_test)` → `X_test_sc = sc.transform(X_test)`.
  Las celdas siguientes usan `X_test_sc`, que nunca se creaba (`NameError`). Es una errata del notebook,
  no un cambio de librería.

## Correcciones de errores del curso (noviembre de 2026)

Aquí no ha cambiado ninguna librería: son fallos del código del curso que enseñaban algo mal. **Lo que
verás en pantalla será distinto del vídeo**, y está explicado en cada punto. Comprobado: los capítulos 4 y 5 y el BONUS de SVR se han
vuelto a ejecutar enteros con `requirements.txt` y la réplica de Yahoo de arriba (todos sin error). Las
cifras de antes y después salen de Yahoo Finance real, descargado el 19 de noviembre de 2026, y de los CSV del repositorio. El capítulo 7 (MetaTrader 5) **no se ha ejecutado** (solo Windows): se ha
comprobado que todas sus celdas compilan y, con un MetaTrader 5 simulado, lo que dicen los puntos 2 y 3.
Los notebooks conservan la salida guardada del vídeo: en las celdas de estos puntos, lo guardado es el
resultado antiguo (por ejemplo, `Alpha: 0.9 %`); al ejecutarlas, sale el nuevo.

### 1. Alfa (capítulos 4, 5 y BONUS)

**En el vídeo:** alfa se calcula como `252 * mean * (1-beta)` (celda 13) y, dentro de `BackTest`, como
`serie.mean() - beta*serie.mean()`. Sale un 0,9 % para Google. Los `BackTest` del capítulo 5 y del BONUS de SVR hacen el mismo cálculo.

**Qué estaba mal:** las dos expresiones son el retorno medio de la cartera multiplicado por (1 − beta).
Ahí no aparece el mercado, así que no dicen si la cartera lo ha hecho mejor que él, y el signo puede salir
al revés que el real. La fórmula escrita en la celda 12 tenía el mismo fallo.

**Ahora:** es el alfa de Jensen, con tipo libre de riesgo cero: el retorno medio de la cartera menos beta
por el retorno medio del SP500, anualizado igual que antes.

```python
alpha = (252 * (mean - beta*val["SP500"].mean()))*100                        # celda 13
alpha = annualiazed_scalar * (serie.mean() - beta*val["SP500"].mean())       # BackTest
```

La fórmula de la celda 12 pasa a ser $r_{PF} - r_f - \beta (r_M - r_f)$. `BackTest` se llama igual y
recibe lo mismo.

**Qué cambia en los números** (alfa anual):

| | Antes | Ahora |
|---|---|---|
| Google 2004–2020 (beta 0,965), celda 13 | 0,9 % | **17,2 %** |
| `BackTest(return_serie)`, celda 21 | 0,918 % | 17,231 % |
| `BackTest(..., 52)` sobre los CSV de forex (beta −0,009), cap. 4, celda 25 | 0,126 % | 0,154 % |
| SVR sobre Google (beta ≈ 0,95), BONUS, celda 27 | ≈ 0,9 a 1,3 % | 11,4 % |
| SVR sobre BTC-USD (beta 0,39), BONUS, celda 31 | −19,7 a −20,3 % | −37,7 % |

El `BackTest` del capítulo 5 no se llama en el notebook. Las del BONUS usan precios hasta hoy, así que
varían un poco de una ejecución a otra.

Alfa no mide seguridad: es la rentabilidad que se obtiene por encima de la que tocaría por el riesgo de
mercado asumido (beta). Con beta cercana a 0 apenas cambia; con beta cercana a 1, el cálculo antiguo daba
casi 0 fuera cual fuera el alfa real.

### 2. La clase `MT5` del capítulo 7 no se podía ejecutar (celda 0)

**En el vídeo:** se ejecuta la celda con la clase `MT5` (`get_data`, `orders`, `resume`, `run` y
`close_all_night`) y se colocan órdenes con ella.

**Qué estaba mal:** en el notebook, `orders` y `resume` tenían la sangría mezclada (7 y 8 espacios en una,
10 y 8 en la otra). La celda daba `IndentationError` en la línea 42 antes de definir nada, y así está desde
que se subió el notebook (enero de 2023). Debajo había un segundo fallo: en `resume`, el `pd.concat` estaba
fuera del `for`. Con dos posiciones abiertas devolvía una sola fila, y sin ninguna daba `UnboundLocalError`.
Como `run` llama a `resume` cada vez, el robot no habría podido abrir la primera posición (comprobado con un
MetaTrader 5 simulado).

**Ahora:** 18 líneas re-sangradas, y el `pd.concat` pasa a estar dentro del `for`, como explica el vídeo («las
iremos concatenando»). La clase queda idéntica (el mismo árbol sintáctico) a la del curso de trading con
Machine Learning. Con el MetaTrader 5 simulado, `resume` devuelve 2 filas con 2 posiciones y 0 filas sin
ninguna. Ejecutarlo de verdad sigue necesitando Windows con MetaTrader 5.

### 3. El SVR en vivo usaba precios del mismo día (capítulo 7, celda 1)

**En el vídeo** («Estrategia de Trading en Vivo: SVR»): la función `svm_reg_trading` es «el mismo código
que habéis visto» en el capítulo 5.

**Qué estaba mal:** el capítulo 5 elige las variables con `6:dfc.shape[1]-6`, y el capítulo 7 lo hacía con
`6:dfc.shape[1]-1`. Esas cinco columnas de más son el `open`, `high`, `low`, `close` y `volume` del mismo
día sin desplazar, es decir, el precio con el que se calcula el retorno que el modelo tiene que predecir.
Comprobado con un MetaTrader 5 simulado sobre `EURUSD_D1.csv`: 90 variables, y las cinco últimas eran esas.

**Ahora:** `6:dfc.shape[1]-6` en el entrenamiento y en el test, como en el capítulo 5 (85 variables, ningún
precio sin desplazar).

### 4. Faltaba `import time` (capítulo 7, celda 1)

El bucle en vivo termina cada ronda con `time.sleep(1)`, pero ninguna celda importaba `time`. El robot daba
`NameError` justo después de enviar la primera ronda de órdenes. Ahora la celda 1 lo importa.

### 5. Capítulo 7 (MetaTrader 5): avisos (sin ejecutar)

- **Avisos, sin cambiar el código:** hay un comentario en `filling_mode - 1` (en la celda 0 del notebook). `filling_mode` es una máscara de bits del broker: restarle 1 solo acierta si el
  símbolo admite un único modo de ejecución, y si no, la orden puede rechazarse. Opera
  siempre en una **cuenta demo**, y empieza con `live = False`.
- **El SVR en vivo usa `SVR()`, no el del capítulo 5.** El capítulo 5 usa `SVR(kernel="rbf",
  epsilon=0.005)`. Con el `epsilon=0.1` por defecto, todos los retornos diarios del forex caben en el margen
  de ±10 % que el modelo no penaliza, y apenas aprende. Con el MetaTrader 5 simulado sobre los últimos 60 días
  de `EURUSD_D1.csv`, compra el 100 % de los días. Si quieres el modelo del capítulo 5, pon allí el mismo
  `epsilon`. No se ha cambiado porque decide qué hace el robot y no se ha podido probar contra un broker.

## Erratas del vídeo (el código no cambia)

- **Anualizar con 52** («Probando el backtesting de nuestra estrategia en Forex + Apalancamiento», capítulo 4,
  celda 25). El vídeo usa `BackTest(..., 52)` porque cada posición dura cinco días y salen «52 posiciones» al
  año. Pero la serie sigue teniendo una fila por día (EURUSD: unas 260 al año) y `BackTest` anualiza la
  media por fila, así que con 52 sale unas cinco veces menos de lo que toca: en el test de EURUSD, la media
  por fila × 252 da 0,0061, × 52 da 0,0013, y la suma real por año es 0,0063. Las cifras de alfa de esta
  página siguen calculadas con 52, como en el vídeo.
- **Pesos, umbrales y apalancamiento elegidos sobre el test** (capítulo 6). `ML.csv` y `TA.csv` guardan los
  retornos del periodo de test (`train=False`), y sobre esos mismos datos se optimizan los pesos del Sortino,
  se eligen los umbrales del drawdown (1,5 %, 1 %, 6 %) y el apalancamiento (×10 a ×80), y se juzga el
  resultado. El propio vídeo lo advierte: el umbral «lo tenéis que colocar antes de crear la estrategia». Así
  medido, el resultado es optimista: para juzgarlo, elige todo eso con el entrenamiento y evalúa en el test.
- **«Un 6 % en cada uno de 10 activos es una pérdida máxima del 60 %»** (capítulo 6). Con pesos que suman el
  100 %, perder un 6 % en cada activo es perder en torno a un 6 % de la cartera, no un 60 %.
