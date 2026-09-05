
# ANÁLISIS DE LA VARIACIÓN EN LA CALIDAD DEL AIRE EN LAS REGIONES DEFINIDAS POR EL CENSO EN ESTADOS UNIDOS

## Grupo2_Salud

### Integrantes:

- JUSTAILS: Justin Guevara  
- DRKca089: Derek Cahuate  

### Descripción del proyecto

El proyecto consiste en el análisis de la variación histórica del Índice de Calidad del Aire (AQI) en Estados Unidos durante el periodo 1980–2021, utilizando datos recopilados por la Agencia de Protección Ambiental de los Estados Unidos (EPA). El análisis se realiza agrupando los registros según las cuatro regiones definidas por el Censo de Estados Unidos: Northeast, Midwest, South y West.

Los datos utilizados se limpiaron, normalizarón y agruparon en regiones. Luego se aplicaron los métodos númericos para modelar el comportamiento del AQI. Se utiliza la interpolación del spline cúbico para representar la tendencia histórica del Median AQI, y se usa mínimos cuadrados para realizar extrapolaciones.

Finalmente, los resultados obtenidos son representados graficamente el objetivo de analizar las tendencias del AQI, comparar diferencias entre regiones e identificar seguira su tendencia hacia el futuro.

### Variables utilizadas

Para el análisis se utilizaron las siguientes variables, las cuales se encuentran en el dataset normalizado generado durante la etapa de limpieza:

- **Year:** Año en el que fueron registrados los datos.
- **Region:** Región geográfica asignada según la clasificación del Censo de Estados Unidos.
- **Median AQI:** Valor mediano anual del Índice de Calidad del Aire utilizado como variable principal para analizar la tendencia histórica.
- **AQI level days:** Proporción o cantidad normalizada de días clasificados según las categorías del AQI (Good, Moderate, Unhealthy for Sensitive Groups, Unhealthy, Very Unhealthy y Hazardous).
- **Pollutant days:** Proporción o cantidad normalizada de días donde cada contaminante fue el principal responsable del AQI registrado (CO, NO2, Ozone, SO2, PM2.5 y PM10).

### Descripción del Github

El repositorio contiene las siguientes carpetas y archivos necesarios para la ejecución y documentación del proyecto. Su estructura se divide en:

- **Avances:** Contiene el informe del proyecto que se actualiza conforme se va avanzando el proyecto.
- **Codigo:** Incluye los códigos desarrollados en Python para la limpieza del dataset, aplicación de spline cúbico y aplicación de mínimos cuadrados.
- **Dataset:** Contiene el dataset original obtenido desde Kaggle y el dataset limpiado utilizado para el análisis final.
- **Graficas:** Almacena las graficas generadas durante la aplicación de splines cúbicos y mínimos cuadrados.
- **Readme:** Contiene la información general del proyecto. 

### Fases del desarrollo del proyecto

**Fase 1: Preparación y análisis de datos**

Se realizó la limpieza y transformación del dataset original, seleccionando los registros del periodo 1980–2020 debido a la menor disponibilidad de datos en 2021. Se asignó cada registro a una de las cuatro regiones definidas por el Censo de Estados Unidos y se eliminaron valores nulos y registros no válidos.

Además, se normalizaron las variables relacionadas con categorías del AQI y contaminantes mediante proporciones, y los datos fueron agrupados por región y año.

**Fase 2: Aplicación de Splines Cúbicos**

Se implementó el método de interpolación mediante splines cúbicos naturales. Los coeficientes obtenidos permitieron construir funciones polinómicas por tramos para representar la evolución histórica del AQI en cada región, utilizando condiciones de frontera natural.

**Fase 3: Aplicación de Mínimos Cuadrados**

Se aplicó el método de mínimos cuadrados polinómicos para modelar la tendencia del AQI en cada región, utilizando Year como variable independiente y Median AQI como variable dependiente.

Mediante la matriz de Vandermonde se obtuvieron los coeficientes del polinomio de grado 2, utilizado posteriormente para realizar extrapolaciones basadas en la tendencia histórica del AQI.

**Fase 4: Validación del modelo**

En esta fase se compararon los resultados obtenidos mediante los métodos de splines cúbicos y mínimos cuadrados, analizando la tendencia histórica dada por el spline cúbico y su relación con las variables de la distribución de días según las categorías del AQI y los contaminantes principales.

Para el método de mínimos cuadrados se utilizó un ajuste polinómico de grado 2, seleccionado debido a que permite representar la tendencia general del AQI manteniendo una adecuada estabilidad del modelo y evitando oscilaciones excesivas durante la extrapolación.

Para evaluar el comportamiento del modelo, se utilizaron los datos comprendidos entre 1980 y 2015 para realizar el ajuste, mientras que los registros correspondientes al periodo de 2016 a 2020 fueron utilizados como datos de validación. De esta manera, se verificó si la tendencia estimada por los modelos mantiene un comportamiento similar al observado históricamente.

