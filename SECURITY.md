# Security Policy

## Scope

This repository is a local learning stack for Prometheus, Grafana, Alertmanager, Node Exporter and cAdvisor. It is not a production-hardened monitoring deployment.

## Reporting

Do not publish passwords, webhook URLs, tokens or other private credentials in public issues. Use GitHub's private vulnerability reporting feature when available.

## Safe use

- Copy .env.example to .env and set a unique Grafana password before running the stack.
- Never commit .env or real notification credentials.
- Keep Grafana, Prometheus and Alertmanager bound to localhost unless you deliberately configure TLS, authentication and network access controls.
- Do not publish Node Exporter, cAdvisor or Docker socket endpoints to the internet.
- Understand that Node Exporter and cAdvisor require sensitive host filesystem/runtime access to collect host and container metrics.
- Review upstream release notes and update pinned images when security fixes are available.
- Treat the included alert receiver as local-only. Configure external notification channels with secrets stored outside Git.
