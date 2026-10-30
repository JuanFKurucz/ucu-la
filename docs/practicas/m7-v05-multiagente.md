# M7 · v0.5 Equipo de agentes con aprobación humana

| Módulo | Clase | En clase | IA permitida |
|---|---|---|---|
| M7 | 6 | 110 min | como copiloto: escribe con vos y vos lo explicás (A2) |

**Entregás:** v0.5 en el repo, diagrama del grafo, prueba de rechazo, tabla agente único vs equipo.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/JuanFKurucz/ucu-la/blob/main/notebooks/m7_v05_equipo.ipynb) Abrir el cuaderno en Colab (para quien no tiene el entorno instalado).


## Objetivo

- Tu asistente como **equipo** (supervisor, investigador, analista, redactor), con **una persona que aprueba**.
- Medir si el equipo vale lo que cuesta frente a `v0.4`. Es la **v0.5** (requisito 3).

## Punto de partida

- **Tu repo**, con `v04_agente.py` y `medir.py` de M6. Creás `v05_equipo.py`.
- **El contrato:** `responder(pregunta)` devuelve lo de v0.4 más `aprobada`: si la persona rechaza, no hay respuesta.
- **¿Sin el entorno instalado?** Lo mismo en Colab: el botón «Open in Colab» de arriba.
- **Con tu asistente de IA:** pegale cada paso y pedile «explicame cada renglón».

El grafo es un flujo de Power Automate escrito en código: cada `add_node` es una caja y cada `add_edge`, una flecha. El **estado** es una fila que viaja de caja en caja; cada caja completa sus columnas.

## Arranque

1. **El estado, los especialistas y el supervisor.** Cada especialista es un agente como el de M6, con menos herramientas:

    ```python
    import operator, uuid
    from typing import Annotated, TypedDict
    from langchain.agents import create_agent
    from langgraph.checkpoint.memory import InMemorySaver
    from langgraph.graph import END, START, StateGraph
    from langgraph.types import Command, interrupt
    from herramientas import Medidor, buscar_web, calcular
    from v04_agente import buscar_documentos, modelo  # tu v0.4
    Estado = TypedDict("Estado", {"pregunta": str, "notas": Annotated[list, operator.add], "siguiente": str, "borrador": str, "aprobada": bool})
    def especialista(herramientas, consigna):  # un agente como caja: lee la pregunta y las notas, y SUMA su nota
       agente = create_agent(modelo, herramientas, system_prompt=consigna)
       return lambda e: {"notas": [agente.invoke({"messages": [("user", f"{e['pregunta']}\nNotas: {e['notas']}")]})["messages"][-1].text]}
    def supervisor(e):  # el modelo elige quién sigue; con 3 notas, al redactor (el tope va en código)
       r = modelo.invoke(f"{e['pregunta']}\nNotas: {e['notas']}\n¿Quién sigue? Una palabra: investigador, analista o redactor.").text.lower()
       return {"siguiente": next((n for n in ("investigador", "analista") if n in r and len(e["notas"]) < 3), "redactor")}
    ```

    Dos renglones para leer despacio:

    - `lambda e: {…}` es una función de una línea: recibe el estado `e` y devuelve la nota del agente.
    - `next((n for n in ("investigador", "analista") if …), "redactor")` es un `CASE WHEN` de SQL: si la respuesta nombra a uno de los dos (y hay menos de 3 notas), va ese; si no, `ELSE 'redactor'`.

