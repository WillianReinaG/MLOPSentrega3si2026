# Changelog

Registro de cambios entre la **propuesta original (Semana 1 / rama `main`)** y la **propuesta Unidad 3 (rama `unidad3`)**.


---

## Versión 2.0.0 — Unidad 3 (2026-05-29)

Esta versión transforma la idea inicial del Punto 1 en un **pipeline MLOps documentado de punta a punta**, con stack tecnológico argumentado, suposiciones explícitas y diagrama general alineado al ejemplo del profesor, adaptado al caso **estado clínico simulado (cuatro categorías)**.

### Nuevas características / arquitectura

* **Documento principal de pipeline:** [`docs/PROPUESTAPipeLine.md`](docs/PROPUESTAPipeLine.md) — estructura **Data Pipeline → Develop → Staging → PROD + Monitoreo**, igual que el ejemplo `entrega3`, con texto detallado por etapa.
* **Stack de soluciones definido y justificado:**

  | Categoría | Semana 1 (`main`) | Unidad 3 (`unidad3`) |
  | :--- | :--- | :--- |
  | CI/CD | No especificado | **GitHub Actions** (en el ejemplo: Jenkins) |
  | Seguridad en pipeline | No especificado | **Bandit + Trivy** (en el ejemplo: SonarQube) |
  | Datos / almacenamiento | CSV en carpeta | **DVC + MinIO/S3** (en el ejemplo: PostgreSQL) |
  | Modelo producción | Reglas MVP | **LightGBM + Optuna** vs baseline reglas |
  | Registro de modelos | No especificado | **MLflow Model Registry** |
  | Contenedores | Docker local | **Docker + GHCR** |
  | Cloud | No especificado | **Google Cloud Run** |
  | Monitoreo ops | No especificado | **Prometheus + Grafana** |
  | Monitoreo ML | No especificado | **Evidently AI** (drift) |
  | Explicabilidad | Implícita en reglas | **SHAP** documentado |
  | Gobernanza | No existía | Suposiciones S1–S10 y restricciones en `PROPUESTAPipeLine.md` |

* **Diagrama general:** [`PipeLineML.png`](PipeLineML.png) en la raíz (visible en README y `PROPUESTAPipeLine.md`); misma topología que el ejemplo `entrega3`.
* **Suposiciones:** tabla global S1–S10 + suposiciones **por etapa** (Data, Develop, Staging, PROD).
* **Roles QA formalizados:** QA en Develop (`Api Deploy Dev`) y **QA Medical System simulado** en Staging, como en el ejemplo de referencia.
* **Enfermedades huérfanas y desbalance:** política explícita para AGUDA (~1,9 %) y patologías no presentes en CSV.
* **README raíz:** explica objetivo, pipeline, estructura del repo y enlace al CHANGELOG.

### Modificaciones respecto a etapas de la propuesta Semana 1

| Etapa / tema | Semana 1 | Unidad 3 |
| :--- | :--- | :--- |
| Alcance documental | PDF en rama `main` | Markdown + [`PipeLineML.png`](PipeLineML.png) |
| Data Pipeline | Mención genérica a datos | DVC, EDA, GX, Feast, gates detallados |
| Develop | Inferencia con reglas | Unit tests, CI seguridad, entrenamiento MLflow |
| Staging | No existía | Registry MLflow + Api Deploy ST + QA medical |
| PROD | Docker local en MVP | Edge + Cloud Run + aprobación deploy |
| Monitoreo | No existía | Performance + logs + drift + re-entrenamiento |
| Métricas | No definidas | F1 macro, recall AGUDA, SLO latencia |
| Ciclo cerrado | No | Drift → re-entrenamiento → Data Pipeline |

### Documentación añadida (entrega Unidad 3)

* [`PipeLineML.png`](PipeLineML.png) — diagrama general en la raíz  
* [`docs/PROPUESTAPipeLine.md`](docs/PROPUESTAPipeLine.md) — propuesta detallada Unidad 3  
* [`CHANGELOG.md`](CHANGELOG.md) — este archivo (comparación Semana 1 → Unidad 3)  
* [`README.md`](README.md) — índice del repositorio para evaluación  


El servicio MVP (`servicio_estado_clinico/`) luego de ver en clase mayor informacion sobre sistema de MLops se realiza propuesta mejorada de la unida 1, ahora se documenta la evolución propuesta.

---

## Versión 1.0.0 — Semana 1 / rama `main` (estado inicial)

Propuesta inicial del Punto 1 y MVP funcional.

### Contenido en `main`

* [Propuesta Semana 1 (PDF en rama `main`)](https://github.com/WillianReinaG/2MLops_unidad1/blob/main/docs/punto%201%20descripcion%20pipeline%20MLops.pdf) — descripción inicial del pipeline.
* [`servicio_estado_clinico/`](servicio_estado_clinico/) — API Flask + reglas en `modelo_simulado.py` + Dockerfile.
* [`data/`](data/) — CSV raw y procesado (~70 000 filas, cuatro categorías).
* [`scripts/ajustar_cuatro_categorias.py`](scripts/ajustar_cuatro_categorias.py) — preparación de datos.

### Limitaciones de la versión 1.0.0 (motivo del CHANGELOG)

* Sin **CHANGELOG** ni README raíz.
* Sin tabla de **tecnologías por etapa** ni justificación comparativa de herramientas.
* Sin etapas explícitas **Develop / Staging / PROD** ni monitoreo con drift.
* Sin registro de **suposiciones** ni gates de promoción de modelo.
* Despliegue documentado solo como **Docker manual** en README del servicio.

---

## Cómo leer este archivo

1. **Versión 1.0.0** = lo que existía en Semana 1 (`main`).  
2. **Versión 2.0.0** = lo entregado en Unidad 3 (`unidad3`).

