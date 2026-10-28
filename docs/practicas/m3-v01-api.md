# M3 · v0.1 Tu primer programa con un LLM

| Módulo | Clase | En clase | IA permitida |
|---|---|---|---|
| M3 | 2 | 25 min | como copiloto: escribe con vos y vos lo explicás (A2) |

**Entregás:** v0.1 en el repo, tabla de las dos configuraciones, decisión de configuración, estimación de costo.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/JuanFKurucz/ucu-la/blob/main/notebooks/m3_v01_api.ipynb) Abrir el cuaderno en Colab (para quien no tiene el entorno instalado).


## Objetivo

- Escribís tu **v0.1**: un programa que le pregunta al modelo sobre **tu tema**, guarda respuesta, tokens y tiempo, y no se cae si algo falla.
- Comparás dos configuraciones, **una cosa por vez**, y estimás el costo.

## Punto de partida

- **Tu repo** del proyecto ([plantilla](https://github.com/JuanFKurucz/ucu-la-project)). Creás `v01_api.py` al lado de `llm.py`.
- **El contrato** (README de tu repo): `responder(pregunta)` devuelve un `dict` con `respuesta`, `tokens_entrada`, `tokens_salida` y `latencia_s`; cada pregunta deja un renglón en `logs/v01.jsonl`.
- **¿Sin el entorno instalado?** Lo mismo en Colab: el botón «Open in Colab» de arriba.
- **Con tu asistente de IA** (el que usás para programar): pegale cada paso y pedile «explicame cada renglón». Si sale un error, pegáselo. Cada renglón lo tenés que poder explicar.

## Arranque

1. **Las perillas.** Arriba de todo: en `SISTEMA`, de qué se ocupa tu asistente (si querés, con datos de tu tema pegados: horarios, fechas); en `PREGUNTAS`, **5 de tu tema** que se respondan con eso, entre comillas y con coma. Lo que no le diste, no lo sabe.

    ```python
    import json, os, time
    from llm import cargar_modelo, explicar, nombre_modelo, sin_claves
    SISTEMA = "Sos un asistente para estudiantes. Respondé en español, en dos frases como máximo."
    PREGUNTAS = ["¿Qué es un token en un LLM?", "¿Qué es una alucinación en un LLM?"]
    modelo = cargar_modelo()  # el de config.toml
    ```

    Corré `uv run python v01_api.py`. **Qué deberías ver:** nada, sin errores. Si falta la clave o algo está mal, `cargar_modelo()` te lo dice.

2. **`responder()`.** Abajo, la función: manda la pregunta, muestra la respuesta de a pedazos (*streaming*), los junta y mide.

    ```python
    def responder(pregunta: str) -> dict:
       inicio, junta = time.perf_counter(), None
       for pedazo in modelo.stream([("system", SISTEMA), ("human", pregunta)]):
           print(pedazo.text, end="", flush=True)  # la respuesta aparece de a poco
           junta = pedazo if junta is None else junta + pedazo  # y se va juntando
       print()
       return {"respuesta": junta.text, "latencia_s": round(time.perf_counter() - inicio, 2),
               "tokens_entrada": junta.usage_metadata["input_tokens"], "tokens_salida": junta.usage_metadata["output_tokens"]}
    ```

    Agregá al final `print(responder(PREGUNTAS[0]))` y corré. **Qué deberías ver:** la respuesta, de a poco, y abajo algo como `{'respuesta': '…', 'latencia_s': 1.2, 'tokens_entrada': 31, 'tokens_salida': 40}`.

3. **Las 5 y el log.** Cambiá ese `print` por el bucle: un reintento si falla, y cada resultado (o error) en un renglón de `logs/v01.jsonl`.

    ```python
    os.makedirs("logs", exist_ok=True)  # la primera vez no existe
    for pregunta in PREGUNTAS:
       print("\n>", pregunta)
       for intento in (1, 2):  # el primero y un reintento
           try:
               fila = responder(pregunta)
               break
           except Exception as error:  # cuota, red, un modelo que no existe…
               fila = {"error": type(error).__name__, "detalle": sin_claves(str(error))[:200]}
               print(f"  Error (intento {intento} de 2):", explicar(error) or fila["error"])
               time.sleep(2)  # un 429 al instante vuelve a fallar
       with open("logs/v01.jsonl", "a", encoding="utf-8") as log:  # "a": agrega, no borra
           log.write(json.dumps({"modelo": nombre_modelo(), "sistema": SISTEMA, "pregunta": pregunta, **fila},
                                ensure_ascii=False) + "\n")
    ```

    **Qué deberías ver:** las 5 respuestas, cada una debajo de su `>`. El log es una tabla, una fila por pregunta: en `logs/v01.jsonl` (abrilo en VS Code), 5 renglones con `modelo`, `tokens_entrada`, `tokens_salida` y `latencia_s`.

**Primer control · 10–15 min:** el log tiene modelo, tokens y latencia de tus 5 preguntas.

<details markdown="1"><summary>¿Sale un error?</summary>

- **`IndentationError`:** dentro de `def`, `for`, `try` y `with`, cada renglón lleva 4 espacios más que el de arriba.
- **`SyntaxError` en `PREGUNTAS`:** falta una comilla, o la coma entre dos preguntas.
- **«config.toml está mal escrito»:** los textos van entre comillas rectas (`model = "google_genai:gemini-3.5-flash-lite"`) y los decimales con punto (`temperature = 0.5`).
- **«429: se terminó la cuota del día…»:** reintentar no sirve. Esperá a que se renueve o cambiá de modelo en `config.toml`.
- **`Error (intento 1 de 2): TypeError`:** ese modelo no devolvió los tokens. Volvé a `google_genai:gemini-3.5-flash-lite`.
- **Otro mensaje:** pegalo en el foro «Consultas».

</details>

## En clase

1. **Predecí** y variá **una** cosa: `SISTEMA` corto vs con reglas («si no estás seguro, decí "no sé"»), o dos modelos (`model` en `config.toml`, por ejemplo `google_genai:gemini-3.1-flash-lite`).
2. **Borrá** `logs/v01.jsonl` y corré una vez con cada configuración. Quedan 10 respuestas.
3. **Un error a propósito:** en `config.toml` poné un modelo que no existe, `google_genai:gemini-que-no-existe`. **Predecí** qué queda en el log y corré. ¿Sirvió el reintento? Después volvé al modelo de antes.
4. **Compará:** guardá esto como `tabla.py` y corré `uv run python tabla.py`. Si variaste el `SISTEMA`, cambiá `"modelo"` por `"sistema"`:

    ```python
    import pandas as pd
    grupos = pd.read_json("logs/v01.jsonl", lines=True).groupby("modelo")  # los errores no tienen tokens: no cuentan
    print(grupos.agg({"tokens_entrada": "mean", "tokens_salida": "mean", "latencia_s": "median"}).round(1).to_string())
    ```


    **Qué deberías ver:** una fila por configuración: tokens promedio y latencia mediana. El modelo que no existe sale con `NaN`: sin respuestas no hay tokens.

5. **Decidí** con qué configuración te quedás y dejala puesta.

## Controles

- Las dos configuraciones, con las mismas 5 preguntas: cambia una sola cosa.
- El log tiene las 10 preguntas y los errores del experimento.
- `.env` **no** aparece en los cambios de GitHub Desktop (`logs/` tampoco: no se sube).
- ¿Una gana por poco en latencia? Corré de nuevo: la primera llamada suele tardar más.

## Resultado probado

| Configuración | Tokens de salida (prom.) | Latencia (mediana) | Respuestas buenas |
|---|---|---|---|
| A: … | | | /5 |
| B: … | | | /5 |

## Evidencia

En `evidencia/v0.1.md`, entrega de avance 1 (hasta el mar 3/11, 23:59): la tabla, qué configuración elegiste y por qué, qué quedó en el log con el modelo que no existe (¿sirvió el reintento?) y el costo del «Para casa». *Commit* («v0.1: …») y *push*.

## Para casa

**Costo mensual** a 1000 preguntas por día, con tus tokens promedio y el precio oficial con fecha ([Gemini](https://ai.google.dev/gemini-api/docs/pricing), [OpenAI](https://developers.openai.com/api/docs/pricing); *Paid tier*):

costo por pregunta = tokens de entrada × precio de entrada / 1.000.000 + tokens de salida × precio de salida / 1.000.000

costo por mes = costo por pregunta × 1000 × 30

Sale en una planilla, una celda por término. En Python: `1_000_000`, `print(f"{costo:.7f}")`.

## Microdefensa

- ¿Qué error vale reintentar y cuál no? ¿Cómo los separarías en el `except`?
- Confianza del 1 al 4.
