# Observability Stack

A self-hosted observability stack — metrics, logs, dashboards, and HTTPS — deployed on Oracle Cloud (Ampere A1 / ARM) via Docker Compose.

## 🔴 Live Demo

**[grafana.livefleet.site](https://grafana.livefleet.site)**

A running Grafana instance, publicly reachable over HTTPS via Caddy (automatic TLS, Let's Encrypt). This isn't a screenshot or a local-only setup — it's the actual deployed stack, monitoring the host and containers it runs on in real time.
---

## What's in the stack

| Component | Role |
|---|---|
| **Prometheus** | Scrapes and stores metrics from the host, containers, and any app exposing `/metrics` |
| **Alertmanager** | Routes alert conditions Prometheus fires to Slack/email/etc. |
| **Node Exporter** | Host-level metrics — CPU, memory, disk, network |
| **cAdvisor** | Per-container resource metrics |
| **Loki + Promtail** | Centralized log aggregation from every container |
| **Grafana** | Dashboards — unified view over metrics + logs |
| **Caddy** | Reverse proxy + automatic HTTPS for the public demo site |

Everything runs via a single `docker-compose.yml`. New app containers are picked up automatically by Prometheus (Docker service discovery) and Promtail (log shipping) — no manual config edits per app.

## Architecture

```
                     ┌─────────────┐
    Appps    ───────▶│  Prometheus  │──scrapes──▶ node-exporter, cAdvisor,
  (expose /metrics)  │              │             app containers (auto-discovered)
                     └──────┬───────┘
                            ▼
                     ┌──────────────┐
                     │ Alertmanager │
                     └──────────────┘

   App containers ──▶ Promtail ──▶ Loki ──┐
                                            ├──▶ Grafana ──▶ Caddy (HTTPS) ──▶ 🌐 public internet
                     Prometheus ───────────┘
```

Caddy handles certificate issuance and renewal automatically — no manual cert management.

## Repo layout

```
observability-stack/
├── docker-compose.yml
├── Caddyfile
├── prometheus/
│   ├── prometheus.yml       # scrape config + Docker auto-discovery
│   └── alert.rules.yml      # symptom-based alert rules
├── alertmanager/
│   └── alertmanager.yml     # alert routing (receiver TBD — see below)
├── loki/
│   └── loki-config.yml
├── promtail/
│   └── promtail-config.yml
├── grafana/
│   └── provisioning/        # auto-wired datasources + dashboard loading
```
