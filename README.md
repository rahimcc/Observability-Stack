# Observability Stack

A self-hosted observability stack — metrics, logs, dashboards, and HTTPS — deployed on Oracle Cloud (Ampere A1 / ARM) via Docker Compose.

## 🔴 Live Demo

**[grafana.livefleet.site](https://grafana.livefleet.site)**

A running Grafana instance, publicly reachable over HTTPS via Caddy (automatic TLS, Let's Encrypt). This isn't a screenshot or a local-only setup — it's the actual deployed stack, monitoring the host and containers it runs on in real time.

> Default credentials are for demo purposes only — change them before using this for anything beyond a demo. See [Security notes](#security-notes) below.

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
   Your apps ───────▶│  Prometheus  │──scrapes──▶ node-exporter, cAdvisor,
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

## Running it yourself

```bash
git clone <this-repo>
cd observability-stack
docker network create observability   # if not already created
docker compose up -d
docker compose ps                      # confirm everything's running
```

Access points (local):
- Grafana: `http://localhost:3000`
- Prometheus: `http://localhost:9090`
- Alertmanager: `http://localhost:9093`

To expose your own instance publicly like the demo site, point a DNS A record at your host and add a `Caddyfile`:
```
your-subdomain.yourdomain.com {
    reverse_proxy grafana:3000
}
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
└── PROGRESS.md              # build log, best practices, open items
```

## Security notes

- Grafana admin credentials are set via environment variables in `docker-compose.yml` — **rotate the default before real use.**
- Prometheus, Alertmanager, and cAdvisor have **no built-in authentication** — only Grafana (behind Caddy/HTTPS) is intended to be public. Everything else should stay on the internal Docker network / localhost only.
- Caddy's `/data` volume holds certificates — back it up or at least don't delete it casually, or you'll re-issue certs (and risk Let's Encrypt rate limits) unnecessarily.

## Status / open items

See [`PROGRESS.md`](./PROGRESS.md) for the full build log, best practices applied, and what's left — currently: wiring a real Alertmanager receiver, adding a Pushgateway + dbt-test bridge for data-quality signals, and building out custom Grafana dashboards beyond the community defaults.