**Fase 5: Interpretación de resultados**

Finalmente, se realizará un análisis comparativo entre las cuatro regiones utilizando las gráficas generadas durante las etapas anteriores.

A partir de estos resultados se buscará identificar:

- Tendencias crecientes o decrecientes del AQI.
- Contaminantes responsables del nivel del AQI.
- Diferencias entre regiones.
- Tendencia futura del comportamiento del AQI.

### Ejecución del proyecto

Para ejecutar el proyecto se deben seguir los siguientes pasos:

1. **Clonar el repositorio**

   Descargar el repositorio de GitHub y acceder a la carpeta principal del proyecto.

2. **Ejecutar el notebook principal**

   Abrir y ejecutar las celdas del archivo `Proyecto.ipynb`, donde se encuentran los procesos de:

   - Limpieza y normalización de datos.
   - Aplicación de splines cúbicos y mínimos cuadrados.
   - Generación de gráficas.

3. **Verificación de resultados**

   Al finalizar la ejecución se debe comprobar la generación de los archivos resultantes:

   - El dataset procesado se almacena en la carpeta `dataset`.
   - Las gráficas generadas por los métodos numéricos se almacenan en la carpeta `graficas`.
  
"""
Crea "nuevos grupos" a partir de condados.gpkg, combinando CSA/CBSA/condados
sueltos. Todo el proceso corre POR SEPARADO dentro de cada uno de los grupos
de estados definidos en GRUPOS_ESTADOS -- un CSA/CBSA que cruce dos de esos
grupos siempre termina separado en dos grupos distintos.

REGLAS:

CASO 1 - CBSA que pertenecen a un CSA real (no "XXX"):
    Dentro de cada CSA, se evalua CADA CBSA por separado (el CSA es solo el
    "corral" dentro del cual se puede fusionar, no un grupo en si mismo):
      - Si ese CBSA ya tiene mas de 1 condado (y >=1000 hab), se queda tal
        cual, como su propio grupo final.
      - Si tiene un solo condado (o <1000 hab), intenta fusionarse con OTRO
        CBSA del MISMO CSA que este vecino (comparta borde) y que tambien
        haya quedado suelto. Si hay varios candidatos, se une al de MENOR
        poblacion; si hay uno solo, se une sin importar poblacion.
      - Si no encuentra ningun candidato del mismo CSA, se queda SOLO (nunca
        busca fuera de su propio CSA).

CASO 2 - CBSA sueltos (no pertenecen a ningun CSA):
    Misma logica que el Caso 1 pero sin la restriccion de CSA: si tiene mas
    de 1 condado se queda tal cual; si es de 1 solo condado, primero busca
    fusionarse con OTRO CBSA vecino que tampoco pertenezca a ningun CSA
    (menor poblacion si hay varios candidatos, el unico si hay uno solo).
    Si no encuentra ningun otro CBSA, tiene una ultima chance ANTES de darse
    por vencido: se fusiona con UN condado sin-CBSA vecino (el de menor
    poblacion si hay varios). Ese condado, aunque no es un CBSA de verdad,
    de ahi en mas CUENTA como parte de un grupo "con CBSA" -- y el CBSA
    queda cerrado para siempre, nunca vuelve a buscar otro. Recien si no
    tiene NINGUN candidato de ningun tipo (ni otro CBSA, ni un condado
    sin-CBSA vecino) se queda solo.
    El orden importa: esta ultima chance del CBSA corre ANTES de que el
    Caso 3 empiece a procesar los condados sin-CBSA. La prioridad general
    del algoritmo es estrictamente CBSA en CSA -> CBSA sin CSA -> condados
    sin CBSA, en ese orden. El condado que un CBSA se lleva aca simplemente
    deja de existir como opcion para el Caso 3 -- no le esta quitando nada
    a nadie, porque el Caso 3 ni siquiera arranco todavia cuando esto pasa.

CASO 3 - Condados sin CBSA (CSA=XXX y CBSA=XXX):
    Arranca DESPUES de que todos los CBSA (Casos 1 y 2, incluida la ultima
    chance de este ultimo) ya terminaron de resolverse. Cualquier condado
    sin-CBSA que un CBSA ya se haya llevado en el Caso 2 no forma parte de
    este calculo -- ya cuenta como CBSA. A diferencia de los CBSA, un
    condado sin CBSA NUNCA puede quedar suelto por su cuenta -- siempre
    debe terminar en algun grupo. Prioridad:
      1) Otro condado sin-CBSA vecino que TODAVIA este solo (nadie se le
         unio aun) -- el de menor poblacion. Si no hay ninguno solo, se une
         a un grupo sin-CBSA YA FORMADO (2+ condados) vecino, el de menor
         poblacion.
      2) Si NO tiene ningun vecino sin-CBSA (esta totalmente rodeado de
         condados con CBSA), intenta en este orden:
           a) un CBSA vecino que originalmente era de 1 solo condado (pudo
              haberse fusionado con otro originalmente-solo en el Caso 2),
              que NO pertenezca a ningun CSA -- el de menor poblacion
           b) si no hay (a), un CBSA vecino que originalmente tenia 2+
              condados, que NO pertenezca a ningun CSA -- el de menor
              poblacion
           c) si no hay (a) ni (b), un CBSA vecino que SI pertenezca a un
              CSA (ultimo recurso, sin excepcion) -- el de menor poblacion
      Un CBSA solo puede absorber UN condado/grupo sin-CBSA en total (ya
      sea aca o en su ultima chance del Caso 2): en cuanto lo hace, queda
      cerrado para siempre y no puede ser elegido de nuevo. Si un condado
      sin-CBSA de verdad no tiene NINGUN vecino (islas aisladas), se queda
      solo y se marca para revisar a mano.
    Un condado sin-CBSA que ya tiene con quien (2+ condados en su grupo)
    queda resuelto de inmediato y deja de buscar mas pareja -- esto NO
    depende de cuanta poblacion tenga el grupo, solo de que ya no este solo.
    Igual puede seguir recibiendo pasivamente a otros condados sin-CBSA que
    SI sigan buscando (ver nivel 1 arriba), solo que el ya no inicia mas
    busquedas por su cuenta.

LIMPIEZA FINAL:
    Cualquier grupo (de cualquier caso) que termine con poblacion total
    menor a POBLACION_MINIMA intenta fusionarse con su vecino de menor
    poblacion (sin importar el tipo), repitiendo hasta superar el minimo o
    quedarse sin vecinos (se marca Revisar_manual en el Excel).

Salidas:
    - grupos_resultado.xlsx: condado -> CBSA/CSA original -> grupo nuevo
      (con poblacion y si necesita revision manual)
    - un PNG por cada grupo de estados, coloreado por grupo nuevo, con
      nombre y poblacion de cada grupo

Requiere: pip install geopandas matplotlib pandas
"""

