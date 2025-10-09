# keycloak-practices

- OpenID Connect
- Magic Link

## Why Enterprises Need Keycloak: Strategic IAM Overview

### What Is Keycloak?

Keycloak is an **open-source Identity and Access Management (IAM)** system that provides:

- Authentication (who you are)

- Authorization (what you can do)

- User Federation (from LDAP, AD, social providers)

- Single Sign-On (SSO) across apps

It supports standard protocols like:

- OAuth 2.0

- OpenID Connect

- SAML 2.0

- LDAP

---

## Keycloak’s Role in Enterprise Architecture

| Capability                      | Why It Matters                                                                 |
|--------------------------------|---------------------------------------------------------------------------------|
| Centralized Identity Management| Avoids duplicating user logic across microservices and apps                    |
| SSO Across Apps                | Users log in once, access many apps securely                                   |
| Identity Brokering             | Bridges your app to external IdPs (e.g., Google, Azure AD, partner orgs)       |
| Social Login Support           | Enables login via Google, Facebook, Apple, etc.                                |
| LDAP Integration               | Syncs users from Active Directory or other LDAP servers                        |
| Admin Console                  | UI for managing users, roles, groups, and protocol configs                     |
| Token Management               | Rotate signing keys, revoke sessions, enforce policies                         |
| Multi-Factor Authentication (MFA)| Built-in support for stronger security                                       |
| Fine-Grained Authorization     | Define access rules per endpoint (Keycloak-specific feature)                   |

---

### Why Not Let Apps Handle Identity?

- Security risks from inconsistent implementations

- Hard to manage user lifecycle across apps

- No centralized audit trail or compliance strategy

- Difficult to support external users or federated identities

### Deployment Flexibility

- Works across clouds (Azure, AWS, GCP) or on-prem data centers

- **Language-agnostic:** Java, Node.js, Python, etc.

- **UI-agnostic:** React, Angular, Struts, etc.

---

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
$env:KC_ADMIN_PASSWORD = "admin"
cd C:\keycloak\keycloak-26.4.0\bin
.\kc.bat bootstrap-admin user --username admin --password:env KC_ADMIN_PASSWORD
.\kc.bat start-dev --http-port=8180
```

Access Keycloak at: http://localhost:8180

<img width="1206" height="679" alt="image" src="https://github.com/user-attachments/assets/b2a32bf3-e190-4d18-bd01-c04cf7276edc" />


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

---

# Keycloak Multi-Tenancy with Organizations (Post Keycloak 26+)

## Why Use Organizations?

| Feature                  | Multiple Realms         | Organizations (Single Realm)     |
|--------------------------|-------------------------|----------------------------------|
| Isolation                | Strong (per realm)      | Logical (per org)               |
| Resource Efficiency      | High overhead           | Lightweight, scalable           |
| Shared Config Extensions | Hard to maintain        | Centralized and reusable        |
| Token Customization      | Realm-specific          | Org-specific claims supported   |
| SaaS Onboarding          | Complex                 | Streamlined via identity-first  |

---

## Setup Guide

### 1. Enable Organizations
- Navigate to **Realm Settings → Organizations**
- Toggle `Enabled` to `true`

### 2. Create Organizations
- Go to **Organizations → Create**
- Define:
  - **Name**: e.g., `AcmeCorp`
  - **Alias**: used in login URLs
  - **Redirect URI**: post-login destination

### 3. Invite Users
- Use **Organization Invitations** to onboard users
- Assign roles and groups per organization

### 4. Customize Login Flow
- Enable **Identity-First Login**:
  - Users enter email → Keycloak detects org → routes to org-specific flow
- Customize themes or flows per org if needed

### 5. Token Customization
- Add org-specific claims via **Protocol Mappers**
- Example: `"org_id": "AcmeCorp"` in access token

---

## Example Login Flow

```plaintext
1. User enters email → user@acme.com
2. Keycloak detects `AcmeCorp` via domain mapping
3. Routes to AcmeCorp login flow
4. Token includes org-specific claims
```

---

## Audit & Compliance Tips

- Enable event logging per organization

- Use custom attributes for org metadata

- Externalize org configs via REST API for CI/CD

---

## Keycloak Admin Console Setup

### Accessing the Console
- URL: `http://<host>:8080/auth`
- Default realm: `master`
- Default admin user: `admin`
- Default password: `admin` (change immediately)

### Key Concepts
| Term         | Description |
|--------------|-------------|
| Realm        | Logical tenant for users, clients, roles |
| Client       | Application registered with Keycloak |
| Role         | Permission grouping |
| Group        | User grouping |
| Identity Provider | External source of user identities |

---

### Common Tasks

- Create a new realm
- Add users manually or via import
- Configure password policies
- Assign roles and groups

## Identity Brokering with Keycloak

### Use Case

Allow users to log in via external IdPs (Google, Azure AD, etc.)

### Steps
1. Go to **Identity Providers** in your realm
2. Choose provider type (e.g., OpenID Connect, SAML)
3. Enter client ID, secret, and endpoints
4. Enable **Account Linking** if needed
5. Test login flow via `/auth/realms/<realm>/account`

<img width="1350" height="652" alt="image" src="https://github.com/user-attachments/assets/06b9f69d-fc1d-469b-8f5f-bf5e9ce13e6b" />

<img width="1347" height="649" alt="image" src="https://github.com/user-attachments/assets/56e6b345-585a-4fcf-9a2f-2a7b2cbb375e" />

### Visual Flow
- App → Keycloak → External IdP → Keycloak → App
- Use curved arrows to show redirect-based login

---

## Spring Boot + Keycloak Integration

### Dependencies

```xml
<dependency>
  <groupId>org.keycloak</groupId>
  <artifactId>keycloak-spring-boot-starter</artifactId>
</dependency>
```

### Configuration

```yaml
keycloak:
  realm: demo
  auth-server-url: http://localhost:8080/auth
  resource: spring-app
  credentials:
    secret: <client-secret>
```

### Securing Endpoints

```java
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends KeycloakWebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .antMatchers("/admin/**").hasRole("admin")
        .anyRequest().permitAll();
  }
}
```

---


## OpenID Connect vs SAML: Authentication Showdown

| Feature                      | OpenID Connect (OIDC)                          | SAML 2.0                                      |
|-----------------------------|-----------------------------------------------|-----------------------------------------------|
| Protocol Type               | RESTful, JSON-based                           | XML-based                                     |
| Transport Format            | JSON over HTTP                                | XML over HTTP (often via POST or Redirect)    |
| Token Format                | JWT (JSON Web Token)                          | SAML Assertions (XML)                         |
| Mobile & SPA Friendly       | ✅ Excellent for mobile apps & SPAs           | 🚫 Not ideal for mobile or JavaScript clients |
| Modern Web Integration      | ✅ Built for OAuth flows                      | 🚫 Legacy browser-based flows                 |
| Ease of Implementation      | ✅ Lightweight, developer-friendly            | ⚠️ Verbose, XML-heavy                         |
| Identity Federation         | ✅ Supported via Identity Brokering           | ✅ Supported                                   |
| SSO Support                 | ✅ Yes                                        | ✅ Yes                                         |
| Standard Adoption           | Widely used in modern apps (Google, Azure AD) | Common in legacy enterprise apps              |
| Security Features           | Strong with PKCE, nonce, scopes               | Strong but harder to implement securely       |

---

## Recommendation

- Use **OpenID Connect** if you're building modern web apps, mobile apps, SPAs, or integrating with OAuth-based ecosystems.

- Use SAML if you're integrating with legacy enterprise systems or existing SAML-based IdPs.
  
---

