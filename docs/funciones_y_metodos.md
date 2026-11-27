# Funciones y métodos del tutorial

Complemento del tutorial «Suficiencia energética del SIN 2026-2030 y expansión de mínimo costo» (`Tutorial_Suficiencia_SIN.ipynb`). Reúne las funciones y los métodos que usa cada sección, en el orden en que aparecen: qué hacen, con qué parámetros los llamamos y por qué. Las funciones propias tienen, además, su docstring en el código del notebook.

## 2.1 Instalación y configuración

| Función | Qué hace | Parámetros que usamos | Por qué así |
|---|---|---|---|
| `importlib.metadata.version(paquete)` | Devuelve la versión instalada de un paquete; lanza `PackageNotFoundError` si no está | El nombre del paquete en PyPI | Viene con Python: no hace falta importar cada paquete para saber su versión |
| `revisar_paquetes(probadas)` (propia) | Compara cada versión instalada con la que probamos | Un diccionario `{paquete: versión}` | Atrapa `PackageNotFoundError` para reportar el paquete que falta en lugar de detener el notebook |

## 2.2 Las fuentes y el paquete de datos

| Función | Qué hace | Parámetros que usamos | Por qué así |
|---|---|---|---|
| `requests.get(url, timeout)` | Hace una solicitud HTTP GET y devuelve la respuesta | `timeout=60` segundos | Sin `timeout`, una conexión colgada detiene el notebook para siempre |
| `respuesta.raise_for_status()` | Lanza un error si la respuesta no fue exitosa (404, 500...) | Ninguno | Así un error del servidor no se guarda como si fuera el archivo |
| `hashlib.sha256(bytes).hexdigest()` | Calcula la huella SHA-256 de un contenido | Los bytes del archivo | Cambia por completo si cambia un solo byte |
| `huella(ruta)`, `descargar(nombre)` y `obtener_datos()` (propias) | Calculan la huella, bajan un archivo con reintentos y dejan verificado todo el paquete | El nombre del archivo; 3 intentos | Una red intermitente no debería tumbar el notebook al primer intento |

## 2.3 El índice de El Niño

| Función | Qué hace | Parámetros que usamos | Por qué así |
|---|---|---|---|
| `pd.read_csv(texto, sep=r"\s+")` | Lee una tabla de texto | `sep=r"\s+"`: uno o más espacios | El archivo de NOAA separa las columnas con espacios, no con comas |
| `Series.map(diccionario)` | Reemplaza cada valor por el del diccionario | `MES_CENTRAL`: trimestre → mes central | Deja el trimestre como un mes, comparable con las demás series |
| `Series.shift()` y `cumsum()` | Desplazan la serie un lugar y acumulan | — | Juntos numeran cada racha de meses seguidos que cumplen la condición |
| `groupby(...).transform("size")` | Le da a cada fila el tamaño de su grupo | Las rachas | Dice cuántos meses dura la racha de cada mes |
| `leer_oni(texto)` y `rachas(serie, umbral)` (propias) | Leen el archivo de NOAA y marcan las rachas de al menos cinco meses | `umbral=±0,5`, `minimo=5` | Es la definición de NOAA |

## 2.4 Carga, limpieza y descripción

| Función | Qué hace | Parámetros que usamos | Por qué así |
|---|---|---|---|
| `pd.read_csv(ruta, parse_dates, dtype)` | Lee un CSV | `parse_dates` con las columnas de fecha; `dtype=str` en los códigos | Fechas como fechas; un código como "0001" no debe perder sus ceros |
| `DataFrame.info()` | Muestra columnas, tipos y valores no faltantes | — | Es la primera mirada a una tabla: tamaño, tipos y huecos |
| `DataFrame.isna().sum()` | Cuenta los faltantes de cada columna | — | Dice qué columnas tienen huecos y cuántos |
| `DataFrame.describe()` | Estadísticas descriptivas: media, desviación, mínimo, cuartiles y máximo | Las columnas numéricas que interesan | Resume cada variable en una sola tabla |
| `leer(nombre, fechas)` (propia) | Lee un CSV del paquete con sus fechas | El nombre del archivo | Evita repetir la ruta y la conversión de fechas |
| `DataFrame.select_dtypes("number")` | Toma las columnas numéricas | — | Los valores extremos solo tienen sentido en números |
| `DataFrame.nunique()` | Cuenta los valores distintos de cada columna | Más de 10 | Deja por fuera los indicadores 0/1 y los parámetros con pocos valores, donde «extremo» no significa nada |
| `DataFrame.quantile(q)` | Cuantiles de cada columna | 0,25 y 0,75 | Dan el rango intercuartílico (RIC) |
| `DataFrame.duplicated(subset)` | Marca las filas cuya llave ya apareció antes | La llave de cada tabla | Una llave repetida duplicaría filas en las uniones |
| `revisar_tabla(nombre, llave, fechas)` (propia) | Revisa una tabla y resume la revisión en una fila | La llave y las fechas de cada tabla | La misma revisión para todas las tablas |