import geopandas as gpd
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.patheffects as pe
from collections import defaultdict

GPKG_PATH = r"C:\Users\user\Downloads\condados.gpkg"
OUTPUT_XLSX = r"C:\Users\user\Downloads\grupos_resultado.xlsx"
OUTPUT_DIR = r"C:\Users\user\Downloads"

CSA_COL = "CSA (Combined Statistical Area)"
CBSA_COL = "CBSA (Core Based Statistical Area)"
POBLACION_MINIMA = 1000
XXX = "XXX"

GRUPOS_ESTADOS = [
    ("01_connecticut_massachusetts_rhode_island", ["Connecticut", "Massachusetts", "Rhode Island"]),
    ("02_maine_new_hampshire_vermont", ["Maine", "New Hampshire", "Vermont"]),
    ("03_new_jersey_new_york", ["New Jersey", "New York"]),
    ("04_pennsylvania_delaware", ["Pennsylvania", "Delaware"]),
    ("05_illinois_indiana_wisconsin", ["Illinois", "Indiana", "Wisconsin"]),
    ("06_michigan_ohio", ["Michigan", "Ohio"]),
    ("07_iowa_kansas_missouri_nebraska", ["Iowa", "Kansas", "Missouri", "Nebraska"]),
    ("08_minnesota_north_dakota_south_dakota", ["Minnesota", "North Dakota", "South Dakota"]),
    ("09_dc_maryland_north_carolina_virginia_west_virginia",
     ["District of Columbia", "Maryland", "North Carolina", "Virginia", "West Virginia"]),
    ("10_florida_georgia_south_carolina", ["Florida", "Georgia", "South Carolina"]),
    ("11_alabama_mississippi", ["Alabama", "Mississippi"]),
    ("12_kentucky_tennessee", ["Kentucky", "Tennessee"]),
    ("13_arkansas_louisiana", ["Arkansas", "Louisiana"]),
    ("14_oklahoma_texas", ["Oklahoma", "Texas"]),
    ("15_arizona_nevada_new_mexico_utah", ["Arizona", "Nevada", "New Mexico", "Utah"]),
    ("16_colorado_idaho_montana_wyoming", ["Colorado", "Idaho", "Montana", "Wyoming"]),
    ("17_alaska_washington", ["Alaska", "Washington"]),
    ("18_california_oregon", ["California", "Oregon"]),
    ("19_hawaii", ["Hawaii"]),
    ("20_puerto_rico_islas_virgenes", ["Puerto Rico", "United States Virgin Islands"]),
    ("21_guam_marianas", ["Guam", "Commonwealth of the Northern Mariana Islands"]),
    ("22_samoa_americana", ["American Samoa"]),
]
ESTADO_A_GRUPO = {estado: nombre for nombre, estados in GRUPOS_ESTADOS for estado in estados}


