# Multi-host infrastructure monitoring

A **Docker Compose / Prometheus / Grafana** home-lab configuration for monitoring two Linux hosts and a Windows workstation. It defines persistent monitoring services, 15-second scrape intervals, a local Linux exporter, and restart policies.

This project shows the configuration and troubleshooting work relevant to systems operations: collecting host metrics, separating exporter/network failures from host failures, retaining monitoring data, and inspecting disk capacity.

**Review the evidence:** [validation record and troubleshooting runbook](docs/VALIDATION.md). Both YAML files were parsed and their service/scrape configuration inspected. Container startup, target reachability, dashboards, and restart recovery were not exercised in that check.

## Architecture and scope

| Component | Responsibility | Checked-in evidence |
| --- | --- | --- |
| Prometheus | Scrape and store metrics; configuration reload/startup diagnostics | [docker-compose.yaml](docker-compose.yaml), [prometheus/prometheus.yml](prometheus/prometheus.yml) |
| Grafana | Query Prometheus and visualize host metrics | Service definition and persistent `grafana_data` volume; dashboards are configured manually |
| Node Exporter | Expose metrics from the Linux machine running Compose | Host networking, host PID namespace, read-only root filesystem mount |
| Other Linux host | Supply its own exporter on port `9100` | Scrape target only; exporter installation is external to this repository |
| Windows workstation | Supply Windows exporter metrics on port `9182` | Scrape target only; exporter installation is external to this repository |

Prometheus scrapes the three configured LAN targets. Grafana reaches Prometheus through the Compose network at `http://prometheus:9090`. Named volumes retain Prometheus data, Grafana state, and the local exporter's textfile directory. All three services specify `restart: unless-stopped`.

The repository does **not** contain exported dashboard JSON, provisioning files, alert rules, exporter installers for the other hosts, or a validated automated backup/recovery process. Those are separate from the configuration that is already present.

## Setup on a Linux lab host

Requirements: Docker Engine with Compose, and reachable exporters on the target hosts. The local exporter uses Linux host namespaces and a host filesystem mount; it is intended for a Linux host you administer.

```bash
git clone https://github.com/danielmakoutz-popcorn/monitoring.git
cd monitoring
```

Edit `prometheus/prometheus.yml` to match your hosts. The checked-in IP addresses are lab-specific. Keep the Linux exporters on the intended port `9100` and Windows exporter on `9182`, or update the targets accordingly. Restrict exporter/firewall access to the monitoring host.

```bash
docker compose config --quiet
docker compose up -d
docker compose ps
docker compose logs --tail=100 prometheus grafana node_exporter
```

These are setup commands to run locally, **not a recorded clean-install pass**.

| Interface | Local URL / port |
| --- | --- |
| Prometheus | `http://127.0.0.1:9090` |
| Grafana | `http://127.0.0.1:3000` |
| Local Node Exporter | Port `9100` on the Linux host |

In Grafana, complete first-login setup and add a Prometheus data source with URL **`http://prometheus:9090`**. `localhost:9090` inside the Grafana container points to Grafana's own container, not Prometheus. Add dashboards suitable for the installed exporter versions; none are automatically provisioned by this repository.

The supplied Compose port mappings publish Prometheus and Grafana on host interfaces. Keep access controlled on the lab network; this configuration is not a public authenticated deployment.

## Verify collection and troubleshoot

```bash
curl -fsS http://127.0.0.1:9090/-/ready
curl -fsS http://127.0.0.1:3000/api/health
curl -fsSG --data-urlencode 'query=up' http://127.0.0.1:9090/api/v1/query
```

Check Prometheus's **Targets** page and the `up` query for each configured job. `up=1` means a successful scrape, not a healthy business application. If a target is down, check exporter availability, IP/port, firewall rules, and scrape errors before diagnosing the host itself. See the [runbook](docs/VALIDATION.md#local-validation-runbook) for metric queries and a controlled collection-gap exercise.

## Reported operational use

The project author's résumé reports using monitored host data to diagnose a disk-capacity outage and adjusting Bitcoin Core pruning. That is **author-reported experience**, not a newly reproduced incident. This repository has no dated incident logs, dashboard captures, or before/after capacity values to independently substantiate the outcome.

The runbook shows how to collect that evidence in a future lab exercise without inventing measurements or recovery times.

## Stop, persistence, and next improvements

```bash
docker compose down
```

Named volumes are retained by this command. `docker compose down -v` removes them. Restart policies are present in configuration; actual crash/restart recovery remains unverified here.

Next improvements, **not completed**: pin container versions instead of `latest`, export sanitized dashboards and provision the data source, add host-capacity/target-loss alerts, and record a dated restart/volume-recovery exercise. No uptime, response-time, or recovery-time measurements are claimed.
