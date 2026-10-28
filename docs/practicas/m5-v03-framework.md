# M5 · v0.3 Framework y memoria

| Módulo | Clase | En clase | IA permitida |
|---|---|---|---|
| M5 | 4 | 100 min | como copiloto: escribe con vos y vos lo explicás (A2) |

**Entregás:** v0.3 en el repo, medición antes/después, test de hilos, nota de decisión.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/JuanFKurucz/ucu-la/blob/main/notebooks/m5_v03_memoria.ipynb) Abrir el cuaderno en Colab (para quien no tiene el entorno instalado).


## Objetivo

- Tu RAG, ahora armado con LangChain, **sin que empeore** (lo mide `medir.py`).
- Memoria por conversación, sin mezclar hilos. Es la **v0.3**.

## Punto de partida

- **Tu repo**, con `v02_rag.py` y `medir.py` de M4. Creás `v03_memoria.py` en la misma carpeta.
- **El contrato:** `responder(pregunta, hilo)` devuelve lo de v0.2 y recuerda lo dicho en el mismo `hilo`.
- **¿Sin el entorno instalado?** Lo mismo en Colab: el botón «Open in Colab» de arriba.
- **Con tu asistente de IA:** pegale cada paso y pedile «explicame cada renglón». Cada renglón lo tenés que poder explicar.

El hilo es como la clave de un cliente en una tabla de ventas: cada mensaje se guarda con su clave, y el modelo ve solo los de la misma (un `WHERE hilo = 'A'`).

## Arranque

1. **El mismo RAG, con LangChain.** `create_agent` arma el ciclo «mensajes → modelo → respuesta»; `con_contexto` corre antes de cada llamada y suma lo que trajo tu `buscar`; `InMemorySaver` guarda cada hilo.

    ```python
    import time, uuid
    from langchain.agents import create_agent
    from langchain.agents.middleware import ModelRequest, dynamic_prompt
    from langgraph.checkpoint.memory import InMemorySaver
    from llm import cargar_modelo
    from v02_rag import SISTEMA, buscar  # tu búsqueda de M4, sin cambios
    fuentes = []  # los documentos de la última búsqueda
    @dynamic_prompt
    def con_contexto(pedido: ModelRequest) -> str:  # corre antes de cada llamada al modelo
       hits = buscar(pedido.messages[-1].text)  # busca solo con el último mensaje
       fuentes.clear()  # las de la pregunta anterior, fuera
       fuentes.extend(dict.fromkeys(h["doc"] for h in hits))  # cada documento una vez, como un DISTINCT
       contexto = "\n\n".join(f"[{h['doc']}]\n{h['texto']}" for h in hits)
       return f"{SISTEMA}\n\nContexto:\n{contexto}"
    asistente = create_agent(cargar_modelo(), tools=[], middleware=[con_contexto], checkpointer=InMemorySaver())
    ```

2. **Responder dentro de un hilo**, y una demo que solo corre desde la terminal (no cuando `medir.py` importa el archivo):

    ```python
    def responder(pregunta: str, hilo: str = "") -> dict:  # sin hilo, una conversación nueva
       inicio, config = time.perf_counter(), {"configurable": {"thread_id": hilo or uuid.uuid4().hex}}
       ultimo = asistente.invoke({"messages": [("user", pregunta)]}, config)["messages"][-1]
       return {"respuesta": ultimo.text, "fuentes": list(fuentes), "latencia_s": round(time.perf_counter() - inicio, 2),
               "tokens_entrada": ultimo.usage_metadata["input_tokens"], "tokens_salida": ultimo.usage_metadata["output_tokens"]}
    def mensajes_guardados(hilo):  # cuántos mensajes guardó ese hilo
       return len(asistente.get_state({"configurable": {"thread_id": hilo}}).values.get("messages", []))
    if __name__ == "__main__":
       print(responder("¿Qué guarda un checkpointer en LangGraph?", "A")["respuesta"])
       print(responder("¿Y qué pasa con eso si se reinicia el proceso?", "A")["respuesta"])
       print(responder("¿Y qué pasa con eso si se reinicia el proceso?", "B")["respuesta"])
       print("hilo A:", mensajes_guardados("A"), "· hilo B:", mensajes_guardados("B"))
    ```

    Corré `uv run python v03_memoria.py`. **Qué deberías ver:** tres respuestas y `hilo A: 4 · hilo B: 2`. Las respuestas varían; la prueba de memoria es la cuenta (cada turno guarda tu pregunta y la respuesta).

**Primer control · 10–15 min:** «hilo A: 4 · hilo B: 2».

## En clase

1. **Predecí:** de v0.2 a v0.3, ¿el hit cambia?
2. **Medí:** en `medir.py`, cambiá `v02_rag` por `v03_memoria` (las dos veces) y corré `uv run python medir.py`, con tu suite. **Qué deberías ver:** el mismo hit que en M4 (con la suite de ejemplo, 10 de 11): la búsqueda es la misma.
3. **Cada diferencia:** buscá en las dos tablas las preguntas con otra respuesta. El contexto ahora llega en las instrucciones, no junto a la pregunta: ¿eso cambió algo?
4. **Tu conversación:** en la demo, cambiá las preguntas por una tuya y un seguimiento que la **necesite** (en `"B"`, el mismo seguimiento).

    ![El final de v03_memoria.py, abreviado: tu pregunta y un seguimiento que la necesite en el hilo A, el mismo seguimiento en el hilo B y la prueba de que B no guarda lo de A](../assets/conversacion-hilos.svg)

5. **Aislamiento:** sumá al final de la demo, con 4 espacios de sangría, y corré:

    ```python
       print(responder("otra pregunta de tu tema", "B")["respuesta"])
       print("hilo A:", mensajes_guardados("A"), "· hilo B:", mensajes_guardados("B"))
    ```

    **Qué deberías ver:** al final, `hilo A: 4 · hilo B: 4`. B creció; A no se enteró.

6. *Commit* («v0.3: …») y *Push*.

## Controles

- Misma suite, modelo y perillas que en M4.
- Pregunta y seguimiento en el mismo hilo.
- B no ve nada de A.

## Resultado probado

Tu test de hilos: la demo con tus preguntas y el paso 5.

| Versión | Hit | NO ENCONTRADO bien | Correctas (tu juicio) | Latencia (mediana) |
|---|---|---|---|---|
| v0.2 | | | | |
| v0.3 | | | | |

## Evidencia

En `evidencia/v0.3.md`, entrega de avance 3 (hasta el mar 17/11, 23:59): tabla, test de hilos, diferencias y una **nota de decisión** de 3 líneas.

## Para casa

- El seguimiento busca solo con el último mensaje: mirá sus `fuentes`. ¿Trajo lo que hacía falta?
- Alargá una conversación hasta 10 turnos o hasta que moleste: ¿cuántos aguantó?
- ¿Últimos mensajes (`trim_messages`) o resumen (`SummarizationMiddleware`)? Decidí y justificá, sin programarlo.

## Microdefensa

- ¿Qué ya no podrías explicar sin leer el código de LangChain?
- Confianza del 1 al 4.