## 2.5 Unir las fuentes

| Función | Qué hace | Parámetros que usamos | Por qué así |
|---|---|---|---|
| `DataFrame.merge(otra, on, how, validate)` | Une dos tablas por una llave | `on="mes"`, `how="inner"` y `validate="one_to_one"` | Un mes del sistema tiene un solo IPC y una sola fase; si no, el error salta aquí |
| `pd.cut(serie, bins, labels)` | Clasifica valores en intervalos | Cortes en 80% y 110% | Separan los meses secos, normales y húmedos según sus aportes frente a la media histórica |
| `groupby(...).agg(...)` | Resume por grupo | Media y número de meses por condición y por fase | Una fila por categoría |
| `precio_real(sistema, ipc, base)` (propia) | Deflacta el precio y clasifica la hidrología | Base: septiembre de 2026 | El último mes con IPC publicado a la fecha de corte |
| `DataFrame.corr(method)` | Correlación entre columnas | `method="spearman"`, entre el precio real, el margen y el embalse | La correlación de rangos no depende de los picos del precio |
| `make_subplots(rows, cols, specs, subplot_titles)` | Arma una figura con varios paneles | Dos filas: la evolución arriba, a lo ancho (`colspan`), y dos nubes de puntos abajo | La evolución y la relación con los dos indicadores se leen juntas |
| `Series.dt.year` y `dt.month` | Extraen el año y el mes de una fecha | — | El período depende de si el mes es diciembre |
| `Series.dt.days_in_month` | Días de cada mes | — | La energía firme se mide en GWh por día; hay que saber cuántos días tiene cada período |
| `periodo_cxc(meses)` (propia) | Período del cargo de cada mes | Fechas | Un diciembre pertenece al período que empieza ese año |

## 3.1 Homologación de plantas y proyectos con RapidFuzz

