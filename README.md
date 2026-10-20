# Entrega 3 – Análisis de Datos · Business Intelligence

**Instituto Tecnológico Metropolitano (ITM)** · Ingeniería de Sistemas
**Curso:** Análisis de Datos · **Docente:** Daniel Alexis Nieto Mora · **Semestre:** 2026-2
**Equipo 2 · Tema:** Clientes bancarios · **Herramienta:** Power BI

> **En una frase:** el banco pierde 1 de cada 5 clientes (20,4 %) y casi 1 de cada 4 pesos (24,3 %). Se van más los mayores de 40 años, los inactivos, los de Alemania y los que tienen 1 o 3 y 4 productos. Con un modelo de riesgo identificamos **293 clientes que siguen en el banco y están en riesgo alto**: son la lista para llamar primero.

## Integrantes

- Isabel Johana Hoyos Echavarria
- Jhon Eduardo Zabala Garzón
- Sebastian Villa Castillo
- Edwin Ramírez Gonzáles

## Video

https://youtu.be/fL7XLfV54hc

## Dataset elegido

| | |
|---|---|
| **Base de datos final** | **Bank Customer Churn** (Kaggle): [mathchi/churn-for-bank-customers](https://www.kaggle.com/datasets/mathchi/churn-for-bank-customers), archivo `churn.csv` |
| Qué contiene | 10.000 clientes de un banco con presencia en Francia (5.014), Alemania (2.509) y España (2.477). 14 columnas: puntaje de crédito, país, género, edad, años como cliente, saldo, productos, tarjeta, actividad, salario estimado y si el cliente se fue (`Exited`) |
| Tipo de dato | Secundario: lo recogió alguien más. No sabemos cómo se midió y **no trae fechas** |
| Calidad | Sin nulos, sin filas repetidas, un cliente por fila. Todos los valores tienen sentido (edades 18 a 92, puntajes 350 a 850, saldos sin negativos) |
| Qué se descarta | `RowNumber` (solo cuenta filas). `CustomerId` se queda como llave y `Surname` solo en la lista de detalle |
| Resultado de prepararlo | 12 tablas en `data/processed/` (1 de hechos y 11 de dimensiones, incluida `dim_riesgo` del modelo) |

## Objetivo

**Problema de negocio:** *el banco pierde 1 de cada 5 clientes. ¿Qué perfil de cliente se va y qué puede hacer el banco para retenerlo?*

Para responderlo seguimos tres preguntas:

1. ¿Cómo cambia el abandono según país, género y edad?
2. ¿La actividad del cliente y la cantidad de productos se relacionan con que se vaya?
3. ¿Qué clientes valiosos están en riesgo y a quién conviene llamar primero?

El producto final es un dashboard en Power BI que funciona como **termómetro del abandono**: cada indicador tiene un número, un semáforo y una acción.

## Metodología

| Fase | Qué hicimos | Dónde |
|---|---|---|
| 1. Comprensión | Contexto del dataset, calidad, variable objetivo, problema y tres preguntas | `01_comprension.ipynb` |
| 2. Hipótesis | Cuatro hipótesis con H0 y H1, supuestos, p-valor, tamaño del efecto e intervalos de confianza | `02_hipotesis.ipynb` |
| 3. Preparación | ETL, modelo en estrella (1 tabla de hechos y 11 dimensiones), comprobación de que no se perdió ni se inventó nada | `03_preparacion_modelo.ipynb` |
| 4. KPIs y riesgo | 10 KPIs con semáforo, medidas DAX y modelo de riesgo por cliente | `04_kpis_y_modelo_riesgo.ipynb` |
| 5. Dashboard | Cinco páginas en Power BI con una historia de principio a fin | `dashboard/churn_clientes.pbix` |

**Reglas que fijamos antes de mirar resultados:** α = 0,05, pruebas bilaterales, y siempre reportar el tamaño del efecto (con 10.000 filas casi todo sale "significativo", así que el p-valor solo no basta). Como hicimos cuatro pruebas a la vez, repetimos la decisión con la corrección de Bonferroni (α = 0,05 / 4 = 0,0125).

## Hipótesis y resultados estadísticos

Cada hipótesis compara a los clientes que se fueron con los que se quedaron.

| # | Hipótesis | Prueba | Resultado | Decisión |
|---|---|---|---|---|
| H1 · Edad | **H0:** la edad promedio de quienes se van es igual a la de quienes se quedan. **H1:** es distinta | t de Welch (respaldo: Mann-Whitney) | Los que se fueron tienen 44,8 años en promedio, contra 37,4: 7,4 años más (IC 95 %: 7,0 a 7,9). p = 4,7 × 10⁻¹⁷⁹. d de Cohen = 0,74 (efecto mediano) | Se rechaza H0 |
| H2 · País | **H0:** el abandono es independiente del país. **H1:** depende del país | Chi-cuadrado | Alemania pierde 32,4 %, Francia 16,2 % y España 16,7 %. En Alemania se fueron 814 clientes y se esperaban unos 511. p = 3,8 × 10⁻⁶⁶. V de Cramér = 0,17 (pequeño) | Se rechaza H0 |
| H3 · Actividad | **H0:** el abandono es independiente de ser cliente activo. **H1:** depende | Chi-cuadrado (corrección de Yates) | Inactivos 26,9 % contra activos 14,3 %: 12,6 puntos más (IC 95 %: 11,0 a 14,2). p = 8,8 × 10⁻⁵⁵. V = 0,16 (pequeño) | Se rechaza H0 |
| H4 · Productos | **H0:** el abandono es independiente de la cantidad de productos. **H1:** depende | Chi-cuadrado | 1 producto: 27,7 %. 2 productos: 7,6 %. 3 productos: 82,7 %. 4 productos: 100 %. p menor que 10⁻³⁰⁰. V = 0,39 (mediano, la relación más fuerte) | Se rechaza H0 |

**Con Bonferroni (α = 0,0125) las cuatro se rechazan igual.** Lo que cambia entre ellas es la fuerza: productos y edad son las más importantes; país y actividad, de efecto pequeño. Con una regresión logística que mira todo junto, el efecto de 3 y 4 productos se mantiene dentro de cada país, edad, género y actividad.

**Qué NO dicen estas pruebas:**
- Rechazar H0 no prueba H1, solo que los datos son poco compatibles con H0.
- Relación no es causa: que los inactivos se vayan más no prueba que la inactividad los haga irse.
- La edad no sube en línea recta: el abandono llega a su punto más alto entre los 41 y los 60 años y baja después.

## KPIs

Son 10 KPIs. Los nueve primeros alertan (están en `data/processed/kpi_definiciones.csv`) y cada uno tiene una fórmula, un semáforo y una acción; el décimo, "lo que no pesa", descarta variables. **Los umbrales son una propuesta del grupo**: el dataset no trae metas del banco, y en la práctica las definiría el banco.

| KPI | Qué mide | Valor | Verde hasta | Rojo desde | Estado | Qué hacer si se enciende |
|---|---|---|---|---|---|---|
| K1 Tasa de abandono | Clientes que se fueron sobre el total | 20,4 % | 18 % | 22 % | Amarillo | Activar el plan de retención en toda la base |
| K2 Saldo en fuga | Dinero que se fue sobre el saldo total | 24,3 % | 20 % | 25 % | Amarillo | Priorizar clientes con saldo alto |
| K3 Riesgo de Alemania | Abandono de Alemania sobre el del resto | 1,99 veces | 1,2 | 1,5 | Rojo | Revisar oferta y servicio en Alemania |
| K4 Riesgo de inactivos | Abandono de inactivos sobre el de activos | 1,88 veces | 1,2 | 1,5 | Rojo | Contactar al cliente cuando deja de usar el banco |
| K5 Abandono de 41 a 60 años | Tasa de ese grupo de edad | 39,7 % | 20 % | 30 % | Rojo | Diseñar un servicio para este grupo |
| K6 Clientes con un solo producto | Porcentaje de la base | 50,8 % | 40 % | 50 % | Rojo (frontera) | Venta cruzada del segundo producto |
| K7 Abandono con 3 o más productos | Tasa de ese grupo | 85,9 % | 30 % | 50 % | Rojo | Investigar esos productos antes de vender más |
| K8 Saldo en riesgo alto | Saldo de clientes en riesgo alto que siguen, sobre el saldo de quienes siguen | 4,8 % | 5 % | 10 % | Verde | Pasar la lista de clientes al equipo comercial |
| K9 Brecha de género | Abandono de mujeres menos el de hombres | 8,6 puntos | 3 | 6 | Rojo | Revisar productos o canales que expliquen la diferencia |
| K10 Lo que no pesa | Diferencia entre el mejor y el peor grupo de score, salario y antigüedad | 3,4 · 1,6 · 1,5 puntos | n/a | n/a | Descarta | No hacer campañas por score, salario o antigüedad |

Que casi todo esté en rojo no significa que el banco esté colapsando: los KPIs por grupo encienden donde se concentra el abandono, y los umbrales son nuestros. Al filtrar en el dashboard (por ejemplo, solo Francia) los semáforos cambian y se ve dónde el problema es grave.

## Modelo de riesgo

Los KPIs dicen **qué grupos** se van; el modelo dice **qué clientes**. Comparamos tres modelos con 25 % de los clientes apartados (datos que no vieron al aprender):

| Modelo | AUC en datos de prueba |
|---|---|
| Regresión logística | 0,857 |
| Random Forest | 0,874 |
| **Gradient Boosting (elegido)** | **0,879** |

Los tres quedan entre 0,857 y 0,879: el patrón es claro y no depende de un modelo complicado. Para dar un riesgo a **todos** los clientes usamos validación cruzada de 5 partes, así nadie recibe un riesgo "memorizado".

- **Qué pesa más:** la edad (al desordenarla, el AUC cae 0,103, tres veces más que cualquier otra), luego productos, actividad, saldo y Alemania. El score de crédito casi no pesa.
- **Niveles de riesgo** (la tasa real sube de forma ordenada de un nivel a otro):

| Nivel | Riesgo | Clientes | Se fue de verdad |
|---|---|---|---|
| Bajo | menos de 20 % | 7.010 | 7,3 % |
| Medio | de 20 % a 50 % | 1.734 | 32,4 % |
| Alto | 50 % o más | 1.256 | 76,7 % |

El nivel Alto concentra 963 de los 2.037 clientes que se fueron (cerca de la mitad) con solo el 12,6 % de la base. De esos 1.256, **293 siguen en el banco**: son los que todavía se pueden salvar.

## Dashboard (Power BI)

Cinco páginas, una por paso de la historia: **situación → hallazgo → decisión**. Los segmentadores permiten filtrar y todos los semáforos cambian con ellos.

| Página | Pregunta que responde | Qué se ve |
|---|---|---|
| 1. Termómetro | ¿Qué tan grave es? | Tasa de abandono (20,4 %), saldo en fuga (24,3 %), seis alertas por grupo con semáforo y la lista de los 8 clientes de mayor riesgo que siguen |
| 2. ¿Quién se va? | ¿Qué perfil tiene el que se va? | Abandono por edad, país, género y saldo. Panel de "lo que no pesa": antigüedad, score y salario |
| 3. Productos y actividad | ¿Qué es lo que más sorprende? | Abandono por número de productos y actividad, y dos matrices de color (productos por país y por actividad) |
| 4. A quién llamar | ¿Quién en concreto? | 293 clientes en riesgo alto que siguen (75 con saldo alto), 27,8 millones de saldo, perfil de la lista y tabla de trabajo exportable |
| 5. Qué proponemos | ¿Qué decisión tomamos? | Cinco propuestas atadas a un KPI, con una barra de "qué porcentaje logramos retener" que recalcula el saldo salvado |

## Propuestas

El **saldo que se espera perder** es el saldo de quienes siguen en el grupo × la tasa de abandono histórica del grupo. El **porcentaje retenido** (10 %, 20 % o 30 %) es un supuesto nuestro, no una predicción.

| # | Propuesta | Evidencia | Clientes que siguen | Saldo que manejan | Saldo que se espera perder | Salvado con 20 % |
|---|---|---|---|---|---|---|
| 1 | Llamar esta semana a los clientes de riesgo alto que siguen | Del nivel Alto se fue el 76,7 %; del Bajo, 7,3 % | 293 | 27,8 M | 21,3 M | 4,3 M |
| 2 | Programa de reactivación para inactivos de 41 a 60 años | Inactivos se van 1,88 veces más; en este grupo se fue el 51 % | 764 | 57,4 M | 29,3 M | 5,9 M |
| 3 | Revisar oferta y servicio en Alemania | Alemania pierde 1,99 veces más que el resto | 1.695 | 202,4 M | 65,7 M | 13,1 M |
| 4 | Ofrecer un segundo producto a quienes tienen uno solo | 1 producto: 27,7 %; 2 productos: 7,6 % (asociación, no causa comprobada) | 3.675 | 371,4 M | 102,9 M | 20,6 M |
| 5 | Investigar los productos 3 y 4 antes de venderlos más | Con 3 o 4 productos se va el 85,9 % | 46 | 1,2 M | 1,0 M | 0,2 M |

Los grupos se traslapan (un cliente alemán inactivo de 45 años cuenta en varios), así que las filas no se suman. El costo de cada acción lo definiría el banco; el esfuerzo se mide en clientes a contactar, de 293 llamadas a 3.675 ofertas. **Por dónde empezar:** 1 y 2 (pocos clientes, mucho riesgo), en paralelo 5 (casi sin costo), luego 3 y 4.

## Conclusiones

1. **El problema es mayor en dinero que en personas.** Se va el 20,4 % de los clientes pero el 24,3 % del saldo: los que se van tienen en promedio más dinero.
2. **El abandono se concentra en cuatro lugares:** mayores de 40 (34 % de 41 a 50 y 56 % de 51 a 60), Alemania (el doble que Francia y España), clientes inactivos (casi el doble que los activos) y la cantidad de productos.
3. **Productos es la relación más fuerte.** Con 2 productos casi nadie se va (7,6 %); con 1 se va el 27,7 % y con 3 o 4 se va el 85,9 %. La relación se mantiene dentro de cada país, edad y actividad, así que no la explican las otras variables.
4. **Lo que no pesa también es un hallazgo.** Entre el mejor y el peor grupo, el score cambia 3,4 puntos, el salario 1,6 y la antigüedad 1,5, frente a 78 puntos en productos y 49 en edad. Una campaña por ingreso, score o años como cliente no tiene respaldo en los datos.
5. **Se puede actuar sobre personas, no solo sobre grupos.** El modelo identifica 293 clientes que siguen con riesgo alto (75 con saldo alto) y 27,8 millones de saldo en juego. Es una lista que un equipo comercial atiende en una semana.
6. **Decisión que deja:** empezar por llamar a los de riesgo alto y reactivar a los inactivos de 41 a 60 años; vender un segundo producto parece sano, pero vender un tercero no hasta entender qué pasa con esos productos.

## Cómo reproducirlo

1. Clonar el repo y crear un entorno: `python -m venv .venv`, activarlo e instalar con `pip install -r requirements.txt`.
2. Ejecutar los notebooks en orden: `01` a `04`. El notebook 01 descarga el dataset de Kaggle con `kagglehub`; el 03 exporta las tablas a `data/processed/` y el 04 agrega el riesgo.
3. Abrir `dashboard/churn_clientes.pbix` en Power BI Desktop (los datos ya van dentro del archivo). El modelo es una tabla de hechos (`fact_clientes`) con 11 dimensiones conectadas uno a muchos.

## Estructura del repositorio

- `notebooks/`: análisis por fases (01 comprensión, 02 hipótesis, 03 preparación y modelo, 04 KPIs y modelo de riesgo).
- `data/processed/`: tablas limpias (hechos y dimensiones), diccionario de datos y definición de KPIs.
- `dashboard/`: archivo `.pbix` de Power BI.
- `requirements.txt`: librerías de Python.