# Greenhouse Monitoring Test Application

A Python application that demonstrates OpenTelemetry integration with metrics, logs, and distributed tracing using Prometheus, Loki, and Tempo.

## Overview

The [app.py](app.py) application is a test service that showcases observability best practices by generating:

- **Metrics**: Request counters exposed via Prometheus format
- **Logs**: Structured logging at multiple levels (DEBUG, INFO, WARNING, ERROR, CRITICAL)
- **Traces**: Distributed tracing spans with attributes and events

The application runs continuously, processing simulated requests with a 2-second interval, making it ideal for testing and visualizing telemetry data in Grafana.

## Architecture

The app integrates with the monitoring stack defined in [docker-compose.yml](../../docker-compose.yml):

- **OpenTelemetry Collector**: Receives telemetry data via OTLP (gRPC on port 4317)
- **Prometheus**: Scrapes and stores metrics
- **Loki**: Stores and queries logs
- **Tempo**: Stores and queries distributed traces
- **Grafana**: Visualizes all telemetry data

## Requirements

- Docker and Docker Compose
- Python 3.11+ (for local development only)

## Running with Docker (Recommended)

The greenhouse-app is included in the main docker-compose stack and will run continuously as a container.

### Start the Complete Stack

From the project root directory:

```bash
docker-compose up -d
```

This will start all services including:
- **greenhouse-app** on port 8000 (Prometheus metrics)
- OpenTelemetry Collector on ports 4317 (gRPC) and 4318 (HTTP)
- Prometheus on port 9090
- Loki on port 3100
- Tempo on port 3200
- Grafana on port 3000

### View Logs

```bash
docker-compose logs -f greenhouse-app
```

### Rebuild After Changes

If you modify the application code:

```bash
docker-compose build greenhouse-app
docker-compose up -d greenhouse-app
```

### Stop the Stack

```bash
docker-compose down
```

## Running Locally (Development)

For local development without Docker:

### Installation

Using pip:

```bash
pip install -r requirements.txt
```

Using uv (recommended):

```bash
uv sync
```

### Start the Monitoring Stack

First, start the monitoring services (excluding greenhouse-app):

```bash
cd ../..
docker-compose up -d otel-collector prometheus loki tempo grafana
```

### Run the Application

From this directory:

```bash
python app.py
```

Or with uv:

```bash
uv run app.py
```

The application will:
- Start a Prometheus metrics server on port 8000
- Continuously process simulated requests with 2-second intervals
- Send logs to the OpenTelemetry Collector
- Send traces to Tempo
- Expose metrics for Prometheus to scrape

## Configuration

The application uses the following environment variable:

- `OTEL_EXPORTER_OTLP_ENDPOINT`: OpenTelemetry Collector endpoint
  - Docker: `http://otel-collector:4317` (set automatically)
  - Local: `http://localhost:4317` (default)

To use a different endpoint locally:

```bash
export OTEL_EXPORTER_OTLP_ENDPOINT=http://custom-host:4317
python app.py
```

## Dockerfile

The [Dockerfile](Dockerfile) builds a lightweight Python 3.11 image that:
- Installs dependencies from requirements.txt
- Runs the application with unbuffered output
- Exposes port 8000 for Prometheus metrics scraping

## Viewing Telemetry Data

Once the application is running, access Grafana to visualize the data:

1. Open [http://localhost:3000](http://localhost:3000) in your browser
2. Login with credentials: `admin` / `admin`
3. Navigate to:
   - **Dashboards > Greenhouse App - Full Observability**: Complete dashboard with metrics, logs, and traces
   - **Explore > Prometheus**: View metrics (`app_requests_total`)
   - **Explore > Loki**: Query logs from the `greenhouse-app` service
   - **Explore > Tempo**: Visualize distributed traces and spans

## Telemetry Details

### Metrics

- `app_requests_total`: Counter of total requests processed

### Logs

Logs are sent with these resource attributes:
- `service.name`: `greenhouse-app`
- `service.instance.id`: `instance-1`

Log levels generated:
- DEBUG (every 10th request starting at 1)
- INFO (every request + additional at 2)
- WARNING (every 10th request starting at 3)
- ERROR (every 10th request starting at 4)
- CRITICAL (every 10th request starting at 5)

### Traces

Each request creates a span named `process_request` with:
- Attribute: `request.number` (integer)
- Event: "Request processed"