| Función | Qué hace | Parámetros que usamos | Por qué así |
|---|---|---|---|
| `unicodedata.normalize("NFKD", texto)` | Separa cada letra de su tilde | `"NFKD"` y después `.encode("ascii", "ignore")` | Así "Á" queda en "A" y "Ñ" en "N" |
| `re.sub(patron, reemplazo, texto)` | Reemplaza lo que cumple una expresión regular | Capacidades (`\d+ MW`), signos y la cola "Y SU CONEXIÓN..." | Lo que no identifica a la planta no debe sumar puntaje |
| `re.findall(r"\b\d+\b", texto)` | Encuentra todos los números sueltos | El nombre normalizado | Los números distinguen etapas: "LLANOS 8" no es "LLANOS 6" |
| `normalizar(nombre)` y `numeros(nombre)` (propias) | Dejan solo lo que identifica a la planta, y sus números | El nombre tal como viene | Son el primer paso de la homologación |
| `process.cdist(consultas, opciones, scorer, workers)` | Calcula el puntaje de todos los pares entre dos listas de nombres | `scorer=fuzz.token_set_ratio` (y otra vez con `fuzz.WRatio`) y `workers=-1` | `workers=-1` usa todos los núcleos; la matriz completa evita un ciclo en Python sobre cada par |
| `np.argsort(-fila, kind="stable")` | Ordena de mayor a menor y devuelve las posiciones | `kind="stable"` | Ante un empate, conserva el orden original: el resultado no cambia entre corridas |
| `candidatos(izq, der, top_k)` (propia) | Los `top_k` mejores candidatos de cada registro, dentro del mismo bloque de tecnología | `top_k=3` | Basta para que la pareja correcta casi siempre esté entre los candidatos |
| `np.where(condicion, si, no)` | Elige elemento a elemento entre dos valores | Faltante (`np.nan`) cuando un lado no tiene el dato | "No aplica" no es lo mismo que "falla": no debe descartar el par |
| `np.maximum(a, b)` | Máximo elemento a elemento | 10% de la capacidad y 2 MW | Una tolerancia relativa sola sería demasiado estrecha en las plantas pequeñas |
| `atributos(c)` (propia) | Añade las cinco pruebas y la columna `conflicto` | La tabla de candidatos | Un conflicto descarta el par por alto que sea su puntaje |
| `Series.where(condicion)` | Deja el valor donde se cumple la condición y faltante donde no | `~conflicto` | La regla "con atributos" no deja competir a un par con conflicto |
| `groupby(...).transform("max")` | Le da a cada fila el máximo de su grupo | El registro de la izquierda | Para saber si un par es el mejor candidato de su registro |
| `merge(..., how="right", validate="one_to_one")` | Une las predicciones con las etiquetas, conservando todas las etiquetas | La llave tarea, id_izq, id_der | Una pareja verdadera que no quedó entre los candidatos cuenta como no encontrada |
| `curva(c, etiquetas)` y `elegir_umbral(cur, precision_min)` (propias) | Miden precisión, exhaustividad y F1 por umbral, y escogen el umbral | Umbrales de 50 a 100, de 2 en 2; `precision_min=0.9` | Entre los umbrales con al menos 90% de precisión, el de mayor F1 |
| `groupby("es_misma").sample(frac, random_state)` y `DataFrame.drop(índice)` | Sacan al azar una fracción de cada grupo, y devuelven la tabla sin esas filas | `frac=0.5` y `random_state=SEMILLA` | Las dos mitades de las etiquetas quedan con la misma proporción de parejas verdaderas, y la partición es la misma en cada corrida |
| `Series.str.contains(patron)` | Marca los textos que contienen un patrón | `"TERCERA SUBASTA"` y `"Nuevas"` | La subasta de 2019, plantas nuevas |
| `np.select(condiciones, valores, default)` | Elige un valor según la primera condición que se cumpla | Entró tarde (más de un mes), entró a tiempo, no ha entrado | Tres grupos excluyentes |
| `np.ceil(x)` | Redondea hacia arriba | El retraso medio en meses / 12 | Un retraso de 21 meses ocupa dos períodos del cargo |

## 3.2 Costos y bloques candidatos

| Función | Qué hace | Parámetros que usamos | Por qué así |
|---|---|---|---|
| `pd.to_numeric(serie, errors="coerce")` | Convierte texto en números; lo que no es número queda como faltante | La columna `valor` de `fuentes_supuestos.csv` | El primer período es texto ("2028-2029"): no debe romper la conversión |
| `supuesto(tecnologia, parametro, escenario)` (propia) | Lee un valor de la tabla de fuentes | La tecnología y el parámetro | Los cupos, el tope solar y los factores de las menores salen de la tabla, no se escriben a mano |
| `crf(r, n)` (propia) | Factor de recuperación de capital | $r$ = 10% y la vida útil de cada tecnología | Anualiza la inversión con la tasa del PIEG |
| `bloques(escenario_costos, r, cupos, retraso_periodos, objetivo)` (propia) | Arma un bloque por tecnología con su costo anual, su energía firme y su primer período factible | Escenario central y costo total (fijo + variable) | Los demás valores los usan las sensibilidades (3.5) y el costo por MWh firme (4.4) |
| `Series.map(diccionario o función)` | Traduce cada valor | El grupo de cupo → MW; el período → su posición | Une cada bloque con su cupo y su primer período |
| `DataFrame.pivot(index, columns, values)` | Pasa una tabla larga a una ancha | Parámetro × tecnología, con la fuente en cada celda | Una fila por parámetro se lee de un vistazo |
| `json.loads(texto)` | Lee un JSON | El Catálogo, tal como lo devuelve la API de la UPME | Es la fuente original, sin procesar |
| `del_catalogo(tecnologia, variable)` (propia) | Valor del Catálogo en 2027 | Tecnología y variable del Catálogo | El Catálogo da 2024 y 2030, y los costos son de 2027: se interpola |

