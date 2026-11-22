# rdv

Medical appointment booking service built to practice **observability** and
**SRE** on a real-world application: instrumentation, SLOs, alerting,
resilience, and progressive delivery.

> 🚧 Project under active development. This README describes what currently exists.

## Objectives

- Build a containerized web service along with its database.
- Instrument it end-to-end: metrics, logs, and traces.
- Make it reliable and prove it: SLOs, load testing, and chaos engineering.

## Datadog Agent

When enabled, the Datadog Agent collects and forwards telemetry to Datadog.
Configure it with a Datadog API key and site, and keep credentials in environment
variables rather than committing them to the repository.

The Agent can collect host processes and display them in Datadog Live Processes,
including `systemd`, `dockerd`, and `containerd`.

**Security:** Access to `docker.sock` can provide extensive control over the
Docker host, even when the socket is mounted read-only.

**Security:** `pid: host` reduces container isolation by exposing host processes;
depending on its privileges, the container may also be able to interact with them.

## Host diagnostics

The `tools` service provides commands to inspect the host. It uses a Compose
profile, so it does not start during a normal `docker compose up -d`.

Run a temporary container to inspect system activity:

```bash
docker compose --profile tools run --rm tools vmstat 1 5
```

This command displays five samples at one-second intervals. The first line
primarily reports averages since boot.

List processes by resident memory usage:

```bash
docker compose --profile tools run --rm tools ps aux --sort=-rss
```

Generate CPU load with one worker for 30 seconds:

```bash
docker compose --profile tools run --rm tools stress-ng --cpu 1 --timeout 30s --metrics-brief
```

Use `stress-ng` to test how the system and Datadog respond to load. Excessive
stress can slow down the VM and its services or exhaust resources, so keep
the load and duration limited.

The temporary container is removed when each command finishes.