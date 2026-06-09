# Monitoreo con Docker — Prometheus + Grafana + Node Exporter + cAdvisor

Stack de monitoreo para contenedores Docker y recursos del servidor.

## Servicios

| Servicio | Puerto | Imagen | Función |
|---|---|---|---|
| **Grafana** | `3005` | `grafana/grafana:13.0.1` | Visualización de métricas y dashboards |
| **Prometheus** | `9090` | `prom/prometheus:v3.12.0` | Almacenamiento y consulta de métricas |
| **Node Exporter** | `9100` | `quay.io/prometheus/node-exporter:v1.11.1` | Métricas del sistema host (CPU, RAM, disco, red) |
| **cAdvisor** | `8080` | `ghcr.io/google/cadvisor:v0.57.0` | Métricas de contenedores Docker |

### Puertos locales

- Grafana: **http://localhost:3005**
- Prometheus: **http://localhost:9090**
- Node Exporter: **http://localhost:9100/metrics**
- cAdvisor: **http://localhost:8080**

## Requisitos

- Docker Engine 24+
- Docker Compose v2

## Uso

```bash
# Iniciar el stack (en segundo plano)
docker compose up -d

# Ver logs
docker compose logs -f

# Detener
docker compose down

# Detener y eliminar volúmenes (borra datos)
docker compose down -v

# Recargar configuración de Prometheus sin reiniciar
curl -X POST http://localhost:9090/-/reload

# Verificar estado
docker compose ps
```

## Dashboard

El archivo `dashboard-metrics.json` contiene un dashboard en español con paneles de contenedores (cAdvisor) y sistema (Node Exporter).

### Importar en Grafana

1. Ir a **http://localhost:3005**
2. Iniciar sesión (ver credenciales abajo)
3. Menú → **Dashboards** → **Import**
4. Subir `dashboard-metrics.json` o pegar el JSON
5. Seleccionar **Prometheus** como data source
6. Click en **Import**

El dashboard incluye:

**Sistema** — Paneles del host:
- Contenedores Activos
- CPU del Servidor
- Disco Libre y Usado
- RAM del Servidor

**Visión General del Host** — Métricas agregadas de contenedores:
- Uso de CPU (Núcleos)
- Memoria Total por Host

**CPU, Memoria, I/O, Red** — Desglose por contenedor individual
**Detalles** — Reinicios de contenedores

## Credenciales

Archivo `.env`:

```
GRAFANA_ADMIN_USER=admin
GRAFANA_ADMIN_PASSWORD=admin
```

Puedes cambiarlas editando `.env` y reiniciando:

```bash
docker compose up -d --force-recreate grafana
```

## Estructura de archivos

```
monitoring/
├── .env                         # Credenciales de Grafana
├── docker-compose.yml           # Definición de servicios
├── prometheus.yml               # Configuración de scrape de Prometheus
├── prometheus-alerts.yml        # Reglas de alertas
├── dashboard-metrics.json       # Dashboard para importar en Grafana
├── grafana/
│   └── provisioning/
│       └── datasources/
│           └── datasource.yml   # Data source Prometheus preconfigurado
└── README.md
```

## Alertas (Prometheus)

Configuradas en `prometheus-alerts.yml`:

| Alerta | Condición | Severidad |
|---|---|---|
| **InstanceDown** | Servicio sin responder por 1 min | critical |
| **HighCpuUsage** | CPU del host > 80% por 2 min | warning |
| **HighMemoryUsage** | RAM del host > 85% por 2 min | warning |
| **DiskSpaceLow** | Disco raíz < 10% libre por 2 min | critical |

Las alertas se pueden ver en Prometheus → http://localhost:9090/alerts o configurar un notificador en Grafana (Alerting → Contact points).

## Notas

- Los datos de Prometheus y Grafana persisten en volúmenes Docker (`prometheus-data`, `grafana-data`).
- La retención de datos en Prometheus es de 30 días.
- Si cambias el `.env`, reinicia solo Grafana con `--force-recreate` para que tome las nuevas variables.
- Para exponer en un VPS o internet, usa un reverse proxy (nginx/caddy) con HTTPS.
