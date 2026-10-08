# Infrastructure Monitoring with Prometheus + Grafana

A **Docker Compose observability lab** for monitoring Linux and Windows machines on a private network.

The repository defines a Prometheus time-series collector, Grafana visualization service, and a Node Exporter for Linux host metrics.

## Components

| Service | Purpose | Local port |
| --- | --- | --- |
| Prometheus | Scrape, store, and query time-series metrics | `9090` |
| Grafana | Explore metrics and build dashboards | `3000` |
| Node Exporter | Export Linux host metrics | `9100` (host networking) |

The [Prometheus configuration](prometheus/prometheus.yml) defines scrape jobs for a Linux node, Linux laptop, and Windows workstation. It expects an accessible exporter at each target. The addresses in the sample configuration are **LAN-specific** and should be changed for another environment.

## Run locally

Requirements: Docker Engine and Docker Compose, plus appropriate metric exporters running on the target systems.

```bash
docker compose up -d
docker compose ps
```

Open:

- Prometheus: `http://localhost:9090`
- Grafana: `http://localhost:3000`

In Grafana, add **Prometheus** as a data source using `http://prometheus:9090`, then create dashboards or import dashboards appropriate to your exporters.

Stop the lab:

```bash
docker compose down
```

Docker volumes preserve the Prometheus and Grafana application data across container restarts.

## Operational notes

- This repository provides the monitoring services and scrape configuration, **not preprovisioned dashboards or alert rules**.
- The Node Exporter container uses host networking and a read-only view of the host filesystem, which is appropriate only on a machine you control.
- Do not expose Grafana, Prometheus, or exporter ports directly to the public internet. Add authentication, network controls, and TLS before remote access.
- Production environments should pin container image versions, back up persistent volumes, and define alerts and retention policies.

## Skills demonstrated

**Docker Compose, infrastructure monitoring, metric collection, Prometheus scraping, Grafana dashboards, Linux/Windows observability, and service troubleshooting.**
