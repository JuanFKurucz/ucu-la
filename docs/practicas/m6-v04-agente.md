# M6 · v0.4 Agente con herramientas

| Módulo | Clase | En clase | IA permitida |
|---|---|---|---|
| M6 | 5 | 105 min | como copiloto: escribe con vos y vos lo explicás (A2) |

**Entregás:** v0.4 en el repo, suite de selección propia, 2 corridas, fallo manejado.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/JuanFKurucz/ucu-la/blob/main/notebooks/m6_v04_agente.ipynb) Abrir el cuaderno en Colab (para quien no tiene el entorno instalado).


## Objetivo

- Que tu asistente **elija bien**: documentos, internet o calculadora.
- Que ante un fallo **lo diga en vez de inventar**. Es la **v0.4** (requisito 2).

## Punto de partida

- **Tu repo**, con `v02_rag.py` y `medir.py`. Creás `v04_agente.py`.
- **De la plantilla:** `herramientas.py` (`buscar_web` y `calcular`, listas) y `datos/m6_seleccion.jsonl` (12 preguntas con la herramienta esperada). ¿No están en tu repo? Bajalos de la plantilla (*Download raw file*) a las mismas carpetas.
- **El contrato:** `responder(pregunta)` devuelve lo de v0.2 más `herramientas`, las que pidió.
- **¿Sin el entorno instalado?** Lo mismo en Colab: el botón «Open in Colab» de arriba.
- **Con tu asistente de IA:** pegale cada paso y pedile «explicame cada renglón».

Un agente es un flujo de Power Automate donde la condición la decide el modelo: lee la pregunta y la descripción de cada herramienta, pide una (o ninguna), lee el resultado y vuelve a decidir.

## Arranque

1. **Lo que el modelo ve:** `uv run python herramientas.py`. **Qué deberías ver:** `25`, un resultado web grabado y el nombre, los parámetros y la descripción de cada herramienta. El modelo elige solo con eso.
2. **Tu herramienta.** En `v04_agente.py`, `@tool` vuelve tu `buscar` de M4 una herramienta; su descripción es lo que el modelo lee:

    ```python
    from langchain.agents import create_agent
    from langchain.agents.middleware import ModelCallLimitMiddleware
    from langchain.tools import tool
    from herramientas import Medidor, buscar_web, calcular  # vienen en la plantilla
    from v02_rag import buscar, modelo  # tu búsqueda y tu modelo de M4
    @tool
    def buscar_documentos(consulta: str) -> str:
       """Busca en mis documentos sobre LangGraph y LangChain."""  # poné tu tema
       return "\n\n".join(f"[{h['doc']}]\n{h['texto'][:600]}" for h in buscar(consulta))
    ```

3. **El agente**, con un tope de llamadas en código (no en el prompt). `Medidor` mira cada llamada y cada herramienta, como el monitor de rendimiento de Power BI:

    ```python
    SISTEMA = ("Usá buscar_documentos para LangGraph y LangChain, buscar_web para lo reciente o externo y calcular para "
              "cualquier cuenta. Si una herramienta devuelve ERROR, decilo y no inventes. Respondé en español, breve.")
    agente = create_agent(modelo, tools=[buscar_documentos, buscar_web, calcular], system_prompt=SISTEMA,
                         middleware=[ModelCallLimitMiddleware(run_limit=6)])  # hasta 6 llamadas por pregunta
    def responder(pregunta: str) -> dict:
       medidor = Medidor()  # cuenta llamadas, herramientas, documentos, tokens y segundos
       mensajes = agente.invoke({"messages": [("user", pregunta)]}, {"callbacks": [medidor]})["messages"]
       return {"respuesta": mensajes[-1].text, **medidor.resumen()}
    if __name__ == "__main__": print(responder("¿Qué hace interrupt() en LangGraph?"))
    ```

    Corré `uv run python v04_agente.py`. **Qué deberías ver:** `'herramientas': ['buscar_documentos']`, `'llamadas': 2` (pide; lee y responde) y la respuesta.

**Primer control · 10–15 min:** ves qué herramienta pidió.

## En clase

1. **Tu tema:** en la descripción de `buscar_documentos` y en `SISTEMA`, cambiá «LangGraph y LangChain» por el tuyo.

    ![La herramienta buscar_documentos, abreviada: @tool la vuelve herramienta; el modelo ve el nombre, los tipos y la descripción, que se reescribe con tu tema](../assets/herramienta-descripcion.svg)

2. **Medí la elección** con `web = "grabada"` en `config.toml` (mide la elección, no la respuesta). `medir.py` queda así:

    ```python
    import json, pandas as pd, v04_agente
    suite = [json.loads(linea) for linea in open("datos/m6_seleccion.jsonl", encoding="utf-8") if linea.strip()]
    tabla = pd.DataFrame([{**caso, "esperadas": caso["herramientas"], **v04_agente.responder(caso["pregunta"])} for caso in suite])
    tabla["eligio_bien"] = [set(e) == set(h) for e, h in zip(tabla["esperadas"], tabla["herramientas"])]
    print(tabla[["id", "tipo", "eligio_bien", "herramientas", "llamadas"]].to_string())
    ```

    **Qué deberías ver:** una fila por pregunta. Cada `False` es una casilla de tu matriz de confusión: qué esperabas y qué pidió.

3. **Tu suite:** `datos/mi_seleccion.jsonl`, unas 10 preguntas, al menos una de cada `tipo`: `corpus` (tus documentos), `web`, `calculo`, `ninguna` (un saludo, con `"herramientas": []`) y `combinada` (con las dos, por ejemplo `["buscar_documentos", "calcular"]`). Cambiá el archivo en `medir.py`.
4. **Predecí** en cuáles se confunde y medí (corrida **a**).
5. **UNA cosa** (descripción, `SISTEMA`, tope o modelo) y medí de nuevo (corrida **b**).
6. **Fallá a propósito:** `web = "caida"`, y en el último renglón de `v04_agente.py` (el `print` del final) cambiá la pregunta por «¿Cuál es la última versión de LangGraph?». ¿Avisa o inventa? ¿Pidió `buscar_web`? Volvé a `web = "real"`.
7. *Commit* («v0.4: …») y *Push*.

## Controles

- Misma suite y modo de web en las dos corridas.
- Ninguna pregunta pasa de 6 `llamadas`.
- Ninguna herramienta escribe ni borra.

## Resultado probado

| Corrida | Eligió bien | Llamadas por pregunta | Qué cambié |
|---|---|---|---|
| a | | | (base) |
| b | | | |

## Evidencia

En `evidencia/v0.4.md`, entrega de avance 3 (hasta el mié 18/11, 23:59): tabla, **matriz**, tu cambio y la web caída. Declará la web grabada. Subí `v04_agente.py`, `medir.py` y tu suite.

## Para casa

- 3 preguntas **ambiguas** (`"tipo": "ambigua"`), con la herramienta que elegirías vos.
- Medilas: ¿eligió bien?

## Microdefensa

- Una elección equivocada: ¿descripción, `SISTEMA`, pregunta o modelo?
- Confianza del 1 al 4.
