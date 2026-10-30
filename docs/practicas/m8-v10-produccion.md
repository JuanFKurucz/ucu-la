# M8 · v1.0 ¿Lo liberarías?

| Módulo | Clase | En clase | IA permitida |
|---|---|---|---|
| M8 | 7 | 25 min | como co-diseñadora: propone, vos decidís (A3) |

**Entregás:** v1.0 en el repo, extracto de la traza en la evidencia, costo por consulta y por mes, test del guardrail, decisión go/no-go.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/JuanFKurucz/ucu-la/blob/main/notebooks/m8_v10_produccion.ipynb) Abrir el cuaderno en Colab (para quien no tiene el entorno instalado).


## Objetivo

- Evidencia para decidir si gente real podría usarlo: **traza**, **costo**, **protección con test** y una práctica de producción.
- Es la **v1.0** (requisito 4).

## Punto de partida

- **Tu repo**, con `v05_equipo.py` (o `v04_agente.py`, si en M7 ganó el agente solo) y `medir.py`. Creás `v10_produccion.py` y `tests/test_produccion.py`.
- **El contrato:** lo de v0.5 más `costo_usd`, y una línea de traza por pregunta en `logs/`.
- **¿Sin el entorno instalado?** Lo mismo en Colab: el botón «Open in Colab» de arriba.
- **Con tu asistente de IA:** acá podés pedirle más (nivel A3), pero cada renglón lo tenés que poder explicar.

v1.0 es tu versión de siempre con lo que pide un tablero que usa gente real: un registro de cada consulta (la traza, como el log de actualizaciones de Power BI), su costo, y que un dato personal no se escape.

## Arranque

1. **Tu v1.0:** envuelve tu última versión; tapa los emails, calcula el costo, deja la traza y, si algo falla, avisa en vez de inventar.

    ```python
    import datetime, json, os, re
    import v05_equipo as base  # o v04_agente, si en M7 ganó el agente solo
    PRECIO_ENTRADA, PRECIO_SALIDA, PRECIO_FECHA = 0.30, 2.50, "19/10"  # US$ por millón de tokens, de la página del proveedor
    def tapar(texto: str) -> str:  # el guardrail: un email no llega al modelo ni a la traza
       return re.sub(r"[\w.+-]+@[\w-]+(\.[\w-]+)+", "[email]", texto)
    def responder(pregunta: str) -> dict:
       try:
           r = base.responder(tapar(pregunta))
       except Exception as error:  # fallo seguro: un aviso claro, nunca una respuesta inventada
           r = {"respuesta": "No pude responder ahora. Probá en unos minutos.", "error": type(error).__name__,
                "tokens_entrada": 0, "tokens_salida": 0}
       r["costo_usd"] = (r["tokens_entrada"] * PRECIO_ENTRADA + r["tokens_salida"] * PRECIO_SALIDA) / 1_000_000
       os.makedirs("logs", exist_ok=True)
       with open("logs/traza.jsonl", "a", encoding="utf-8") as traza:  # una línea por pregunta
           fila = {"cuando": datetime.datetime.now().isoformat(timespec="seconds"), "pregunta": tapar(pregunta), "precio_del": PRECIO_FECHA, **r}
           traza.write(json.dumps(fila, ensure_ascii=False) + "\n")
       return r
    if __name__ == "__main__": print(responder("¿Qué hace interrupt()? Respondeme a ana.perez@ucu.edu.uy"))
    ```

    Corré `uv run python v10_produccion.py`. **Qué deberías ver:** la respuesta con `costo_usd`. En `logs/traza.jsonl`, la última línea: la pregunta con `[email]`, las herramientas, los tokens y el costo.

**Primer control · 10–15 min:** en la traza, ningún email y el costo de la consulta.

## En clase

1. **El guardrail, con test.** Guardá esto como `tests/test_produccion.py` y corré `uv run pytest` (no gasta: el segundo simula la caída):

    ```python
    import v10_produccion
    def test_un_email_no_pasa():
       assert v10_produccion.tapar("soy ana.perez@ucu.edu.uy, ¿me ayudás?") == "soy [email], ¿me ayudás?"
    def test_si_falla_avisa(monkeypatch):
       monkeypatch.setattr(v10_produccion.base, "responder", lambda pregunta: 1 / 0)  # simula una caída
       assert "No pude responder" in v10_produccion.responder("hola")["respuesta"]
    ```

    **Qué deberías ver:** todos `passed`, ningún `failed`.

2. **Tu peor fallo:** un test más, con un caso tuyo (otro dato personal, una pregunta que lo hizo inventar). ¿`failed`? Anotá qué no atrapa el guardrail.
3. **Operá y costeá**, con `web = "grabada"`: en `medir.py`, cambiá tu versión por `v10_produccion` y sumá al final:

    ```python
    print("US$ por consulta:", tabla["costo_usd"].mean(), "· por mes:", tabla["costo_usd"].mean() * 1000 * 30)
    tabla.to_csv("logs/medir.csv", index=False)  # para la tabla dinámica de tus fallas, en Excel o Power BI
    ```

    Es un mes a 1000 por día. ¿Cuánto sería con tu uso real?

4. **Tu práctica de producción** (una): trazas, costos, guardrail, caché o evaluación continua.
    Caché en tres líneas, arriba de `v10_produccion.py`: `from langchain_core.caches import InMemoryCache`, `from langchain_core.globals import set_llm_cache` y `set_llm_cache(InMemoryCache())`.
    Vive mientras corre el programa: repetí una pregunta y medí `latencia_s`.

5. **Go / no-go** en media página: «**No-go por ahora.** Suite: x/n. Costo: US$ … por consulta y por mes (precio del dd/mm). Test: pasa. Sigue fallando: … Probaría: …».
6. *Commit* («v1.0: …») y *Push*.

## Controles

- La medición no empeora (o explicás por qué).
- `uv run pytest` sin `failed`.
- La traza de la evidencia no tiene claves ni datos personales.

## Resultado probado

`v1.0` en el repo y el go / no-go del paso 5.

## Evidencia

En `evidencia/v1.0.md`, después al registro de evaluación (`docs/registro-evaluacion.md` de tu repo): traza, costo con fecha, test y decisión.

## Para casa

- La **lista de entrega** de la guía 8 (también en la tarea «Entrega final» del aula virtual), y «Antes de entregar» del README de tu repo.
- Los 4 entregables hasta el **domingo 22/11, 23:59**.

## Microdefensa

- ¿Qué te impide liberarlo hoy? ¿Qué prueba lo resolvería?
- Confianza del 1 al 4.
