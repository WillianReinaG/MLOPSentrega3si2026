# MLOPS – MIAA: Diseño del Pipeline para Estado Clínico Simulado

**Estudiante:** Willian Reina G.  
**Repositorio:** [WillianReinaG/entrega3MLops2026](https://github.com/WillianReinaG/entrega3MLops2026) · **Rama:** `main`  
**Versión de propuesta:** 2.0.0 (Unidad 3)

---

## Resumen

Este documento describe el **pipeline MLOps completo** para un servicio de clasificación orientativa en **cuatro estados clínicos simulados** (`NO ENFERMO`, `ENFERMEDAD LEVE`, `ENFERMEDAD AGUDA`, `ENFERMEDAD CRÓNICA`), partiendo del MVP entregado en Semana 1 (reglas + Flask + Docker) hacia un sistema reproducible, desplegable y monitoreado.

La estructura del diagrama del modelo de la unidad 1 de MLOps: **Data Pipeline → Develop → Staging → PROD**, con ciclo de re-entrenamiento. Las **herramientas difieren** en CI/CD, almacenamiento de datos, registro de modelos y detección de drift (ver §1.1).

---

## 1. Diseño del Pipeline y Restricciones

### 1.1. Stack de soluciones y tecnologías

| Categoría | Herramienta elegida | Justificación breve |
| :--- | :--- | :--- |
| **Lenguaje / código** | Python 3.11+ | Ecosistema ML maduro; mismo lenguaje en entrenamiento e inferencia. |
| **Versionamiento** | Git + GitHub | Colaboración, PRs, trazabilidad de cambios en código y workflows. |
| **Automatización CI/CD** | **GitHub Actions** | Integrado al repo; sin servidor Jenkins adicional. Sustituye Jenkins del ejemplo de referencia. |
| **Contenedores** | Docker + **GHCR** | Imagen inmutable probada en CI; publicación en GitHub Container Registry. |
| **Datos (offline)** | CSV + **DVC** + **MinIO/S3** | Versionado de datasets con hash reproducible; no requiere PostgreSQL operacional para entrenamiento batch académico. |
| **Calidad de datos** | **Great Expectations** + Pandera | Gates automáticos antes de entrenar; coherente con validación API del MVP. |
| **Feature store (offline)** | **Feast** (modo offline) + sklearn  | Contrato único train/serve; un solo artefacto serializado. |
| **Entrenamiento / experimentos** | **LightGBM** + **Optuna** + **MLflow Tracking** | Adecuado a datos tabulares; búsqueda de hiperparámetros trazable. |
| **Registro de modelos** | **MLflow Model Registry** | Staging → Production con rollback; sustituye registro genérico del diagrama de referencia. |
| **Infraestructura cloud** | **Google Cloud Run** | Serverless, TLS, escalado; despliegue remoto sin administrar VMs. |
| **Inferencia edge** | Docker Compose en PC del médico | Cumple restricción de recursos locales y privacidad. |
| **Monitoreo operativo** | **Prometheus** + **Grafana** | Latencia, errores 5xx, disponibilidad (como en el ejemplo). |
| **Monitoreo ML (drift)** | **Evidently AI** | Detección de data/concept drift tabular; no presente en el ejemplo original. |
| **Seguridad de código / imagen** | **Bandit** + **Trivy** | Análisis estático Python y CVE en imagen Docker; sustituye SonarQube del ejemplo. |
| **Explicabilidad (XAI)** | **SHAP** | Factores que explican la predicción para auditoría académica. |
| **Documentación de modelo** | Secciones §1.2–1.4 y §5 de este documento | Alcance, limitaciones y suposiciones S1–S10 en la misma propuesta (sin archivos aparte). |

**Decisión explícita vs ejemplo `entrega3`:** herramientas vistas en MLOps moderno (DVC, MLflow, GitHub Actions, Evidently) en lugar de Jenkins, PostgreSQL central y SonarQube.

---

### 1.2. Restricciones y limitaciones

| Restricción | Descripción | Implicación en el diseño |
| :--- | :--- | :--- |
| **Ética y privacidad** | Datos simulados, pero el flujo imita datos clínicos. | Sin PII en logs; anonimización en EDA si hubiera identificadores; TLS en cloud. |
| **No es diagnóstico real** | Propuesta académica; umbrales no validados clínicamente. | Disclaimer en UI, Model Card y respuestas API conservadoras. |
| **Desbalance de clases** | Clase **AGUDA ~1,9 %** en ~70 000 filas. | Estratificación, `class_weight`, recall AGUDA como gate de promoción, SMOTE opcional. |
| **Recursos edge** | Médico con laptop modesta. | Contenedor único, inferencia síncrona, sin Spark batch. |
| **Interpretabilidad (XAI)** | El médico debe entender *por qué* una etiqueta (nivel académico). | SHAP + baseline de reglas interpretable hasta que ML supere métricas con evidencia. |
| **Presupuesto académico** | Free tier cloud. | Cloud Run min-instances=0; almacenamiento S3/MinIO compacto. |

---

### 1.3. Tipos de datos

| Tipo | Origen en este proyecto | Uso en pipeline |
| :--- | :--- | :--- |
| **Estructurados (tabulares)** | `data/raw/enfermedades_cardiacas.csv` → `data/processed/enfermedades_cardiacas_4categorias.csv` | Entrenamiento LightGBM; inferencia vía JSON (`presion_sistolica`, `presion_diastolica`, `nivel_colesterol`, `nivel_glucosa`, `presencia_enfermedad`, `fumador`). |
| **Metadatos de linaje** | Tags DVC, commits Git, runs MLflow | Trazabilidad datos → modelo → despliegue. |
| **Logs de inferencia** | JSON estructurado (sin PII) | Monitoreo, auditoría, detección de drift. |
| **No estructurados** | **Fuera de alcance MVP** | Notas clínicas o imágenes requerirían NLP/CV; se documenta como extensión futura, no en Unidad 3. |

---

### 1.4. Suposiciones globales del proyecto

| ID | Suposición | Si falla | Mitigación |
| :--- | :--- | :--- | :--- |
| S1 | El CSV refleja el dominio simulado acordado | Sesgo sistemático | EDA periódico + drift Evidently |
| S2 | Cuatro categorías cubren el alcance del taller | Subdiagnóstico | Model Card + revisión humana |
| S3 | Latencia p95 &lt;100 ms en edge es alcanzable | Mala UX | Benchmark en CI smoke test |
| S4 | Edge **o** cloud estará disponible para el médico | Sin servicio | Fallback documentado (solo edge) |
| S5 | No se registran datos identificables en logs | Riesgo privacidad | Redacción y retención 30 días |
| S6 | AGUDA permanece ~1–3 % de prevalencia | Métricas engañosas | Recall AGUDA obligatorio en gates |
| S7 | Reglas MVP ≡ lógica acordada del script CSV | Training-serving skew | Tests de paridad baseline |
| S8 | Free tier GCP suficiente para demo | Solo edge | Despliegue híbrido edge prioritario (§5.3) |
| S9 | Ningún agente LLM decide diagnóstico solo | Riesgo ético | AgentOps solo asistencia (horizonte) |
| S10 | Equipo 2–3 personas part-time | Retrasos | Roadmap por fases en §6 |

---

## 2. Etapa: Data Pipeline

**Definición:** ingesta, versionado, anonimización (si aplica), transformación y preparación de datasets para entrenamiento.

**Por qué existe:** sin datos versionados no hay reproducibilidad ni auditoría (“¿con qué CSV se entrenó el modelo en prod?”).

**Para qué en este proyecto:** alimentar entrenamiento LightGBM y vincular cada run MLflow a un snapshot.

### 2.1. Disparador (Schedule / evento)

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | GitHub Actions (`workflow_dispatch`) + cron semanal opcional |
| **Suposición** | Los datos no cambian cada hora; actualización batch es aceptable. |
| **Implicación** | Re-entrenamiento no es tiempo real; se activa también por drift (§5.3). |

### 2.2. Extract Data / Load

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | Scripts Python + **DVC** `dvc pull/push` hacia MinIO/S3 |
| **Entrada** | `data/raw/enfermedades_cardiacas.csv` (fuente original del taller) |
| **Salida** | `data/processed/enfermedades_cardiacas_4categorias.csv` (4 etiquetas) |
| **Justificación DVC vs copia manual** | Hash inmutable; comparación justa entre experimentos; rollback de datos. |
| **Suposición** | El script `scripts/ajustar_cuatro_categorias.py` es determinista. |
| **Implicación** | Cualquier cambio al script exige nuevo tag DVC y re-ejecución de pipeline. |

### 2.3. Personal Data Obfuscation (anonimización)

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | Reglas pandas + validación de esquema (no hay columnas de nombre/documento en CSV actual) |
| **Justificación** | El dataset académico ya es agregado; la etapa queda **documentada** para extensión con EHR real. |
| **Suposición** | No se añadirán columnas identificables sin pasar por esta etapa. |
| **Implicación** | Si se incorporan IDs de paciente, obligatorio pseudonimizar antes de EDA. |

### 2.4. EDA (Exploratory Data Analysis)

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | Jupyter + pandas + matplotlib/seaborn |
| **Enfoque obligatorio** | Distribución de clases (AGUDA ~1,9 %), correlaciones, valores faltantes, rangos (`nivel_colesterol` 1–3, etc.) |
| **Justificación** | Sin EDA documentado no se justifica SMOTE, `class_weight` ni umbrales de gates. |
| **Entregable** | Informe en MLflow como artefacto + capturas en `docs/`. |

### 2.5. Transform / Feature store offline

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | sklearn `ColumnTransformer` + **Feast** (offline store) + export `.parquet` |
| **Justificación** | Un solo pipeline serializado evita training-serving skew respecto a `POST /predecir`. |
| **Suposición** | Los seis campos JSON del MVP son el contrato completo de inferencia. |
| **Implicación** | Añadir un campo al API exige actualizar catálogo, features, GX y Model Card. |

### 2.6. Quality gates (calidad de datos)

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | **Great Expectations** + Pandera en job CI `data-quality` |
| **Reglas ejemplo** | Presiones &gt; 0; colesterol/glucosa en {1,2,3,4}; proporción AGUDA dentro de banda histórica |
| **Justificación** | Coherente con `validar_entrada_minima` del MVP Flask. |
| **Implicación** | Si falla GX, **no** se entrena ni se promueve modelo. |

---

## 3. Etapa: Develop

**Definición:** rama de desarrollo donde convergen **CI de software** y **experimentación ML** antes de integrar a rama principal.

**Por qué:** separar experimentos rápidos de artefactos candidatos a staging; detectar vulnerabilidades y tests rotos temprano.

**Trigger:** merge o PR hacia rama `develop` (o feature branch con CI equivalente).

### 3.1. Security vulnerability analysis

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | **Bandit** (Python) + **Trivy** (imagen Docker) en GitHub Actions |
| **Justificación vs SonarQube** | Menor fricción en repo académico; cubre dependencias e imagen OCI. |
| **Suposición** | No hay secretos en código (usar GitHub Secrets). |

### 3.2. Unit Test

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | pytest |
| **Alcance** | Validación API, paridad reglas MVP, esquema JSON, smoke de carga de modelo |
| **Justificación** | Etapa explícita añadida respecto a propuesta Semana 1 (solo PDF + demo manual). |

### 3.3. Build

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | Docker multi-stage → imagen `:dev` en GHCR |
| **Salida** | Contenedor con código + artefacto de modelo de desarrollo |

### 3.4. Model Training (Develop)

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | **LightGBM** + **Optuna** + **MLflow Tracking** |
| **Baseline** | Reglas en `modelo_simulado.py` como **champion inicial** hasta que LightGBM supere F1 macro y recall AGUDA en hold-out |
| **Estrategia desbalance** | Estratificación en split; `class_weight`; SMOTE opcional; métrica focal: **recall AGUDA** |
| **Suposición** | LightGBM supera reglas solo si mejora F1 macro **y** recall AGUDA en hold-out estratificado. |
| **Implicación** | No se reemplaza el MVP en prod sin evidencia en MLflow. |

### 3.5. Model Evaluation / Model Validation

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | sklearn metrics, matriz de confusión, **SHAP**, k-fold estratificado |
| **Métricas clave** | **Recall** (minimizar falsos negativos en AGUDA), **F1 macro**, AUC-PR en clase minoritaria |
| **Justificación** | Accuracy global es engañosa con ~98 % mayoría LEVE/MODERADA. |
| **Gate humano** | Revisión académica registrada en MLflow antes de pasar a Staging. |

### 3.6. Api Deploy Dev

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | Docker Compose local / entorno dev en Cloud Run (sin tráfico prod) |
| **Actor** | **QA** (equipo) prueba contrato `POST /predecir` y UI |
| **Mejora vs Semana 1** | Despliegue dev repetible vía CI, no solo `docker run` manual documentado en README. |

---

## 4. Etapa: Staging

**Definición:** pre-producción con datos similares a prod, integración end-to-end y registro del modelo.



### 4.1. CI en Staging (Security + Build)

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | GitHub Actions: Bandit, Trivy, build imagen `:staging` |
| **Justificación** | Misma imagen que irá a prod, sin pasos manuales. |

### 4.2. Model Training / Evaluation / Validation (Staging)

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | Re-entrenamiento opcional con datos congelados `data@vN` o reutilización del run aprobado en Develop |
| **Suposición** | Staging usa **exactamente** el snapshot de datos prometido para prod. |

### 4.3. Model Registry — Save Model

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | **MLflow Model Registry** → estado **Staging** |
| **Contenido** | Pipeline sklearn + LightGBM + metadatos (hash DVC, métricas, SHAP summary) |
| **Justificación** | Trazabilidad y rollback; sustituye “Model Registry” genérico del diagrama de referencia. |

### 4.4. Api Deploy ST (Staging)

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | Cloud Run (staging) + Compose local paralelo |
| **Protocolo** | HTTP REST — mismo JSON que MVP |

### 4.5. QA Medical System (validación clínica simulada)

| Aspecto | Detalle |
| :--- | :--- |
| **Definición** | Entorno donde evaluadores/médico simulado prueban casos límite (AGUDA, huérfanas, entradas inválidas). |
| **Tecnología** | Colección de casos JSON + checklist en Model Card + pruebas de integración |
| **Suposición** | “QA Medical” es **simulación académica**, no validación clínica real. |
| **Implicación** | Casos fuera de CSV deben devolver respuesta conservadora, no inventar diagnóstico. |
| **Criterio de salida** | Recall AGUDA ≥ baseline reglas; latencia dentro de SLO; sin regresiones en tests. |

---

## 5. Etapa: PROD y Monitoreo

### 5.1. Request Deploy Prod

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | Aprobación manual en GitHub Environment `production` + workflow `deploy-prod` |
| **Justificación** | Human-in-the-loop antes de servir a usuario final académico. |

### 5.2. Model Registry — Get Model

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | MLflow transición **Staging → Production** |
| **Suposición** | Solo un modelo Production activo por servicio. |

### 5.3. Deploy (Producción)

| Modo | Tecnología | Justificación |
| :--- | :--- | :--- |
| **Edge** | Docker Compose, `localhost:5000` | Privacidad, baja latencia, sin internet (§5.3) |
| **Cloud** | **Cloud Run** + TLS + API key | Acceso remoto; escalado automático |
| **Contrato único** | `POST /predecir` | Misma lógica de negocio; evita duplicar código |

### 5.4. Medical System (usuario)

| Aspecto | Detalle |
| :--- | :--- |
| **Usuario** | Médico / evaluador con formulario web o cliente REST |
| **Suposición** | Usuario lee disclaimer y no usa salida como diagnóstico real. |

### 5.5. Monitoring Services

#### Performance

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | **Prometheus** (métricas) + **Grafana** (dashboards) |
| **SLI/SLO** | Disponibilidad 99,5 %; p95 &lt;100 ms edge / &lt;300 ms cloud; 5xx &lt;0,5 % |
| **Justificación** | Igual rol que en ejemplo `entrega3`; latencia crítica para flujo clínico simulado. |

#### Logs

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | Logs JSON estructurados + Grafana Loki (opcional) |
| **Contenido** | Timestamp, latencia, clase predicha, versión modelo; **sin** PII |
| **XAI en logs** | Top-3 features SHAP por predicción (muestreo en prod académico) |

#### Drift y re-entrenamiento

| Aspecto | Detalle |
| :--- | :--- |
| **Tecnología** | **Evidently AI** (reportes semanales) + alertas Prometheus |
| **Concept drift** | Cambio en epidemiología simulada o definición de síntomas |
| **Data drift** | Distribución de presión/glucosa/colesterol se desvía del train set |
| **Activación** | Si recall offline cae &gt;10 % vs baseline **o** drift &gt; τ → workflow **re-entrenamiento** reinicia en **Data Pipeline** (§5.5) |
| **Mejora vs Semana 1** | MVP no contemplaba drift ni ciclo cerrado; solo inferencia puntual. |

---

## 6. Diagrama general del pipeline

La topología replica el diagrama de referencia del curso (cuatro columnas + monitoreo + retroalimentación). Las etiquetas del dibujo muestran **nuestras** herramientas.

![Diagrama del pipeline MLOps](../PipeLineML.png)

**Lectura del diagrama:** el flujo avanza de izquierda a derecha. El monitoreo en PROD puede disparar un nuevo ciclo en Data Pipeline. Develop y Staging repiten patrones CI/CD + ML, pero Staging añade **registro MLflow** y **QA Medical System** antes de producción.

Documento complementario de cambios: [CHANGELOG](../CHANGELOG.md)

---

## 7. Comparación con MVP Semana 1 (implementado)

| Aspecto | Semana 1 (`main`) | Unidad 3 (propuesta) |
| :--- | :--- | :--- |
| Documento pipeline | PDF en rama [`main`](https://github.com/WillianReinaG/2MLops_unidad1/blob/main/docs/punto%201%20descripcion%20pipeline%20MLops.pdf) | Este documento + README + CHANGELOG + [`PipeLineML.png`](../PipeLineML.png) |
| Modelo | Reglas fijas `modelo_simulado.py` | LightGBM promovido con gates vs baseline |
| Datos | Carpeta `data/` sin versionado | DVC + tags |
| Despliegue | Docker manual | CI/CD → GHCR → edge + Cloud Run |
| Monitoreo | No documentado | Prometheus + Grafana + Evidently |
| Seguridad CI | No documentado | Bandit + Trivy |
| Suposiciones | Implícitas | Tabla S1–S10 + suposiciones por etapa |

---

## 8. Roadmap de implementación (propuesto)

| Fase | Semanas | Entregable | Ops |
| :--- | :--- | :--- | :--- |
| 0 | ✓ | MVP Flask/Docker | Hecho en Semana 1 |
| 1 | 2 | DVC, GX, catálogo | MLOps + DevOps |
| 2 | 2 | LightGBM + MLflow + SHAP | MLOps |
| 3 | 1 | GHCR, Compose, Cloud Run staging | DevOps |
| 4 | 1 | Grafana, Evidently, runbooks | AIOps |
| 5 | 1 | Promoción prod + Model Card v final | Gobernanza |

---

## Referencias

- estructura de referencia vista del curso  
- Propuesta Semana 1 (PDF solo en rama [`main`](https://github.com/WillianReinaG/2MLops_unidad1/blob/main/docs/punto%201%20descripcion%20pipeline%20MLops.pdf))


---
