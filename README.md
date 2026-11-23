# rdv

Medical appointment booking service built to practice **observability** and
**SRE** on a real-world application: instrumentation, SLOs, alerting,
resilience, and progressive delivery.

> 🚧 Project under active development. The repository currently contains a
> Datadog Agent and a diagnostics container. The booking application and its
> database are planned.

## Objectives

- Build a containerized web service along with its database.
- Instrument it end-to-end: metrics, logs, and traces.
- Make it reliable and prove it: SLOs, load testing, and chaos engineering.

## Getting started

The stack uses Docker Engine and the Docker Compose plugin. It is intended for
Linux hosts: the containers share host PID and network namespaces, and the Agent
reads Linux host paths. With Docker Desktop, host diagnostics refer to its Linux
VM rather than the macOS or Windows host; host networking also depends on Docker
Desktop configuration.

Create your local configuration if you do not already have one:

```bash
cp .env.example .env
```

Fill in all required variables in `.env`:

| Variable | Description |
| --- | --- |
| `DD_API_KEY` | Datadog API key from Organization Settings > API Keys. |
| `DD_SITE` | Site of your Datadog organization, for example `datadoghq.eu`. |
| `DD_ENV` | Environment tag, for example `dev`, `staging`, or `prod`. |
| `DD_HOSTNAME` | Hostname used to identify the host in Datadog. |

The `.env` file is ignored by Git. Keep API keys out of committed files.
Compose requires all four variables, including when running only the `tools`
service.

Validate the configuration, then start the Agent:

```bash
docker compose config --quiet
docker compose up -d datadog-agent
```

Check container health, Agent status, and logs:

```bash
docker compose ps
docker compose exec datadog-agent agent status
docker compose logs --tail=100 datadog-agent
```

Stop and remove the stack containers:

```bash
docker compose down
```

## Datadog Agent

The `datadog-agent` service runs Agent **7.85.5** with host networking and host
PID and cgroup namespaces. It is part of the default Compose stack and restarts
unless explicitly stopped.

The current configuration enables:

- Host and container monitoring through the mounted Docker socket, `/proc`,
  and cgroup filesystem.
- Process collection for Datadog Live Processes, including host processes such
  as `systemd`, `dockerd`, and `containerd` when present.
- Network connection state collection through
  `datadog/conf.d/network.d/conf.yaml`, mounted into the Agent with
  `collect_connection_state: true`.
- Tags `project:rdv` and `team:infra`, plus the configured environment and hostname.

APM is disabled (`DD_APM_ENABLED=false`). Application tracing and log collection
are not configured yet.

Inspect the network check directly:

```bash
docker compose exec datadog-agent agent check network
```

## Host diagnostics

The `tools` service is built from Ubuntu 24.04 and includes `procps`, `sysstat`,
`stress-ng`, `iproute2`, `dnsutils`, `curl`, `tcpdump`, and CA certificates.
It uses the `tools` profile, so it does not start during a normal
`docker compose up -d`.

Build or rebuild the image after changing its Dockerfile:

```bash
docker compose --profile tools build tools
```

Open a temporary interactive shell:

```bash
docker compose --profile tools run --rm tools
```

### System activity

Display five samples at one-second intervals. The first sample primarily
reports averages since boot:

```bash
docker compose --profile tools run --rm tools vmstat 1 5
```

List processes by resident memory usage:

```bash
docker compose --profile tools run --rm tools ps aux --sort=-rss
```

Generate CPU load with one worker for 30 seconds:

```bash
docker compose --profile tools run --rm tools stress-ng --cpu 1 --timeout 30s --metrics-brief
```

Use `stress-ng` to observe how the system and Datadog respond to load. Keep the
load and duration limited to avoid exhausting host resources.

### Network diagnostics

The container shares the host network namespace and has the `NET_ADMIN`
capability. Inspect interfaces, routes, and TCP/UDP sockets:

```bash
docker compose --profile tools run --rm tools ip address show
docker compose --profile tools run --rm tools ip route show
docker compose --profile tools run --rm tools ss -tunap
```

Check DNS resolution and HTTPS connectivity:

```bash
docker compose --profile tools run --rm tools dig example.com
docker compose --profile tools run --rm tools curl -I --max-time 10 https://example.com
```

Capture up to 10 DNS packets, stopping after 15 seconds if fewer arrive:

```bash
docker compose --profile tools run --rm tools timeout 15s tcpdump -ni any -c 10 'port 53'
```

A `timeout` exit code of `124` means the time limit was reached. Generate DNS
traffic from another terminal if needed. Each temporary container is removed
when its command finishes.

## Host access

Both services use host PID and network namespaces, reducing container isolation.
The `tools` service can modify host network settings through `NET_ADMIN`.
Access to `docker.sock` can provide extensive control over the Docker host,
even when the socket is mounted read-only. Run this stack on a host you control
and keep diagnostic commands scoped to the intended workload.
