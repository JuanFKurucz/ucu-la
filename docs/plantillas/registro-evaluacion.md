# Registro de evaluación — [nombre del sistema]

Guardalo en tu repo como `docs/registro-evaluacion.md` (en Markdown); al aula virtual va el enlace del repo.

## Suite

Cantidad de consultas por `tipo`, escrito como en tus suites (M4: `documental`,
`sintesis`, `sin_respuesta`, `adversarial`; M6: `corpus`, `web`, `calculo`,
`ninguna`, `combinada` y, del «Para casa», `ambigua`). Al empezar, `suite.py` los cuenta
(con la de ejemplo: `13 preguntas · por tipo: documental 9 · sintesis 1 · sin_respuesta 2 · adversarial 1`). Enlace al archivo de preguntas (`.jsonl`) y al comando que lo corre.

Para la entrega: 20 consultas propias (documentales, de síntesis, sin respuesta, con web,
de cálculo y 2 adversariales), cada una con su respuesta esperada, herramienta y fuente
(proyecto integrador, «Suite de evaluación»).
Ojo con las sin respuesta (`"fuentes": []`): la suite cuenta como abstención solo una respuesta que dice
«NO ENCONTRADO». El `SISTEMA` de v0.2 y v0.3 lo pide; el de v0.4 a v1.0, no: con el agente, sumale esa
regla a tu `SISTEMA` (y medí de nuevo), o juzgá esas preguntas en `correcta` y no reportes la abstención.

## Resultados por versión

| Comparación | Métrica | Antes | Después | Condiciones fijas |
|---|---|---:|---:|---|
| *Ejemplo: fragmento/overlap 1000/150 → 400/50* | *hit* | *10/11* | *11/11* | *mismas 13 preguntas, mismo modelo* |
| Sin RAG / con RAG | respuestas correctas | | | corpus, preguntas, modelo |
| Recuperación inicial / modificada | hit | | | misma suite y modelo; cambia una sola perilla |
| Antes / después de pasar a LangChain | correctas | | | misma suite, modelo y perillas |
| Agente: selección de herramienta | selección (x/n, como la da `suite.py`) | | | misma suite de selección y modo de web |
| Único / multiagente | correctas · llamadas por consulta (más llamadas = más costo) · latencia mediana (las dos las da `suite.py`) | | | misma suite y modo de web |

«Correctas» no la da `suite.py`: contá los «sí» de tu columna `correcta`. En Excel en castellano, por ejemplo
`=CONTAR.SI(M2:M14; "s*")`: el `*` cuenta «sí», «si» y «Sí». `correcta` es siempre la última columna: la M en los CSV
de tu suite de M4 y la L en los de selección (M6 en adelante). Cambiá el 14 por tu último renglón. Escribila como `8/13`.
Antes de copiar una corrida, mirá si `suite.py` avisó «Ojo: N de M preguntas dieron ERROR» (cuota, red, proveedor saturado o, con un 404, el nombre del modelo):
esas filas cuentan como fallas o aciertos sin que el sistema haya respondido. Repetila y copiá la limpia.

**Consistencia (`pass^3`):** la corrida final va 3 veces, con la misma suite (`--tag final-1`, `final-2` y `final-3`).
Anotá con qué `web` la corriste: la evidencia final va con la integración real, o con la grabada si lo declarás
(proyecto integrador, «Restricciones técnicas»).
Reportá cuántas preguntas salen bien **en las 3**, no solo el promedio: un LLM acierta una vez y erra la siguiente
(proyecto integrador). Con v1.0, unos 2,7 pedidos por pregunta: con 20, unos 55 por corrida (165 las tres).
Para contarlas, completá `correcta` en los tres CSV, guardá esto como `pass3.py` en la carpeta del proyecto
y corré `uv run python pass3.py`:

```python
import pandas as pd
leer = dict(sep=None, engine="python", encoding="utf-8-sig", encoding_errors="replace", keep_default_na=False)  # «;» y ANSI de Excel
corridas = [pd.read_csv(f"logs/suite-final-{i}.csv", **leer) for i in (1, 2, 3)]
bien = [set(c["id"][c["correcta"].astype(str).str.strip().str.lower().str.startswith("s")]) for c in corridas]
print(f"pass^3: {len(bien[0] & bien[1] & bien[2])} de {len(corridas[0])} salen bien en las 3")
```

## Ejemplos (mínimo 5)

Salen de los CSV de tus suites en `logs/`: columnas `pregunta`, `respuesta`, `fuentes` o
`herramientas` (al lado, `fuentes_esperadas` o `herramientas_esperadas`: si no coinciden, es
un fallo de recuperación o de selección) y tu `correcta`. Elegí aciertos y fallos (los fallos van también abajo).
Copiá los textos en la tabla: `logs/` no se sube al repositorio (está en `.gitignore`), así que
quien lea tu registro no ve esos CSV.

| Consulta | Respuesta del sistema | Fuente o herramienta | ¿Correcta? | Análisis |
|---|---|---|---|---|
| *¿Qué recargo tiene una cuota pagada el día 25?* | *«2 %, venció el día 10 [reglamento]»* | *buscar_documentos → reglamento* | *Sí* | *El fragmento correcto llegó primero.* |

## Fallos representativos

Al menos 2, clasificados con su causa probable:

- **recuperación**: el fragmento con la respuesta no llegó (M4);
- **generación**: llegó y el modelo lo ignoró o lo leyó mal (M4);
- **selección de herramienta**: pidió otra herramienta o ninguna (M6);
- **coordinación**: en el equipo, el supervisor repartió mal o una nota no llegó al redactor (M7).

Si hiciste el análisis de errores de la clase 7, usá tus etiquetas de `primer_error`.

## Costo y operación

Tokens y costo por consulta (el promedio de 3 preguntas de tu suite, M8 paso 2), proyección
mensual, práctica de producción y su evidencia. El costo, con los precios reales de tu modelo y su fecha
(`PRECIO_FECHA`), no con los de ejemplo de `v10_produccion.py`: con Gemini 3.5 Flash-Lite dan unas 4 veces menos
(guía 7, sección 5).
Si `v10_produccion.py` avisó que el proveedor no informó tokens, decilo: ese costo no se pudo medir.
