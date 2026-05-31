# Objetivo

Modificar el pipeline y propuesta dcon respecto a la unidad 1 teniendo encuenta que durante las ultimas clase se vieron nuevas herramientas al igual que las nuevas investigaciones de un clasificador orientativo del **estado clínico simulado** en cuatro categorías, en el marco de la materia entrega unidad 3**MLOps (MIAA — ICESI)**.

## Contexto del problema

El dominio médico genera mucha información, pero en problemas reales suele haber **asimetría en los datos**:

- **Condiciones frecuentes / “comunes”:** abundancia de ejemplos (p. ej. `ENFERMEDAD LEVE`).
- **Condiciones minoritarias / “huérfanas” en sentido académico:** pocos ejemplos (p. ej. `ENFERMEDAD AGUDA` ≈ **1,9 %** del dataset) o patologías en el CSV de entrenamiento.


## Requerimiento del proyecto

Construir un **modelo predictivo** (evolución: de reglas MVP a **LightGBM** en producción propuesta) que estime el estado clínico simulado a partir de signos vitales y hábitos, con un **pipeline MLOps** reproducible, desplegable y monitoreado.

## Objetivos del modelo

- **Clases frecuentes:** clasificar con estabilidad cuando hay muchos datos de entrenamiento.
- **Clase minoritaria (AGUDA):** no sacrificar **recall** por accuracy global engañosa.

---

## Diseño del pipeline

Para arquitectura técnica, **stack por etapa**, **suposiciones**, **justificaciones** y **diagrama general**, consulta:


| Bloque | Contenido resumido |
| :--- | :--- |
| **Data Pipeline** | DVC, EDA, Great Expectations, features Feast/sklearn |
| **Develop** | Bandit/Trivy, pytest, LightGBM+Optuna, MLflow, Api Deploy Dev |
| **Staging** | Build, MLflow Registry, Api Deploy ST, QA Medical simulado |
| **PROD** | Cloud Run + edge, monitoreo Prometheus/Grafana/Evidently, re-entrenamiento |

### Diagrama general (Unidad 3)

![Diagrama del pipeline MLOps](PipeLineML.png)


### CHANGELOG (Semana 1 → Unidad 3)

Evolución de la propuesta respecto a la rama `main` (PDF Semana 1 + MVP):

**[`CHANGELOG.md`](CHANGELOG.md)**

---

## Estructura del repositorio

| Ruta | Descripción |
| :--- | :--- |
| [`PipeLineML.png`](PipeLineML.png) | **Diagrama general** del pipeline (raíz) |
| [`docs/PROPUESTAPipeLine.md`](docs/PROPUESTAPipeLine.md) | **Propuesta Unidad 3** — pipeline detallado |
| [`CHANGELOG.md`](CHANGELOG.md) | Cambios vs propuesta Semana 1 (`main`) |
| [`data/`](data/) | CSV raw y procesado (~70 k filas) |
| [`servicio_estado_clinico/`](servicio_estado_clinico/) | **MVP implementado** — Flask + reglas + Docker |
| [`scripts/`](scripts/) | Preparación de datos |

**Rama de entrega Unidad 3:** `main`  
**Repositorio:** [github.com/WillianReinaG/entrega3MLops2026](https://github.com/WillianReinaG/entrega3MLops2026)

---

## Path: `/servicio_estado_clinico`

### Servicio de estado clínico simulado (Flask)

API mínima que clasifica en cuatro estados a partir de signos vitales. Expone:

- **`POST /predecir`** — JSON → etiqueta clínica simulada  
- **`/`** — formulario web de prueba  

La lógica actual es **determinista** (`modelo_simulado.py`); la propuesta Unidad 3 define promoción a **LightGBM** solo si supera el baseline en MLflow.

Documentación de uso (Docker, curl, campos JSON): [`servicio_estado_clinico/README.md`](servicio_estado_clinico/README.md)

### Ejecución rápida con Docker

```powershell
cd servicio_estado_clinico
docker build -t estado-clinico-demo .
docker run --rm -p 5000:5000 estado-clinico-demo
```

Abrir `http://localhost:5000/`.

---

## Stack resumido (propuesta Unidad 3)

| Categoría | Herramientas |
| :--- | :--- |
| Lenguaje / VCS | Python, GitHub |
| CI/CD | GitHub Actions |
| Contenedores | Docker, GHCR |
| Datos | DVC, MinIO/S3, Great Expectations |
| ML | LightGBM, Optuna, MLflow, SHAP |
| Cloud | Google Cloud Run |
| Monitoreo | Prometheus, Grafana, Evidently |
| Seguridad | Bandit, Trivy |

*(Comparación detallada con Semana 1 y con el ejemplo `entrega3`: ver [`CHANGELOG.md`](CHANGELOG.md).)*

---

## Aviso

Trabajo este material coresponde a material academico del trabajo para la entrega de la unidad 3 que solisito el profesor David Avila CRUZ curso de MLops

---

Willian Reina G. — MIAA

