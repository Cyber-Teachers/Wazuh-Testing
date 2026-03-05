# Wazuh Lab: Vulnerable Machine + Single-Node Stack

Lab to test Wazuh with a vulnerable agent (official Wazuh agent image) and the official Wazuh single-node stack (indexer, manager, dashboard).

## Prerequisites

- Docker and Docker Compose
- Git

## One-time setup

### 1. Generate indexer certificates

You **must** run this step **before** the first `docker compose up`. From the **project root**:

```bash
docker compose -f generate-indexer-certs.yml run --rm generator
```

If you already ran `docker compose up` without generating certs, Docker may have created directories where certificate files should be. Remove the certs directory and generate again:

```bash
rm -rf config/wazuh_indexer_ssl_certs
docker compose -f generate-indexer-certs.yml run --rm generator
```

Then start the lab with `docker compose up --build`.

## Run the lab

From the **project root**:

```bash
docker compose up --build
```

(The vulnerable-agent uses the official `wazuh/wazuh-agent:5.0.0` image.)

## Access

### Wazuh Dashboard

- **URL**: https://localhost/
- **Username**: `admin`
- **Password**: `SecretPassword`

(You may need to accept the self-signed certificate in the browser.)

### Debian SSH machine

A minimal Debian container with SSH for testing (e.g. Wazuh can detect logins). From the host:

```bash
ssh -p 2222 debian@localhost
```

**Password**: `debian`

## Credentials summary

| Resource        | Username | Password      |
|-----------------|----------|---------------|
| Wazuh Dashboard | admin    | SecretPassword |
| Debian SSH      | debian   | debian        |

## Layout

- `docker-compose.yml` – Wazuh single-node stack, `vulnerable-agent`, and `debian-ssh`.
- `debian-ssh/Dockerfile` – Minimal Debian with OpenSSH server (user `debian` / password `debian`).
- `agent/Dockerfile.amd64` – Optional custom agent image with Wazuh agent + OpenSSH server.

All services share the same Docker Compose network and resolve each other by service name (`wazuh.manager`, `vulnerable-agent`, `debian-ssh`).

## Troubleshooting

**"not a directory: Are you trying to mount a directory onto a file"** – The certificate files did not exist when you first ran `docker compose up`, so Docker created directories instead. Remove the certs directory and regenerate, then start again:

```bash
docker compose down
rm -rf config/wazuh_indexer_ssl_certs
docker compose -f generate-indexer-certs.yml run --rm generator
docker compose up --build
```
