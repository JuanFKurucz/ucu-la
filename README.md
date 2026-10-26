# LLMs en Acción — actividades

Sitio de actividades de la materia LLMs en Acción (Maestría en Ciencia de Datos, UCU): las prácticas y sus cuadernos de Colab. Se genera desde el material del curso con `exportar_ucu_la.py`; no editar a mano.

- `docs/`: una página por práctica y las slides.
- `notebooks/`: los cuadernos (botón **Open in Colab** en cada práctica).

## Ejecutar localmente

```bash
pip install -r requirements.txt
mkdocs serve
```

## Despliegue

Cada `push` a `main` ejecuta el build con `--strict` y publica en GitHub Pages: https://juanfkurucz.com/ucu-la/
