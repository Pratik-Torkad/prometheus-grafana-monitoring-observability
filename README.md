# Prometheus, Grafana and Alertmanager Monitoring Stack

A local monitoring and observability lab using Docker Compose. Prometheus collects metrics, Grafana visualizes them, and Alertmanager manages alerts. Node Exporter exposes host metrics and cAdvisor exposes container metrics.

> **Local-only by default:** all web interfaces are bound to 127.0.0.1. No AWS resources, cloud instances, public endpoints, or paid monitoring services are created.

## Architecture

~~~mermaid
flowchart TD
  Host[Host OS metrics] --> NE[Node Exporter]
  Containers[Docker container metrics] --> CA[cAdvisor]
  NE --> P[Prometheus]
  CA --> P
  AM[Alertmanager] <--> P
  P --> G[Grafana dashboards]
  P --> Rules[Prometheus alert rules]
  Rules --> AM
~~~

## Components and versions

| Component | Version | Purpose |
|---|---:|---|
| Prometheus | 3.13.2 LTS | Metrics collection and time-series database |
| Grafana | 13.3.0 | Dashboards and exploration |
| Alertmanager | 0.34.2 | Alert grouping, silencing and routing |
| Node Exporter | 1.12.1 | Host CPU, memory, filesystem and network metrics |
| cAdvisor | 0.60.6 | Container CPU, memory and resource metrics |

Image tags are pinned for repeatable builds. Re-check upstream release notes before upgrading.

## Repository layout

~~~text
.
├── .github/workflows/validate.yml
├── alertmanager/alertmanager.yml
├── grafana/
│   ├── dashboards/docker-host-overview.json
│   └── provisioning/
│       ├── dashboards/dashboard.yml
│       └── datasources/prometheus.yml
├── prometheus/
│   ├── prometheus.yml
│   └── rules/alerts.yml
├── docker-compose.yml
├── .env.example
├── .gitignore
├── SECURITY.md
└── README.md
~~~

## Prerequisites

- Docker Desktop with Docker Compose v2, or Docker Engine and the Docker Compose plugin.
- At least 4 GB free memory recommended for the local stack.
- Git to clone the repository.

On Docker Desktop for Windows/macOS, Node Exporter and cAdvisor observe the Linux VM that runs containers, not all native host operating-system counters. For native host metrics, run this stack on a supported Linux host.

## 1. Clone and configure

~~~bash
git clone https://github.com/Pratik-Torkad/prometheus-grafana-monitoring-observability.git
cd prometheus-grafana-monitoring-observability
~~~

Create a local environment file.

Linux/macOS:

~~~bash
cp .env.example .env
~~~

Windows PowerShell:

~~~powershell
Copy-Item .env.example .env
~~~

Edit the .env file and replace the example Grafana password with a long random local password. The .env file is ignored by Git and must never be committed.

## 2. Validate the Compose configuration

~~~bash
docker compose config --quiet
~~~

## 3. Start the monitoring stack

~~~bash
docker compose up -d
docker compose ps
~~~

The initial image downloads may take a few minutes.

## 4. Open the interfaces

| Service | Local URL | Purpose |
|---|---|---|
| Grafana | http://localhost:3000 | Dashboards |
| Prometheus | http://localhost:9090 | Targets, queries and rules |
| Alertmanager | http://localhost:9093 | Alert status and silences |

Grafana login username and password are set in .env. The pre-provisioned **Docker Host Overview** dashboard is available from the Dashboards section.

Prometheus targets should include Prometheus, Alertmanager, Node Exporter and cAdvisor. Open http://localhost:9090/targets and check that each target is UP.

## 5. Explore useful PromQL queries

CPU utilization percentage:

~~~promql
100 * (1 - avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])))
~~~

Memory utilization percentage:

~~~promql
100 * (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)
~~~

Filesystem utilization percentage:

~~~promql
100 * (1 - node_filesystem_avail_bytes{fstype!~"tmpfs|overlay|squashfs"} / node_filesystem_size_bytes{fstype!~"tmpfs|overlay|squashfs"})
~~~

Container CPU cores:

~~~promql
sum by (name) (rate(container_cpu_usage_seconds_total{image!=""}[5m]))
~~~

Container memory working set:

~~~promql
sum by (name) (container_memory_working_set_bytes{image!=""})
~~~

## 6. Alerts

The Prometheus rule file includes alerts for:
- Scrape target unavailable.
- Sustained high host CPU utilization.
- High host memory utilization.
- Low available filesystem space.
- Container OOM events, when reported by cAdvisor.

Prometheus sends firing alerts to Alertmanager. Alertmanager's initial receiver is intentionally local-only: alerts appear in its UI, but email, Slack, Teams or other external notifications are not configured. Add a real receiver only after creating and securely storing the required credentials; never commit webhook URLs or SMTP passwords.

## 7. Useful operations

Follow service logs:

~~~bash
docker compose logs -f prometheus
docker compose logs -f grafana
docker compose logs -f alertmanager
~~~

Restart a service:

~~~bash
docker compose restart prometheus
~~~

Stop the stack but keep metric history:

~~~bash
docker compose down
~~~

Start again:

~~~bash
docker compose up -d
~~~

Remove the stack and its stored metric history:

~~~bash
docker compose down -v
~~~

The last command permanently deletes the named Prometheus, Grafana and Alertmanager data volumes. Use it only when you intentionally want a clean reset.

## 8. Validation in GitHub Actions

The workflow validates:
- Prometheus configuration and alert rules with promtool.
- Alertmanager configuration with amtool.
- Grafana dashboard JSON and provisioning YAML.
- Docker Compose configuration.
- A live Compose smoke test: all services start and Prometheus reports four scrape targets.

The CI stack runs only on the GitHub Actions runner and is torn down when the job finishes. It does not deploy to AWS or publish monitoring data externally.

## Security notes

- Web interfaces bind to localhost only. Do not change them to 0.0.0.0 on an untrusted network.
- Node Exporter and cAdvisor are internal-only Compose services and should not be published to the internet.
- Docker socket and host filesystem access are sensitive. Review these permissions before running on a production machine.
- Use a unique Grafana password. Keep .env private.
- Do not commit credentials, tokens, private keys, or alerting webhook URLs.
- This is a learning and local observability stack, not a highly available or production-hardened deployment.

## License

MIT.
