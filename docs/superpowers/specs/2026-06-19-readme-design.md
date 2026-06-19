# Spec: README.md for tailginx

## Overview
Create a comprehensive, professional `README.md` file in the root of the project to document the setup, configuration, and troubleshooting steps for the `tailginx` project.

## Goals
Provide users with a clear, step-by-step guide to run `tailginx`, exposing a local or containerized application (such as LMStudio) to the public internet securely using Tailscale Funnel and Nginx.

## Structure of the README.md

1. **Header & Description**: High-level explanation of `tailginx`.
2. **Architecture**: Brief overview of the data flow:
   `Public Internet -> Tailscale Funnel (HTTPS) -> Tailscale Container -> Nginx Container (HTTP) -> Target Application (host.docker.internal)`
3. **Prerequisites**:
   - Docker and Docker Compose installed.
   - A Tailscale account.
   - Tailscale features enabled:
     - **MagicDNS** (under DNS settings).
     - **HTTPS Certificates** (under DNS settings).
     - **Funnel** (under Access Control / ACLs).
4. **Quick Start**:
   - Copy `.env.example` to `.env`.
   - Obtain a Tailscale Auth Key (recommended: ephemeral, labeled/tagged, or reusable) and add it to `.env` as `TS_AUTHKEY`.
   - Start the stack: `docker compose up -d`.
5. **Configuration Guides**:
   - **Nginx configuration (`nginx.conf`)**: How to change the `proxy_pass` destination from the default `http://host.docker.internal:1234` to any other local port or Docker container.
   - **Tailscale serve configuration (`tailscale-serve.json`)**: Explaining the mapping of TCP port 443 with HTTPS and the use of the `${TS_CERT_DOMAIN}` placeholder.
6. **Security Considerations**:
   - Storing credentials securely (avoiding committing `.env`).
   - Understanding that Tailscale Funnel makes the endpoint accessible to the public internet.
7. **Troubleshooting**:
   - Checking logs with `docker compose logs`.
   - Linux-specific `host.docker.internal` setup (adding `extra_hosts` to `docker-compose.yml` if running on Linux, since it isn't resolved by default like on Mac/Windows).
   - Verifying Tailscale status inside the container.