class DSU:
    def __init__(self, ids):
        self.parent = {i: i for i in ids}

    def find(self, x):
        while self.parent[x] != x:
            self.parent[x] = self.parent[self.parent[x]]
            x = self.parent[x]
        return x

    def union(self, a, b):
        ra, rb = self.find(a), self.find(b)
        if ra != rb:
            self.parent[ra] = rb


def construir_adyacencia(gdf, tolerancia_m=30, borde_minimo_m=90):
    """Devuelve {idx: set(idx_vecinos)} para condados que comparten un
    TRAMO REAL de frontera (una linea), no solo un punto.

    Esto importa en cruces tipo "cuatro esquinas" (como donde se tocan
    Arizona, Utah, Colorado y Nuevo Mexico): ahi los 4 condados se tocan en
    un mismo punto, pero solo 2 pares de ellos comparten frontera de verdad
    (los que estan lado a lado); los otros 2 (las diagonales) solo se tocan
    en ese punto y NO deberian contar como vecinos, porque para "cruzar" de
    uno al otro hay que pasar por el territorio de alguno de los otros dos.

    tolerancia_m: cuanto se cierra un micro-hueco de precision entre
    poligonos del TIGER que en teoria se tocan pero no calzan exacto.
    borde_minimo_m: cuanta frontera compartida se exige como minimo para
    contar como vecinos reales (mas que lo que daria un simple toque de
    esquina, que es del orden de la propia tolerancia_m).
    """
    gdf_m = gdf.to_crs("EPSG:5070")
    idxs = list(gdf_m.index)
    geoms = {i: g for i, g in zip(idxs, gdf_m.geometry)}
    bordes = {i: g.boundary for i, g in geoms.items()}
    buffers = {i: g.buffer(tolerancia_m) for i, g in geoms.items()}

    gdf_buf = gpd.GeoSeries(buffers, crs=gdf_m.crs)
    sindex = gdf_buf.sindex

    vecinos = defaultdict(set)
    for idx in idxs:
        candidatos = list(sindex.intersection(buffers[idx].bounds))
        for c in candidatos:
            otro_idx = gdf_buf.index[c]
            if otro_idx == idx or otro_idx in vecinos[idx]:
                continue
            # tramo del borde de "idx" que cae dentro del buffer de "otro_idx"
            largo_compartido = bordes[idx].intersection(buffers[otro_idx]).length
            if largo_compartido >= borde_minimo_m:
                vecinos[idx].add(otro_idx)
                vecinos[otro_idx].add(idx)
    return vecinos


