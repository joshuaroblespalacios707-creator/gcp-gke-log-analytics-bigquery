**🇲🇽 Español** | [🇬🇧 English](#-monitoring-log-analytics--cybersecurity-on-google-cloud)

# 📊 Monitoreo, Log Analytics y Ciberseguridad en Google Cloud

Documentación técnica de un laboratorio práctico de arquitectura de microservicios en GKE, con almacenamiento avanzado de logs mediante Log Analytics, vinculado a BigQuery para análisis SQL de rendimiento y latencia.

---

## 📌 Módulo 1: Infraestructura GKE, Log Sinks y Análisis de Latencia con SQL

### 🎯 Objetivo General

Desplegar una arquitectura de microservicios en **Google Kubernetes Engine (GKE)**, estructurar un almacenamiento de logs avanzado mediante un **Log Bucket** con soporte de **Log Analytics**, y vincularlo a **BigQuery** para ejecutar consultas SQL de rendimiento y latencia.

---

### 🏗️ Componentes de Infraestructura

| Componente | Detalle |
|---|---|
| Clúster GKE | Entorno donde se ejecuta la aplicación de microservicios "Online Boutique" |
| Log Bucket personalizado | `day2ops-log`, configurado con Log Analytics activado |
| Log Sink | Tubería `day2ops-sink`, para filtrar y enrutar registros específicos |
| BigQuery Dataset | Conjunto de datos enlazado para persistencia y análisis multivariable a largo plazo |

---

### ⚙️ Pasos de Implementación y Código

**Paso 1 — Creación del Log Bucket Personalizado**

Se creó un bucket de logs con retención personalizada y soporte para consultas SQL directas:

```bash
gcloud logging buckets create day2ops-log \
    --location=global \
    --enable-analytics \
    --retention-days=30 \
    --description="Bucket personalizado para logs de GKE y analisis con Log Analytics"
```

**Paso 2 — Configuración del Log Sink**

Se configuró un enrutador de logs para capturar únicamente la telemetría proveniente de los contenedores de Kubernetes (`k8s_container`) y enviarla al bucket recién creado:

```bash
gcloud logging sinks create day2ops-sink \
    logging.googleapis.com/projects/$PROJECT_ID/locations/global/buckets/day2ops-log \
    --log-filter="resource.type=\"k8s_container\""
```

**Paso 3 — Vinculación con BigQuery**

Se creó un enlace para exponer los datos del Log Bucket hacia BigQuery, sin necesidad de duplicar el almacenamiento:

```bash
gcloud logging links create day2ops-link \
    --bucket=day2ops-log \
    --location=global \
    --dataset=day2ops_logs_dataset
```

**Paso 4 — Análisis de Latencia mediante SQL (Log Analytics)**

Con la integración lista, se ejecutó una consulta SQL para analizar los tiempos de respuesta y detectar cuellos de botella en los microservicios:

```sql
SELECT
  timestamp,
  resource.labels.pod_name AS pod_origen,
  httpRequest.requestUrl AS url_solicitada,
  httpRequest.status AS codigo_respuesta,
  CAST(JSON_VALUE(jsonPayload.duration) AS FLOAT64) AS latencia_segundos
FROM
  `day2ops_logs_dataset._AllLogs`
WHERE
  resource.type = "k8s_container"
  AND httpRequest.status IS NOT NULL
ORDER BY
  latencia_segundos DESC
LIMIT 20;
```

---

### 🔧 Errores Diagnosticados y Solución

- **Error:** al intentar consultar los datos en BigQuery o Log Analytics inmediatamente después de crear el sink, la consulta SQL devolvía 0 resultados.
- **Causa:** retraso (*propagation delay*) en la ingesta inicial de datos y en la creación del dataset enlazado.
- **Solución:** se generó tráfico sintético en la aplicación web para forzar la emisión de peticiones HTTP en los pods, y se esperó un lapso de 3 a 5 minutos hasta que los primeros bloques de registros fueron indexados en el bucket.

---

## 🎓 Conceptos Aprendidos

- Configuración de Log Buckets personalizados con retención y Log Analytics habilitado
- Enrutamiento selectivo de logs mediante Log Sinks filtrados por tipo de recurso
- Vinculación de logs a BigQuery sin duplicar almacenamiento (Log Analytics linked datasets)
- Análisis de latencia y rendimiento de microservicios mediante consultas SQL sobre logs de Kubernetes
- Diagnóstico de retrasos de propagación en pipelines de ingesta de logs

---

## 📌 Notas

Este laboratorio forma parte de mi preparación continua hacia la certificación **Google Professional Cloud Security Engineer**, con enfoque práctico en observabilidad de microservicios y análisis de logs a escala en GCP.

---
---

[⬆ Español version above](#-monitoreo-log-analytics-y-ciberseguridad-en-google-cloud)

# 📊 Monitoring, Log Analytics & Cybersecurity on Google Cloud

Technical documentation of a hands-on lab building a microservices architecture on GKE, with advanced log storage via Log Analytics, linked to BigQuery for SQL-based performance and latency analysis.

---

## 📌 Module 1: GKE Infrastructure, Log Sinks & SQL Latency Analysis

### 🎯 General Objective

Deploy a microservices architecture on **Google Kubernetes Engine (GKE)**, structure an advanced log storage layer via a **Log Bucket** with **Log Analytics** support, and link it to **BigQuery** to run SQL queries for performance and latency analysis.

---

### 🏗️ Infrastructure Components

| Component | Detail |
|---|---|
| GKE Cluster | Environment running the "Online Boutique" microservices application |
| Custom Log Bucket | `day2ops-log`, configured with Log Analytics enabled |
| Log Sink | `day2ops-sink` pipeline, filtering and routing specific log entries |
| BigQuery Dataset | Linked dataset for long-term persistence and multivariable analysis |

---

### ⚙️ Implementation Steps & Code

**Step 1 — Creating the Custom Log Bucket**

Created a log bucket with custom retention and support for direct SQL queries:

```bash
gcloud logging buckets create day2ops-log \
    --location=global \
    --enable-analytics \
    --retention-days=30 \
    --description="Bucket personalizado para logs de GKE y analisis con Log Analytics"
```

**Step 2 — Log Sink Configuration**

Configured a log router to capture only telemetry originating from Kubernetes containers (`k8s_container`) and forward it to the newly created bucket:

```bash
gcloud logging sinks create day2ops-sink \
    logging.googleapis.com/projects/$PROJECT_ID/locations/global/buckets/day2ops-log \
    --log-filter="resource.type=\"k8s_container\""
```

**Step 3 — BigQuery Linking**

Created a link to expose the Log Bucket's data to BigQuery, without duplicating storage:

```bash
gcloud logging links create day2ops-link \
    --bucket=day2ops-log \
    --location=global \
    --dataset=day2ops_logs_dataset
```

**Step 4 — Latency Analysis via SQL (Log Analytics)**

With the integration in place, executed a SQL query to analyze response times and detect bottlenecks across microservices:

```sql
SELECT
  timestamp,
  resource.labels.pod_name AS pod_origen,
  httpRequest.requestUrl AS url_solicitada,
  httpRequest.status AS codigo_respuesta,
  CAST(JSON_VALUE(jsonPayload.duration) AS FLOAT64) AS latencia_segundos
FROM
  `day2ops_logs_dataset._AllLogs`
WHERE
  resource.type = "k8s_container"
  AND httpRequest.status IS NOT NULL
ORDER BY
  latencia_segundos DESC
LIMIT 20;
```

---

### 🔧 Diagnosed Issues & Resolution

- **Issue:** querying the data in BigQuery or Log Analytics immediately after creating the sink returned 0 results.
- **Root cause:** propagation delay in the initial data ingestion and linked dataset creation.
- **Resolution:** generated synthetic traffic on the web application to force HTTP requests across the pods, and waited 3 to 5 minutes until the first batches of log records were indexed in the bucket.

---

## 🎓 Key Concepts Learned

- Configuring custom Log Buckets with retention and Log Analytics enabled
- Selective log routing via Log Sinks filtered by resource type
- Linking logs to BigQuery without duplicating storage (Log Analytics linked datasets)
- Analyzing microservice latency and performance via SQL queries over Kubernetes logs
- Diagnosing propagation delays in log ingestion pipelines

---

## 📌 Notes

This lab is part of my ongoing preparation for the **Google Professional Cloud Security Engineer** certification, with a hands-on focus on microservices observability and large-scale log analysis in GCP.