## 3.3 Balance de energía firme

| Función | Qué hace | Parámetros que usamos | Por qué así |
|---|---|---|---|
| `Series.isin(lista)` | Marca los valores que están en la lista | Situación "en operación" o "en pruebas" | Solo esas plantas generan |
| `Series.str.startswith(texto)` | Marca los textos que empiezan así | `"GD "` | Así se nombran las plantas de generación distribuida |
| `parque_firme(plantas)` (propia) | Energía firme de cada planta existente con las reglas de arriba | Las plantas del mes de referencia | Cada planta queda con su motivo si no suma |
| `fraccion_entrada(periodo_entrada, vencido, retrasos)` (propia) | Fracción esperada de la OEF de un proyecto que opera en cada período | El período en que empieza su OEF, si ya venció y si el escenario lleva retrasos | Con retrasos: 53% desde su período y 19% más dos períodos después |
| `np.arange(n)` | Los enteros de 0 a n − 1 | Las posiciones de los cuatro períodos | Para comparar cada período con el de entrada |
| `DataFrame.to_numpy()` | Pasa la tabla a una matriz de NumPy | La OEF de los proyectos por período | El cálculo es una suma de productos, más rápida sobre matrices |
| `oferta_firme(retrasos)` (propia) | Energía firme de cada período (GWh), por componente | Con o sin retrasos | Las dos ofertas de los tres escenarios |
| `balance(escenario)` (propia) | Demanda, oferta y brecha de cada período de un escenario | `"base"`, `"estres"` u `"optimista"` | Cada escenario combina una demanda con una oferta |
| `pd.concat(tablas, ignore_index=True)` | Apila tablas una debajo de otra | Los tres escenarios | Una sola tabla para comparar y graficar |
| `pd.cut(serie, bins, labels)` | Clasifica valores en intervalos | Cortes en 80% y 110% de los aportes medios | Los mismos umbrales de la sección 2.5 |
| `groupby(...).agg([...])` | Resume por grupo con varias funciones | Número, mínimo, mediana y máximo por condición | Muestra cuánto varía la relación en cada condición, no solo su centro |
| `Series.median()` | La mediana | Los períodos secos y los normales | Con tres y siete períodos, un período atípico mueve menos la mediana que la media |
| `Series.drop(etiqueta)` | Devuelve la serie sin ese elemento | Un período normal a la vez | Muestra cuánto depende la mediana de un solo período |
| `balance_hidrologia(escenario, factor)` (propia) | Brecha con otra hidrología | El escenario y el factor del año seco o del normal | Suma a la oferta la energía extra de la hidráulica y deja igual lo demás |

## 3.4 Expansión de mínimo costo (MILP)

| Función | Qué hace | Parámetros que usamos | Por qué así |
|---|---|---|---|
| `np.outer(a, b)` | Producto exterior: una matriz con todos los productos $a_i b_j$ | MW × factor firme de cada bloque y horas de cada período | Da de una vez la energía firme $F_{b,t}$ de cada bloque en cada período |
| `np.triu(np.ones((n, n))) @ delta` | Suma, para cada período, los descuentos de ese período en adelante | La matriz triangular superior | Un bloque que entra en τ paga su costo en todos los períodos desde τ |
| `np.r_[a, b]` | Concatena vectores | Costos de las $y$ y de las $u$ | El vector de costos sigue el orden de las variables |
| `groupby("grupo_cupo", sort=False)` | Agrupa conservando el orden de aparición | Los grupos de cupo | El ciclo abierto y el combinado comparten el cupo de gas |
| `restricciones(...)` y `armar_milp(...)` (propias) | Arman el problema | Los bloques, la brecha, los días, la tasa, la penalidad y el tope | Separan la matriz del resto para que cada pieza se lea sola |
| `scipy.optimize.milp(c, constraints, integrality, bounds, options)` | Minimiza c·v sujeto a lb ≤ A·v ≤ ub, con enteras las variables marcadas en `integrality` | `LinearConstraint(A, lb, ub)`; `integrality` = 1 en las $y$ y 0 en las $u$; `options={"mip_rel_gap": 1e-9}` | Con 28 variables, HiGHS encuentra el óptimo exacto en menos de un segundo: no hace falta aceptar una brecha de optimalidad |
| `LinearConstraint(A, lb, ub)` y `Bounds(lb, ub)` | Empaquetan las restricciones y las cotas | La matriz y los vectores de `armar_milp()` | Es el formato que pide `milp` |
| `resolver(pr, relajado)` (propia) | Resuelve el MILP o su relajación lineal | `relajado=True` quita la integralidad | La relajación sirve para validar: su costo nunca puede superar al del MILP |
| `correr(escenario, caso, **cambios)` (propia) | Resuelve un escenario con los cambios de un caso y resume la solución | `caso="Central"`; `brecha_extra` suma GWh a la brecha (sección 5 y ejercicio 2 del notebook de ejercicios) | Una sola función para los escenarios, las sensibilidades y los ejercicios |
| `dict \| dict` | Une dos diccionarios; el de la derecha manda | Los cambios del caso y los que se pasen | Así un caso puede modificarse sin reescribirlo |