def procesar_grupo_estado(gdf_grupo, nombre_grupo):
    idxs = list(gdf_grupo.index)
    dsu = DSU(idxs)
    pop = gdf_grupo["POP20"].to_dict()
    csa = gdf_grupo[CSA_COL].to_dict()
    cbsa = gdf_grupo[CBSA_COL].to_dict()
    vecinos = construir_adyacencia(gdf_grupo)

    conteo_cbsa_en_grupo = defaultdict(list)
    for i in idxs:
        if cbsa[i] not in (XXX, None) and pd.notna(cbsa[i]):
            conteo_cbsa_en_grupo[cbsa[i]].append(i)
    for nombre_cbsa, miembros in conteo_cbsa_en_grupo.items():
        if len(miembros) == 1 and pop[miembros[0]] < POBLACION_MINIMA:
            i = miembros[0]
            cbsa[i] = XXX
            csa[i] = XXX

    por_cbsa = defaultdict(list)
    for i in idxs:
        if cbsa[i] not in (XXX, None) and pd.notna(cbsa[i]):
            por_cbsa[cbsa[i]].append(i)

    def raiz_pop(root):
        return sum(pop[i] for i in idxs if dsu.find(i) == root)

    def raiz_vecinos(root):
        v = set()
        for i in idxs:
            if dsu.find(i) == root:
                v |= vecinos.get(i, set())
        return {x for x in v if dsu.find(x) != root}

    por_cbsa = defaultdict(list)
    for i in idxs:
        if cbsa[i] not in (XXX, None) and pd.notna(cbsa[i]):
            por_cbsa[cbsa[i]].append(i)

    n_original = {nombre: len(miembros) for nombre, miembros in por_cbsa.items()}
    csa_de_cbsa = {}
    for nombre, miembros in por_cbsa.items():
        csas_no_xxx = [csa[m] for m in miembros if csa[m] not in (XXX, None) and pd.notna(csa[m])]
        csa_de_cbsa[nombre] = csas_no_xxx[0] if csas_no_xxx else XXX

    for nombre_cbsa, miembros in por_cbsa.items():
        for m in miembros[1:]:
            dsu.union(miembros[0], m)

    fragmentos_cbsa_raiz = set()
    for miembros in por_cbsa.values():
        fragmentos_cbsa_raiz.add(dsu.find(miembros[0]))

    def es_fragmento_cbsa_disponible(root):
        return root in fragmentos_cbsa_raiz

    raiz_a_nombres = {dsu.find(miembros[0]): {nombre} for nombre, miembros in por_cbsa.items()}

    def fragmento_sigue_solo(nombre_cbsa, root):
        return raiz_a_nombres.get(root, set()) == {nombre_cbsa}

    def candidato_mismo_csa_suelto(root, csa_objetivo):
        nombres = raiz_a_nombres.get(root, set())
        if not nombres:
            return False
        return all(n_original[n] == 1 and csa_de_cbsa[n] == csa_objetivo for n in nombres)

    def candidato_sin_csa(root):
        nombres = raiz_a_nombres.get(root, set())
        if not nombres:
            return False
        return all(csa_de_cbsa[n] == XXX for n in nombres)

    orden = sorted(por_cbsa.keys(), key=lambda nombre: sum(pop[m] for m in por_cbsa[nombre]), reverse=True)
    resueltos = set()
    for nombre_cbsa, miembros in por_cbsa.items():
        root = dsu.find(miembros[0])
        n_condados = sum(1 for i in idxs if dsu.find(i) == root)
        if n_condados > 1 and raiz_pop(root) >= POBLACION_MINIMA:
            resueltos.add(root)

    # ---- Paso 1: fragmentos CBSA solitarios buscan reunirse con otro fragmento
    #              del MISMO CBSA original que quedó suelto por otro lado. Si el
    #              fragmento pertenece a un CSA, SOLO puede unirse a otro fragmento
    #              del mismo CSA. Si no encuentra nada, se deja solo (no cae a
    #              ningún otro CBSA/CSA distinto en este paso).
    for nombre_cbsa in orden:
        miembros = por_cbsa[nombre_cbsa]
        root = dsu.find(miembros[0])
        if root in resueltos:
            continue
        if not fragmento_sigue_solo(nombre_cbsa, root):
            continue

        vecinos_root = raiz_vecinos(root)
        csa_actual = csa_de_cbsa[nombre_cbsa]

        if csa_actual != XXX:
            candidatos_cbsa = {dsu.find(v) for v in vecinos_root
                                if es_fragmento_cbsa_disponible(dsu.find(v))
                                and dsu.find(v) not in resueltos
                                and candidato_mismo_csa_suelto(dsu.find(v), csa_actual)}
            candidatos_cbsa.discard(root)

            if candidatos_cbsa:
                if len(candidatos_cbsa) == 1:
                    elegido = next(iter(candidatos_cbsa))
                else:
                    elegido = min(candidatos_cbsa, key=raiz_pop)
                nombres_unidos = raiz_a_nombres.pop(root, {nombre_cbsa}) | raiz_a_nombres.pop(elegido, set())
                dsu.union(root, elegido)
                raiz_a_nombres[dsu.find(elegido)] = nombres_unidos
            # Si no hay candidato del mismo CSA: se deja solo. NO se busca fuera del CSA.

        else:
            candidatos_cbsa = {dsu.find(v) for v in vecinos_root
                                if es_fragmento_cbsa_disponible(dsu.find(v))
                                and dsu.find(v) not in resueltos
                                and candidato_sin_csa(dsu.find(v))}
            candidatos_cbsa.discard(root)

            if candidatos_cbsa:
                if len(candidatos_cbsa) == 1:
                    elegido = next(iter(candidatos_cbsa))
                else:
                    elegido = min(candidatos_cbsa, key=raiz_pop)
                nombres_unidos = raiz_a_nombres.pop(root, {nombre_cbsa}) | raiz_a_nombres.pop(elegido, set())
                dsu.union(root, elegido)
                raiz_a_nombres[dsu.find(elegido)] = nombres_unidos

    sin_cbsa = [i for i in idxs if (cbsa[i] in (XXX, None) or pd.isna(cbsa[i]))]

    raiz_tiene_cbsa = {dsu.find(miembros[0]) for miembros in por_cbsa.values()}

    def raiz_es_de_csa(root):
        nombres = raiz_a_nombres.get(root, set())
        return bool(nombres) and any(csa_de_cbsa.get(n, XXX) != XXX for n in nombres)

    def raiz_cbsa_originalmente_solo(root):
        # True si TODOS los cbsa originales fusionados en este grupo eran,
        # cada uno, de un solo condado (pudieron haberse fusionado entre si
        # en el Paso 1, pero ninguno era ya de por si un metro de 2+ condados)
        nombres = raiz_a_nombres.get(root, set())
        return bool(nombres) and all(n_original[n] == 1 for n in nombres)

    def raiz_sin_cbsa_es_solo(root):
        return sum(1 for i in idxs if dsu.find(i) == root) == 1

    def vecinos_sin_cbsa(root):
        return {dsu.find(v) for v in raiz_vecinos(root) if dsu.find(v) not in raiz_tiene_cbsa}

    # ---- CASO 2 (continuacion): un CBSA sin CSA que no encontro pareja
    # entre otros CBSA en el Paso 1 tiene una ultima chance antes de darse
    # por vencido: se fusiona con UN condado sin-CBSA vecino (el de menor
    # poblacion si hay varios), y con eso queda "cerrado" para siempre -- no
    # es un CBSA real, pero de ahi en mas CUENTA como si lo fuera, y nunca
    # vuelve a buscar a otro. Esto corre ANTES del Caso 3 a proposito: el
    # orden de prioridad es CBSA en CSA -> CBSA sin CSA -> condados sin
    # CBSA, en ese orden estricto. El condado que un CBSA se lleva aca
    # simplemente deja de existir como opcion para el Caso 3 -- no le esta
    # quitando nada a nadie, porque el Caso 3 ni siquiera empezo a correr
    # todavia cuando esto pasa.
    for nombre_cbsa in orden:
        if csa_de_cbsa[nombre_cbsa] != XXX:
            continue  # un CBSA de CSA nunca sale a buscar condados sin CBSA
        if n_original[nombre_cbsa] != 1:
            continue  # esta ultima chance es solo para CBSA originalmente de 1 condado
        miembros = por_cbsa[nombre_cbsa]
        root = dsu.find(miembros[0])
        if sum(1 for i in idxs if dsu.find(i) == root) != 1:
            continue  # ya no esta solo (se fusiono con otro CBSA en el Paso 1)

        candidatos_sin_cbsa = {dsu.find(v) for v in raiz_vecinos(root) if dsu.find(v) not in raiz_tiene_cbsa}
        if candidatos_sin_cbsa:
            elegido = min(candidatos_sin_cbsa, key=raiz_pop)
            # OJO con el orden de los argumentos: en DSU.union(a, b) quien
            # sobrevive como representante final es "b", no "a". Si aca
            # hicieramos union(root, elegido), el id que quedaria vivo seria
            # el del lado sin-CBSA (elegido) y no el del CBSA (root) -- y el
            # grupo fusionado dejaria de figurar en raiz_tiene_cbsa, es
            # decir que un CBSA distinto que lo tuviera de vecino mas
            # adelante podria verlo como sin-CBSA libre y fusionarse
            # tambien. Poniendo "root" segundo, el representante final
            # queda siendo el id del CBSA (que SI esta en raiz_tiene_cbsa),
            # asi el condado que se llevo pasa a "contar como CBSA" de ahi
            # en mas, y el propio CBSA queda cerrado (esta iteracion ya lo
            # deja con 2+ condados, asi que no vuelve a calificar si por
            # error se lo procesara de nuevo).
            dsu.union(elegido, root)

    cbsa_ya_absorbio = set()

    # ---- CASO 3: todo condado sin CBSA DEBE terminar en algun grupo. Se
    # repite en pasadas hasta que ya no hay cambios (una fusion puede abrir
    # nuevas oportunidades de vecindad para otro condado en la pasada
    # siguiente). En este punto, los condados sin-CBSA que ya fueron
    # reclamados por un CBSA en el paso anterior NO forman parte de este
    # calculo -- ya cuentan como CBSA y quedaron fuera del "pool". Prioridad
    # de cada condado/grupo sin-cbsa que todavia necesita union:
    #   1) otro condado sin-cbsa que TODAVIA esta solo (nadie se le unio
    #      aun), el de menor poblacion
    #   2) si no hay ninguno solo, un grupo sin-cbsa YA FORMADO (2+
    #      condados), el de menor poblacion
    #   3) si NO tiene ningun vecino sin-cbsa (rodeado solo de CBSA):
    #        3a) CBSA vecino que originalmente era de 1 solo condado (aunque
    #            ya se haya fusionado con otro originalmente-solo en el Caso
    #            2), sin CSA -- el de menor poblacion
    #        3b) si no hay 3a, CBSA vecino que originalmente tenia 2+
    #            condados, sin CSA -- el de menor poblacion
    #        3c) si no hay 3a ni 3b, CBSA vecino que SI es parte de un CSA
    #            (ultimo recurso) -- el de menor poblacion
    #   Si de verdad no tiene NINGUN vecino (caso aislado real, ej. una isla
    #   sin nada alrededor), se queda solo y se marca para revisar.
    #   IMPORTANTE: un CBSA solo puede absorber UN condado/grupo sin-CBSA en
    #   total (nivel 3a, 3b o 3c, o el reclamo del Caso 2 de mas arriba) --
    #   una vez que lo hace, queda cerrado para siempre y no puede ser
    #   elegido de nuevo por otro condado sin-CBSA.
    cambio = True
    while cambio:
        cambio = False
        raices_sin_cbsa = sorted(
            {dsu.find(i) for i in sin_cbsa if dsu.find(i) not in raiz_tiene_cbsa},
            key=raiz_pop, reverse=True,
        )
        for root in raices_sin_cbsa:
            root = dsu.find(root)
            if root in raiz_tiene_cbsa:
                continue  # ya fue absorbido por un cbsa en una pasada anterior
            if not raiz_sin_cbsa_es_solo(root):
                continue  # ya tiene con quien (2+ condados): resuelto, no sigue buscando

            candidatos_sin_cbsa = vecinos_sin_cbsa(root)
            if candidatos_sin_cbsa:
                solos = {r for r in candidatos_sin_cbsa if raiz_sin_cbsa_es_solo(r)}
                pool = solos if solos else candidatos_sin_cbsa
                elegido = min(pool, key=raiz_pop)
                dsu.union(root, elegido)
                cambio = True
                continue

            # sin ningun vecino sin-cbsa: rodeado de CBSA -> 3 niveles de prioridad
            vecinos_root = raiz_vecinos(root)
            candidatos_cbsa = {dsu.find(v) for v in vecinos_root
                                if dsu.find(v) in raiz_tiene_cbsa and dsu.find(v) not in cbsa_ya_absorbio}
            if not candidatos_cbsa:
                continue  # aislado de verdad (o todos los cbsa vecinos ya estan cerrados)

            nivel_3a = {r for r in candidatos_cbsa if raiz_cbsa_originalmente_solo(r) and not raiz_es_de_csa(r)}
            nivel_3b = {r for r in candidatos_cbsa if not raiz_cbsa_originalmente_solo(r) and not raiz_es_de_csa(r)}
            nivel_3c = {r for r in candidatos_cbsa if raiz_es_de_csa(r)}

            if nivel_3a:
                elegido = min(nivel_3a, key=raiz_pop)
            elif nivel_3b:
                elegido = min(nivel_3b, key=raiz_pop)
            else:
                elegido = min(nivel_3c, key=raiz_pop)

            cbsa_ya_absorbio.add(elegido)
            dsu.union(root, elegido)
            cambio = True

    # ---- LIMPIEZA FINAL: cualquier grupo (de cualquier tipo) que quede con
    # menos de POBLACION_MINIMA intenta fusionarse con su vecino de menor
    # poblacion (de cualquier tipo), repitiendo hasta superar el minimo o
    # quedarse sin vecinos (se marca Revisar_manual mas abajo).
    cambio = True
    intentos = 0
    while cambio and intentos < 50:
        cambio = False
        intentos += 1
        for root in {dsu.find(i) for i in idxs}:
            if raiz_pop(root) >= POBLACION_MINIMA:
                continue
            candidatos = {dsu.find(v) for v in raiz_vecinos(root) if dsu.find(v) != root}
            if candidatos:
                elegido = min(candidatos, key=raiz_pop)
                dsu.union(root, elegido)
                cambio = True

    resultado = {}
    grupos_finales = defaultdict(list)
    for i in idxs:
        grupos_finales[dsu.find(i)].append(i)

    for n, (root, miembros) in enumerate(grupos_finales.items(), start=1):
        pob_total = sum(pop[m] for m in miembros)
        nombres_cbsa = sorted({cbsa[m] for m in miembros if cbsa[m] not in (XXX, None) and pd.notna(cbsa[m])})
        nombres_csa = sorted({csa[m] for m in miembros if csa[m] not in (XXX, None) and pd.notna(csa[m])})
        if nombres_cbsa:
            nombre_txt = nombres_cbsa[0] if len(nombres_cbsa) == 1 else " + ".join(nombres_cbsa)
        elif nombres_csa:
            nombre_txt = nombres_csa[0] if len(nombres_csa) == 1 else " + ".join(nombres_csa)
        else:
            nombre_txt = f"Grupo sin CBSA {n}"
        revisar = pob_total < POBLACION_MINIMA
        grupo_id = f"{nombre_grupo}-{n:03d}"
        for m in miembros:
            resultado[m] = (grupo_id, nombre_txt, pob_total, revisar)

    return resultado


