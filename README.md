# keycloak-practices

## Step-by-Step: Keycloak + PostgreSQL on Windows Server

### Prerequisites

- Windows Server 2019 or later

- Admin privileges

- Java 17+ (JDK)

- PostgreSQL installed and running

- Keycloak distribution (ZIP)

---
### Install Java 17+

Download and install OpenJDK 17 Set environment variables:

```powershell
$env:JAVA_HOME = "C:\Program Files\Eclipse Adoptium\jdk-17"
$env:PATH += ";$env:JAVA_HOME\bin"
```
---

### Install PostgreSQL

Download from https://www.postgresql.org/download/windows/ During setup:

- Create user: keycloak

- Create database: keycloak

- Set password: secret

Enable TCP/IP and confirm port 5432 is open.

---
### Download and Configure Keycloak

- Download ZIP: https://www.keycloak.org/downloads

- Extract to C:\Keycloak

Configure DB connection

Edit C:\Keycloak\conf\keycloak.conf:

```ini
db=postgres
db-url=jdbc:postgresql://localhost:5432/keycloak
db-username=keycloak
db-password=secret
hostname-strict=false
```

### Start Keycloak

Open PowerShell:

```powershell
cd C:\Keycloak\bin
.\kc.bat start-dev
```

Access Keycloak at: http://localhost:8080

### Create Admin User

On first launch, Keycloak prompts for:

```bash
KEYCLOAK_ADMIN=admin
KEYCLOAK_ADMIN_PASSWORD=admin
```

You can also set these as environment variables before starting.

## Optional Enhancements

- Reverse proxy with IIS or NGINX for TLS

- Run as Windows Service using NSSM or Task Scheduler

- Externalize config for auditability (keycloak.conf)

- Enable metrics with Prometheus exporter

# Reverse Proxy with NGINX + TLS for Keycloak

## 1. Install NGINX
On Ubuntu/Debian:

```bash
sudo apt update
sudo apt install nginx
```

On Windows Server: Use NGINX for Windows or run via WSL.

## 2. Obtain TLS Certificates

**Option A:** Let’s Encrypt (Linux only)

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

**Option B:** Manual Certificates

Place your certs in /etc/nginx/ssl/:

    fullchain.pem → Certificate
    
    privkey.pem → Private key

## 3. Configure NGINX Reverse Proxy
Edit /etc/nginx/sites-available/keycloak.conf:

```nginx
server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name your-domain.com;

    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;

    location / {
        proxy_pass http://localhost:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Enable and restart:

```bash
sudo ln -s /etc/nginx/sites-available/keycloak.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

## 4. Update Keycloak Config

In keycloak.conf or startup flags:

```ini
hostname=your-domain.com
hostname-strict=true
https-port=443
```

## Optional Enhancements

- Enable HTTP/2 for performance: listen 443 ssl http2;

- Add HSTS headers for security

- Enable access logs for audit trail

- Use Docker or Kubernetes ingress for containerized setups

---

## Multi-Tenancy in Keycloak: Two Approaches

### 1. Multiple Realms (Traditional Approach)

- Each tenant gets its own realm.

- Pros: Strong isolation, separate configurations.

- Cons: High resource usage, complex management at scale.

### 2️. Single Realm + Organizations (Recommended for SaaS)

Introduced in Keycloak 26+, this feature enables multi-tenancy within a single realm using organizational boundaries.

**Key Features:**

- Create and manage multiple organizations inside one realm.

- Invite and onboard users per organization.

- Identity-first login and organization-specific authentication flows.

- Propagate organization-specific claims in tokens for authorization.

- Centralized management with reduced overhead.

### Implementation Highlights

- Enable “Organizations” in realm settings via Admin Console.

- Create organizations with unique aliases and redirect URLs.

- Assign users, roles, and identity providers per organization.

- Customize login flows and token claims based on organization context.

### Why Organizations > Multiple Realms?

- **Resource Efficiency:** Avoids memory and CPU overhead of multiple realms.

- **Centralized Management:** Easier to maintain shared configs and extensions.

- **Scalability:** More sustainable for large-scale SaaS platforms.