2. **El redactor, la aprobación y el grafo.** `interrupt` pausa el grafo; el *checkpointer* guarda el estado mientras espera:

    ```python
    def redactor(e):
       return {"borrador": modelo.invoke(f"Respondé «{e['pregunta']}» en 2 o 3 frases, SOLO con estas notas:\n{e['notas']}").text}
    def aprobar(e):  # PAUSA hasta que una persona decide
       return {"aprobada": interrupt(e["borrador"]) is True}
    grafo = (StateGraph(Estado).add_node(supervisor).add_node(redactor).add_node(aprobar)
            .add_node("investigador", especialista([buscar_documentos, buscar_web], "Buscá y resumí lo hallado, con su fuente."))
            .add_node("analista", especialista([calcular], "Hacé con calcular las cuentas que pida la pregunta."))
            .add_edge(START, "supervisor").add_edge("investigador", "supervisor").add_edge("analista", "supervisor")
            .add_conditional_edges("supervisor", lambda e: e["siguiente"], ["investigador", "analista", "redactor"])
            .add_edge("redactor", "aprobar").add_edge("aprobar", END).compile(checkpointer=InMemorySaver()))
    ```

3. **Responder:** corre hasta la pausa y retoma el **mismo hilo** con la decisión.

    ```python
    def responder(pregunta: str, aprueba: bool = True) -> dict:  # en medir.py, la persona aprueba siempre
       medidor, hilo = Medidor(), {"configurable": {"thread_id": uuid.uuid4().hex}}
       grafo.invoke({"pregunta": pregunta, "notas": []}, {**hilo, "callbacks": [medidor]})  # hasta aprobar: PAUSA
       e = grafo.invoke(Command(resume=aprueba), hilo)  # retoma con la decisión de la persona
       return {"respuesta": e["borrador"] if e["aprobada"] else "", "aprobada": e["aprobada"], **medidor.resumen()}
    ```

4. **La demo, con tu asistente:** pedile «un bloque `if __name__ == "__main__":` que imprima `grafo.get_graph().draw_mermaid()` y responda una pregunta aprobando y la misma rechazando (`aprueba=False`)». Con `web = "grabada"`, corré `uv run python v05_equipo.py`. **Qué deberías ver:** el dibujo (`---`, `config:`, `graph TD;`) y dos respuestas: la rechazada, con `'respuesta': ''`.
5. Pegá el dibujo (del primer `---` a la última `classDef`) en [mermaid.live](https://mermaid.live).

**Primer control · 10–15 min:** el grafo en mermaid.live y las dos respuestas.

## En clase

**Clase 6 · el equipo**

1. **Predecí:** ¿dónde gana el equipo? ¿Cuántas llamadas más?
2. **Tu tema:** en la consigna del investigador, tu tema; en la demo, una pregunta que necesite investigar **y** calcular.
3. **El recorrido:** pedile a tu asistente que `responder` devuelva también las `notas`. ¿Quién actuó, y en qué orden?

**Clase 7 · aprobar y comparar**

4. **Aprobar y rechazar:** con rechazo **no** hay respuesta. ¿Dónde lo ves en el dibujo?
5. **Comparar con v0.4**, misma suite y web: `uv run python medir.py` con `v04_agente`, y otra vez cambiando `v04_agente` por `v05_equipo` (las dos veces). **Qué deberías ver:** la misma tabla; el equipo, con varias veces más `llamadas`.
6. *Commit* («v0.5: …») y *Push*.

## Controles

- Un rechazo no deja respuesta.
- Misma suite, modelo y web en la comparación.
- Ninguna pregunta deja al supervisor dando vueltas: con 3 notas, redacta.

## Resultado probado

| Versión | Eligió bien | Respuestas correctas (tu juicio) | Llamadas por pregunta | Latencia (mediana) |
|---|---|---|---|---|
| v0.4 · agente solo | | | | |
| v0.5 · equipo | | | | |

## Evidencia

En `evidencia/v0.5.md`, después al registro de evaluación (`docs/registro-evaluacion.md` de tu repo): diagrama, aprobar y rechazar, tabla y **tu decisión** (si gana el agente solo, también vale).

## Para casa

- El tope en 1 y en 5 notas (el `3` del supervisor): medí con cada uno. ¿Eligió bien, llamadas, latencia? Al terminar, volvé a 3.

## Microdefensa

- ¿Qué pasa del investigador al redactor, y por qué no toda la conversación?
- Confianza del 1 al 4.