def generar_mapa(gdf_grupo, nombre_grupo):
    gdf_grupo = gdf_grupo.to_crs("EPSG:5070")
    fig, ax = plt.subplots(figsize=(max(10, min(40, len(gdf_grupo) * 0.6)),) * 2)

    grupos_unicos = sorted(gdf_grupo["Nuevo_Grupo_ID"].unique())
    colores = plt.get_cmap("tab20", max(len(grupos_unicos), 1))
    color_de = {g: colores(i % 20) for i, g in enumerate(grupos_unicos)}
    gdf_grupo["_color"] = gdf_grupo["Nuevo_Grupo_ID"].map(color_de)

    gdf_grupo.plot(ax=ax, color=gdf_grupo["_color"], edgecolor="white", linewidth=0.3, zorder=1)

    # contorno negro alrededor de los condados que forman cada grupo nuevo
    disuelto = gdf_grupo.dissolve(by="Nuevo_Grupo_ID")
    disuelto.boundary.plot(ax=ax, color="black", linewidth=1.4, zorder=2)

    ax.set_aspect("equal")
    fig.canvas.draw()
    renderer = fig.canvas.get_renderer()

    FONT_MIN, FONT_MAX = 3.0, 11.0
    raiz_area = gdf_grupo.geometry.area.pow(0.5)
    a_min, a_max = raiz_area.min(), raiz_area.max()

    def tamaño_objetivo(area_condado):
        raiz = area_condado ** 0.5
        if a_max <= a_min:
            return (FONT_MIN + FONT_MAX) / 2
        frac = (raiz - a_min) / (a_max - a_min)
        return FONT_MIN + (FONT_MAX - FONT_MIN) * frac

    for _, row in gdf_grupo.iterrows():
        punto = row.geometry.representative_point()
        texto = f"{int(row['POP20']):,}"
        minx, miny, maxx, maxy = row.geometry.bounds
        p0 = ax.transData.transform((minx, miny))
        p1 = ax.transData.transform((maxx, maxy))
        ancho_disp = abs(p1[0] - p0[0])
        alto_disp = abs(p1[1] - p0[1])

        objetivo = tamaño_objetivo(row.geometry.area)
        t = ax.annotate(texto, (punto.x, punto.y), ha="center", va="center", fontsize=objetivo,
                         path_effects=[pe.withStroke(linewidth=1.5, foreground="white")], zorder=5)
        bbox = t.get_window_extent(renderer=renderer)
        if bbox.width > 0 and bbox.height > 0:
            factor = min(ancho_disp / bbox.width, alto_disp / bbox.height) * 0.85
            factor = min(factor, 1.0)  # solo achica si no entra; nunca agranda mas alla del objetivo por area
            t.set_fontsize(max(1.5, objetivo * factor))

    ax.set_axis_off()
    ax.set_title(f"Nuevos grupos — {nombre_grupo.replace('_', ' ').title()}", fontsize=15, fontweight="bold")
    fig.tight_layout()
    ruta = f"{OUTPUT_DIR}\\nuevos_grupos_para_verificar_{nombre_grupo}.png"
    fig.savefig(ruta, dpi=300, bbox_inches="tight")
    plt.close(fig)
    print(f"[{nombre_grupo}] mapa -> {ruta}")


