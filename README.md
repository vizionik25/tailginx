# tailginx

A lightweight, containerized ingress gateway utilizing **Nginx** and **Tailscale Funnel** to securely expose local or containerized applications to the public internet over HTTPS.

By default, this project is configured to proxy incoming HTTPS traffic to a local instance of **LMStudio** running on the host machine (`http://host.docker.internal:1234`).

## Architecture

```
[ Public Internet ]
        │ (HTTPS)
        ▼
[ Tailscale Funnel ] (via egress_tailscale container)
        │ (HTTP, Internal Docker Network)
        ▼
[ Nginx Ingress Proxy ] (via ingress_nginx container)
        │ (HTTP, proxy_pass)
        ▼
[ Local Application ] (host.docker.internal:YOUR_PORT)
```

## Prerequisites

1. **Docker and Docker Compose**: Installed on the host machine.
2. **Tailscale Account**: Active Tailscale account with a registered machine or client.
3. **Tailscale Configuration**:
   - **MagicDNS**: Must be enabled in the Tailscale Admin Console under **DNS**.
   - **HTTPS Certificates**: Must be enabled in the Tailscale Admin Console under **DNS**.
   - **Tailscale Funnel**: Must be allowed in your Tailscale Access Control Policy (ACLs). Ensure the ACL has a policy allowing funnel access:
     ```json
     "nodeAttrs": [
         {
             "target": ["*"],
             "attr": ["funnel"],
         }
     ]
     ```

## Quick Start

1. **Clone and Navigate** to this project directory:
   ```bash
   cd tailginx
   ```

2. **Create Environment File**:
   Copy `.env.example` to `.env`:
   ```bash
   cp .env.example .env
   ```

3. **Configure Tailscale Auth Key**:
   - Generate a new Auth Key in the **Tailscale Admin Console** (Settings > Keys > Generate auth key).
   - *Tip: Marking the key as ephemeral is recommended so the device is automatically cleaned up when the container is stopped.*
   - Open `.env` and replace `your-tsauth-key` with your generated Tailscale Auth Key:
     ```env
     TS_AUTHKEY=tskey-auth-XXXXXX...
     ```

4. **Run the Containers**:
   Start the stack using Docker Compose:
   ```bash
   docker compose up -d
   ```

5. **Get Your Public URL**:
   Check the Tailscale container logs to find your automatically assigned public URL:
   ```bash
   docker compose logs tailscale
   ```
   Look for lines containing your node's public hostname or the TLS certificate provisioning messages.

## Custom Configuration

### Forwarding to a different Application
By default, `tailginx` forwards requests to `host.docker.internal:1234` (LMStudio). If you want to expose a different app:

1. Open [nginx.conf](file:///Users/vizionik/tailginx/nginx.conf).
2. Modify the `proxy_pass` directive to target your app's port or address:
   ```nginx
   # Example: Forwarding to a local React app running on port 3000
   proxy_pass http://host.docker.internal:3000;
   ```
3. Restart the containers to apply changes:
   ```bash
   docker compose restart nginx
   ```

### Exposing a container on the same Docker Network
If your application runs inside another Docker container:
1. Define a shared network in `docker-compose.yml`.
2. Change the `proxy_pass` in `nginx.conf` to target the container name and port:
   ```nginx
   # Example: Exposing a container named 'webapp' on port 8080
   proxy_pass http://webapp:8080;
   ```

## Security Considerations

- **Exposing Services**: Tailscale Funnel makes your service available to the **entire public internet**. Make sure the application you are exposing has its own authentication/authorization mechanisms if it contains sensitive data.
- **Environment Files**: The `.env` file contains your `TS_AUTHKEY` secrets. It is added to `.gitignore` to prevent accidental commits to public repositories.

## Troubleshooting

### Connection refused to `host.docker.internal` (Linux Hosts)
Unlike macOS and Windows, Docker on Linux does not resolve `host.docker.internal` by default.
To enable it on Linux, open `docker-compose.yml` and add `extra_hosts` configuration to the `nginx` service:

```yaml
services:
  nginx:
    image: nginx:latest
    container_name: ingress_nginx
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    extra_hosts:
      - "host.docker.internal:host-gateway"
```

### Check logs
If your application is not accessible, check the logs for both containers:
```bash
docker compose logs -f
```

### Verifying Funnel Status
You can run the tailscale CLI directly inside the container to check status:
```bash
docker compose exec tailscale tailscale status
docker compose exec tailscale tailscale funnel status
```
