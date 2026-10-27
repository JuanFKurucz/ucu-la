# M1 · Laboratorio de comportamiento

| Módulo | Clase | En clase | IA permitida |
|---|---|---|---|
| M1 | 1 | 15 min | como tutora: te explica (A1) |

**Entregás:** diagnóstico del cuaderno, fila de 1 experimento.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/JuanFKurucz/ucu-la/blob/main/notebooks/m1_python_de_cero.ipynb) Cuaderno de la clase 1: Python desde cero.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/JuanFKurucz/ucu-la/blob/main/notebooks/m1_laboratorio.ipynb) Abrir el cuaderno en Colab (el lab de esta práctica).


## Objetivo

- Lo del cuaderno de Python (`m1_python_de_cero.ipynb`), con un **LLM real**: **una** variable por vez, como en ML.
- Separar lo que viste de por qué creés que pasó.

## Punto de partida

- **Lab:** el cuaderno de Colab de esta práctica (botón «Open in Colab», arriba). Corre en el navegador: no instalás nada.
- **Clave:** la de Gemini, en los **Secretos** de Colab (la llave de la izquierda), como en el cuaderno de Python.
- **Temperatura:** solo cambia algo con `google_genai:gemini-3.1-flash-lite`, el modelo del lab (`MODELO_T`). Gemini 3.5 Flash-Lite la ignora.
- **Decidís:** experimento y predicción, antes de correr.

## Arranque

1. Abrí el lab y corré las celdas en orden con **Shift + Enter** hasta «Experimento 1».
2. Escribí tu predicción en `PREDICCION` y corré la celda de `correr(...)`.
3. **Qué deberías ver:** una tabla con cada respuesta y sus tokens, y `T=0.0: 1 distintas de 5`.

**Primer control · 10–15 min:** tabla con las respuestas y sus tokens.

- **Saldo:** una llamada por repetición. Tope: 20 por persona (`TOPE`, lo pone el lab). Experimento 1: 10; experimento 2: 2.

<details markdown="1"><summary>¿Sale un error?</summary>

- **«No pude leer el secreto GOOGLE_API_KEY»:** el secreto se llama exactamente así y tiene activado **Acceso del notebook**.
- **«503»:** Gemini está saturado, no es tu código. Esperá un minuto o hacé el experimento 2.
- **Otro mensaje:** copialo tal cual en el foro «Consultas» del aula virtual.

</details>

## En clase

Elegí **uno**:

| # | Qué cambiás | Qué mirás |
|---|---|---|
| 1 | `TEMPERATURA` 0 y después 1, con `REPETICIONES = 5` | ¿Cuántas distintas, de 5? |
| 2 | La misma frase en español y en inglés (`ES` y `EN`), 1 llamada cada una | ¿Tokens de entrada? ¿El español usa más? |

1. **Predecí** por escrito, en `PREDICCION`.
2. **Corré** y anotá antes de cambiar otra perilla.
3. **Cambiá una sola cosa** y corré de nuevo.
4. **Explicá** con el cuaderno: ejercicios 7 y 9 (temperatura) u 8 (tokens). Con 0 y distintas no es error: en una API, 0 no garantiza lo mismo.

## Controles

- Mismo prompt en las dos condiciones (salvo el idioma).
- 5 repeticiones al medir variación.
- La explicación no contradice lo que viste.

## Resultado probado

Una fila: **predicción · resultado · explicación**. Ej.: «Predije 5 iguales · salieron 3 distintas · Gemini 3.5 Flash-Lite ignora la temperatura».

## Evidencia

En `evidencia/m1.md` de tu repo (sin nota):

- La fila del experimento y una frase «Esto **no** demuestra que…».
- Tu **diagnóstico** del cuaderno: la última celda lo arma sola con lo que te salió.

## Para casa

- Terminá el cuaderno (el ejercicio 10, JSON, se usa el miércoles).
- Una fila más: un **dato al principio, medio o final** de 2–3 páginas («¿lo encuentra?»), o **algo que no existe**, con y sin «si no sabés, decilo» («¿inventa?»).

## Microdefensa

- ¿Qué contradijo tu predicción? ¿Qué lo explicaría?
- Confianza del 1 al 4.
