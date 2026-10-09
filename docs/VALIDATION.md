# Monitoring — validation evidence and runbook

Recorded **2026-10-09 UTC / 2026-10-08 Pacific**. Configuration inspected at [`31799ca`](https://github.com/danielmakoutz-popcorn/monitoring/commit/31799ca45c04b6c635c92e6dab8d102d219bb090). This update changes documentation only.

## Executed: configuration parsing

Environment: Python **3.12.14**, PyYAML **6.0.3**. Docker and `promtool` were unavailable in the review environment; no hosts were probed.

The following check was executed from the repository root:

```python
from pathlib import Path
import yaml

compose = yaml.safe_load(Path('docker-compose.yaml').read_text())
prom = yaml.safe_load(Path('prometheus/prometheus.yml').read_text())
assert set(compose['services']) == {'prometheus', 'grafana', 'node_exporter'}
assert all(s['restart'] == 'unless-stopped' for s in compose['services'].values())
assert prom['global']['scrape_interval'] == '15s'
assert prom['global']['evaluation_interval'] == '15s'
assert len(prom['scrape_configs']) == 3
assert len(compose['volumes']) == 3
```

Actual result: exit code **0**.

```text
PASS YAML parse: 2 configuration files
PASS configuration inspection: 3 services, 3 scrape jobs, 15-second intervals
PASS configuration inspection: 3 named volumes, unless-stopped on all services
NOT RUN Docker Compose validation, promtool, containers, exporters, dashboards, or recovery
```

This proves parseability and the inspected configuration values. It does not prove Compose or Prometheus schema validity, successful collection, a provisioned dashboard, alert delivery, or automatic recovery. No GitHub Actions workflow or run was present at inspection.

## Author-reported incident

The existing résumé reports a disk-capacity outage diagnosed with monitored host data, followed by an adjustment to Bitcoin Core pruning. The source configuration supports filesystem metric collection, but the repository contains no dated incident artifacts, exact original metrics, pruning configuration change, or measured recovery result.

Do not treat the runbook below as a reconstruction of actions actually taken in that incident. It is a proposed way to collect supporting evidence in a future controlled exercise.

## Local validation runbook

The following checks are **not executed results**. Run them on the authorized lab after updating target addresses.

1. Run `docker compose config --quiet` and validate the Prometheus configuration with `promtool check config prometheus/prometheus.yml` from a compatible Prometheus installation.
2. Start the stack, inspect `docker compose ps` and recent logs, and check Prometheus `/-/ready` and Grafana `/api/health`.
3. Inspect the Targets page and run `up`. Capture each job's timestamped scrape status; do not substitute sample values for actual output.
4. Inspect capacity using the PromQL expressions below and compare them with local host readings such as `df -h`. Exclude virtual filesystems as appropriate for the host.
5. In the local lab, stop the Compose `node_exporter` service and observe the corresponding target's scrape failure. Start it again and record recovery. This tests a deliberate collection gap, not automatic crash recovery.
6. Restart Prometheus and Grafana and check whether stored history and configured dashboards remain available. A persistent-volume declaration alone does not validate recovery.

| Question | Example PromQL to run |
| --- | --- |
| Can each configured target be scraped? | `up` |
| What proportion of a Linux filesystem is available? | `100 * node_filesystem_avail_bytes{fstype!~"tmpfs|overlay|squashfs"} / node_filesystem_size_bytes{fstype!~"tmpfs|overlay|squashfs"}` |
| Has a Linux exporter process restarted? | `process_start_time_seconds{job=~"btcnode-linux|laptop-linux"}` |

Exporter metrics and labels must be checked against the actual installed versions. These query examples are not captured measurements. `process_start_time_seconds` describes the exporter process, not host uptime.

## Evidence to retain next

Keep a sanitized target-status capture, dashboard JSON, a dated filesystem-capacity observation, and restart/persistence results. Record the source commit, command, expected behavior, actual output, and any failure. Alerts, retention tuning, volume backups, and measured incident recovery remain pending.
