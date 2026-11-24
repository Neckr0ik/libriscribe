# Análisis rápido de LibriScribe (fork Neckr0ik) — 24-nov-2026

Diagnóstico, sin cambios al código. Pedido de Giovanni.

## Qué es
Generador de libros con LLM en CLI (Typer) y agentes por tarea: concepto, esquema, personajes, mundo, capítulos, edición, estilo, revisión, verificación de datos, plagio, investigación y formato (Markdown/PDF). Fork MIT de `guerra2fernando/libriscribe`. Proveedores: OpenAI, Anthropic, Gemini, DeepSeek, Mistral y OpenRouter (este último lo añadió Tom McFarland en 2025). Unas 4,400 líneas de Python.

**Atribución:** `LICENSE` ya lleva el copyright de Fernando Guerra (2024) y el de Giovanni (2025). Si se copia código a otro repo hay que conservar ese aviso.

## Estado
- 32 commits: Guerra 21, McFarland 10, Giovanni 1.
- Último commit upstream: 23-oct-2025. Último del fork: 28-may-2026 (solo `LICENSE`). Sin actividad hace ~6 meses.
- Sin carpeta de pruebas (CONTRIBUTING las pide). El roadmap del README está casi todo sin hacer (vector stores, API, frontend, auth).

## Salud (verificado en un venv limpio, Python 3.13)
| Prueba | Resultado |
|---|---|
| `pip install -e .` e `import libriscribe` | Funciona |
| `import libriscribe.main` solo con `setup.py` | **Falla**: `ModuleNotFoundError: anthropic` |
| Tras `pip install -r requirements.txt` | Importa y `libriscribe --help` lista los comandos |
| `pip-audit` sobre las dependencias | Sin vulnerabilidades conocidas |
| `pip list --outdated` | Solo paquetes de apoyo de Google (protobuf 5→7, grpc, api-core), nada crítico |

Hallazgos concretos:
1. **`setup.py` incompleto.** `install_requires` omite `anthropic`, `google-generativeai`, `rich` y `pick`, que sí se importan. La instalación que indica el README (`pip install -e .`) deja un CLI roto.
2. **PyYAML no declarado.** `utils/prompt_loader.py` hace `import yaml` y no está en `setup.py` ni en `requirements.txt`. Falla con `ModuleNotFoundError` al usar `editor_enhanced` / plantillas YAML externas (comprobado con `import libriscribe.utils.prompt_integration`).
3. **`google-generativeai` descontinuado.** Muestra aviso de fin de soporte; hay que migrar a `google-genai`.
4. **`fpdf` 1.7.2** es la rama antigua; la mantenida es `fpdf2`.
5. Dependencias sin versiones fijadas y `requests` duplicado en `requirements.txt`. README dice Python 3.8+, sin `python_requires`.
6. El workflow de docs usa `actions/checkout@v3` y `setup-node@v3` (antiguos) con Node 18.

## ¿Aporta frente a impulse-book-factory?
Limitación: este análisis solo ve el repo `libriscribe`; **no se inspeccionó `impulse-book-factory`**. La comparación es por lo que consta de ese proyecto (repo activo de producción de libros), no por lectura de su código.

Lo que LibriScribe tiene y vale la pena revisar si no está ya en la fábrica:
- Cadena de agentes de control de calidad (revisor de contenido, verificador de datos, plagio).
- Plantillas de prompts en YAML externas (`prompts/templates/`), fáciles de versionar.
- Selección de proveedor con OpenRouter.

Lo que no aporta: arquitectura de producto, pruebas, ni mantenimiento.

## Recomendación (decide Giovanni)
**Fusionar ideas y congelar (archivar con etiqueta), no mantener como producto aparte.**
- Mantenerlo vivo cuesta: hay que parchar dependencias, migrar Gemini y escribir pruebas, y el upstream está parado.
- Archivarlo sin más pierde las plantillas y la cadena de QA.
- Camino intermedio, riesgo bajo: etiquetar la versión con la que se produjeron libros ya publicados (para poder reproducirlos), extraer las ideas útiles a la fábrica citando la licencia MIT, y archivar el fork.
- Si se quiere reproducir un libro antes de archivar, bastan los arreglos 1 y 2 (dos líneas en dependencias).

## 3 siguientes pasos
1. Comparar con `impulse-book-factory` (leer su código): ¿ya tiene QA de datos, plagio y prompts externos? Eso decide si hay algo que fusionar.
2. Si Giovanni elige congelar: arreglar `setup.py` + PyYAML (cambio mínimo), etiquetar la versión y archivar.
3. Si elige mantener: migrar a `google-genai` y `fpdf2`, fijar versiones, añadir pruebas de humo del CLI y actualizar el workflow de docs.

## Nota: Nemotron frente a la verificación
- **Nemotron no respondió.** La herramienta `opencode_run` está en línea (`opencode_status` OK) pero falló dos veces con `Unrecognized flag: --dir in command opencode run` (error del conector, no del modelo). No hubo primer avance de Nemotron, por lo que no hay aciertos ni errores que anotar. Conviene reportar el fallo al dueño del conector.
- Todo lo de este informe sale de verificación propia: lectura del repo, instalación en venv, importación, `pip-audit`, `pip list --outdated`.

## Conectores posibles (no activados)
- **OpenRouter** (o 1min.AI): si se mantiene el proyecto, una sola clave da acceso a varios modelos; el fork ya soporta OpenRouter.
- **Google Drive**: guardar manuscritos e informes fuera del repo.