## 3.5 Sensibilidades

| Función | Qué hace | Parámetros que usamos | Por qué así |
|---|---|---|---|
| `dict.update(otro)` | Agrega las entradas de otro diccionario | Los 7 casos de sensibilidad | `correr()` los lee de `CASOS` |
| `DataFrame.xs(clave, level)` | Toma una sección de un índice de varios niveles | Un escenario | Para ver los 8 casos de un escenario |

## 5.4 Mejora 1: la brecha como distribución

| Función | Qué hace | Parámetros que usamos | Por qué así |
|---|---|---|---|
| `rng.dirichlet(alfa, size)` | Sortea vectores de probabilidades que suman 1 | `alfa` = 1 + 9 × fracciones de 2019 | Representa que las probabilidades salen de una muestra pequeña |
| `rng.random(forma)` | Números uniformes entre 0 y 1 | Uno por simulación y proyecto | Se comparan con las probabilidades para decidir el desenlace |
| `rng.choice(valores, size)` | Escoge al azar entre unos valores | Los tres retrasos observados | El retraso de quien entra tarde sale de lo observado |
| `rng.standard_normal(forma)` | Sortea una normal estándar | Una por simulación | El *z* de la demanda |
| `np.clip(a, 0, 1)` | Recorta los valores al intervalo [0, 1] | La fracción de cada período que opera un proyecto | No puede operar menos de cero meses ni más de doce |
| `operando()` y `simular_brecha()` (propias) | Fracción operando de cada proyecto y brecha simulada | 20.000 simulaciones con la semilla `SEMILLA` | Con la semilla fija, el notebook da siempre lo mismo |

## 5.5 El plan del caso base frente a la distribución

| Función | Qué hace | Parámetros que usamos | Por qué así |
|---|---|---|---|
| `np.maximum(a, 0)` | Deja en cero los valores negativos | La brecha simulada menos la energía firme del plan | La energía sin cubrir es max(0, brecha − plan): un período con holgura no compensa otro con faltante |
| `evaluar_plan(det, sim)` (propia) | Energía firme de un plan, probabilidad de cubrir y energía esperada sin cubrir, por período | El detalle de `correr()` y las simulaciones | Mide cualquier plan contra la misma distribución |
| `correr("base", brecha_extra=...)` | El MILP de la sección 3.4 | La diferencia entre el percentil y el valor esperado | Resuelve el mismo modelo con otra brecha, sin tocarlo |

## 5.6 Mejora 2: los bloques nuevos con el mismo riesgo

| Función | Qué hace | Parámetros que usamos | Por qué así |
|---|---|---|---|
| `con_riesgo(pr)` (propia) | Copia el MILP y multiplica la energía firme de cada bloque por la fracción que se espera que opere | `MODELO_RETRASOS` | Es la regla de la sección 3.3, aplicada ahora a los bloques |
| `dict \| dict` | Une dos diccionarios; el de la derecha manda | El problema y su nueva matriz | Deja intacto el problema original |