def main():
    condados = gpd.read_file(GPKG_PATH, layer="condados")
    condados["grupo_estado"] = condados["State"].map(ESTADO_A_GRUPO)
    condados["POP20"] = condados["POP20"].fillna(0)

    sin_grupo = condados["grupo_estado"].isna().sum()
    if sin_grupo:
        print(f"AVISO: {sin_grupo} condados con un estado que no esta en ninguno de los 19 grupos, se ignoran.")

    filas_resultado = []
    for nombre_grupo, _estados in GRUPOS_ESTADOS:
        gdf_grupo = condados[condados["grupo_estado"] == nombre_grupo].copy()
        if len(gdf_grupo) == 0:
            print(f"[{nombre_grupo}] sin condados, se salta.")
            continue

        resultado = procesar_grupo_estado(gdf_grupo, nombre_grupo)
        gdf_grupo["Nuevo_Grupo_ID"] = gdf_grupo.index.map(lambda i: resultado[i][0])
        gdf_grupo["Nuevo_Grupo_Nombre"] = gdf_grupo.index.map(lambda i: resultado[i][1])
        gdf_grupo["Nuevo_Grupo_Poblacion"] = gdf_grupo.index.map(lambda i: resultado[i][2])
        gdf_grupo["Revisar_manual"] = gdf_grupo.index.map(lambda i: resultado[i][3])

        for _, row in gdf_grupo.iterrows():
            filas_resultado.append({
                "State": row["State"],
                "County": row.get("NAMELSAD", row.get("County", "")),
                "Grupo_Estados": nombre_grupo,
                "CBSA_original": row[CBSA_COL],
                "CSA_original": row[CSA_COL],
                "Nuevo_Grupo_ID": row["Nuevo_Grupo_ID"],
                "Nuevo_Grupo_Nombre": row["Nuevo_Grupo_Nombre"],
                "Nuevo_Grupo_Poblacion": row["Nuevo_Grupo_Poblacion"],
                "Revisar_manual": row["Revisar_manual"],
            })

        generar_mapa(gdf_grupo, nombre_grupo)

    df_resultado = pd.DataFrame(filas_resultado)
    df_resultado.to_excel(OUTPUT_XLSX, index=False)
    n_revisar = df_resultado["Revisar_manual"].sum()
    print(f"\nListo. Excel guardado en: {OUTPUT_XLSX}")
    print(f"Grupos que quedaron con menos de {POBLACION_MINIMA} hab y no encontraron con quien fusionarse: {n_revisar} filas")


if __name__ == "__main__":
    main()
