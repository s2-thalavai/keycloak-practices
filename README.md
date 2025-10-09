# keycloak-practices

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
cd C:\Keycloak\bin
kc.bat bootstrap-admin --user admin --password admin
.\kc.bat start-dev
```

Access Keycloak at: http://localhost:8080

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

# OpenID Connect Setup & Browser Test in Keycloak

## Register a Client (OIDC Application)

Go to Clients → Create

Fill in:

- Client ID: my-react-app (or any name)

- Client Type: OpenID Connect

- Client Protocol: openid-connect

- **Root URL:** http://localhost:3000 (or your app’s URL)

- Click Save

### Configure Client Settings

In the client settings page:

- **Access Type:** public (for SPAs) or confidential (for backend apps)

- **Standard Flow Enabled:** ✅ (for Authorization Code Flow)

- **Implicit Flow Enabled:** ❌ (deprecated)

- **Direct Access Grants Enabled:** ✅ (optional for testing)

- **Valid Redirect URIs:** http://localhost:3000/*

- **Web Origins:** + (auto-fill from redirect URI)

- Click Save

### Add a Test User

- Go to Users → Add User

- Fill in username, email, etc.

- Click Save

Go to Credentials tab → Set password → Toggle "Temporary" to OFF → Save

### Test OpenID Connect Flow from Browser

Use this URL format to initiate login:

```
http://localhost:8180/realms/is_apps/protocol/openid-connect/auth?
client_id=external_users_client
&response_type=code
&scope=openid
&redirect_uri=http://localhost:3000/callback
&code_challenge=Xdhz6EE3Molukkc0IDRyQJCXxjPsvbJr8VqO2kHo2gU
&code_challenge_method=S256
```

<img width="1365" height="723" alt="image" src="https://github.com/user-attachments/assets/65e3e498-f54d-4e36-8d83-6df729556130" />

<img width="1366" height="727" alt="image" src="https://github.com/user-attachments/assets/5d9c4a1b-8a4d-431a-a0f9-5ebc81f1de37" />


```
[http://localhost:3000/callback?
session_state=e50b2571-b0c3-ecdb-68e9-57ab4e5a66f2
&iss=http%3A%2F%2Flocalhost%3A8180%2Frealms
%2Fis_apps
&code=62a9c087-ceef-6e95-474f-a3d00ddbe782.e50b2571-b0c3-ecdb-68e9-57ab4e5a66f2.fdf3e954-8038-4c46-9fef-ee46665c046b](http://localhost:3000/callback?session_state=42ff1da3-0ec9-9800-d32b-284f8c4a778d&iss=http%3A%2F%2Flocalhost%3A8180%2Frealms%2Fis_apps
&code=108997e7-918d-2a50-e54f-7e78cd4257fa.42ff1da3-0ec9-9800-d32b-284f8c4a778d.fdf3e954-8038-4c46-9fef-ee46665c046b)
```

Paste this into your browser. You’ll be redirected to Keycloak’s login screen. After login, Keycloak will redirect to your app with a code in the URL.

---

## Step-by-Step: Exchange Authorization Code for Tokens (PKCE)

### Step 1: Send a POST Request to Token Endpoint
Endpoint:

#### Example with curl
```bash
curl -X POST http://localhost:8180/realms/is_apps/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=authorization_code" \
  -d "client_id=external_users_client" \
  -d "code=108997e7-918d-2a50-e54f-7e78cd4257fa.42ff1da3-0ec9-9800-d32b-284f8c4a778d.fdf3e954-8038-4c46-9fef-ee46665c046b" \
  -d "redirect_uri=http://localhost:3000/callback" \
  -d 'code_verifier=acDUDY-d6GDppRvgyfm157qtbivtluO6HbenFUumRI1qqNXpQ9varQJQmaXDoXCFpkMHhV4At7yhJC-G1eVyrxEkBdKx1NUGOwAjky869_U7n07CiO0xeFOsoREONke7'
```

### Response

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJfSVlHdVBpeGJCbXBHWEZYNDhfNnJPLWlXSWNfY3U1cTg3WUlqSlE2cFpnIn0.eyJleHAiOjE3NTk5ODQyMDcsImlhdCI6MTc1OTk4MzkwNywiYXV0aF90aW1lIjoxNzU5OTgzNjUxLCJqdGkiOiJvbnJ0YWM6MDE5YTYxYzktODBhNC0yOTY2LTkwZTItNmQ3MWJiZmRlN2Q4IiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MTgwL3JlYWxtcy9pc19hcHBzIiwiYXVkIjoiYWNjb3VudCIsInN1YiI6Ijk4NzY4MjExLWQ1YWEtNDExOC1iZDlhLWZlZjIzZTAyZDY1NSIsInR5cCI6IkJlYXJlciIsImF6cCI6ImV4dGVybmFsX3VzZXJzX2NsaWVudCIsInNpZCI6IjQyZmYxZGEzLTBlYzktOTgwMC1kMzJiLTI4NGY4YzRhNzc4ZCIsImFjciI6IjAiLCJhbGxvd2VkLW9yaWdpbnMiOlsiaHR0cDovL2xvY2FsaG9zdDozMDAwIl0sInJlYWxtX2FjY2VzcyI6eyJyb2xlcyI6WyJvZmZsaW5lX2FjY2VzcyIsImRlZmF1bHQtcm9sZXMtaXNfYXBwcyIsInVtYV9hdXRob3JpemF0aW9uIl19LCJyZXNvdXJjZV9hY2Nlc3MiOnsiYWNjb3VudCI6eyJyb2xlcyI6WyJtYW5hZ2UtYWNjb3VudCIsIm1hbmFnZS1hY2NvdW50LWxpbmtzIiwidmlldy1wcm9maWxlIl19fSwic2NvcGUiOiJvcGVuaWQgZW1haWwgcHJvZmlsZSIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwibmFtZSI6IlNpdmFzYW5rYXIgVGhhbGF2YWkiLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJzaXZhc2Fua2FydGhhbGF2YWkiLCJnaXZlbl9uYW1lIjoiU2l2YXNhbmthciIsImZhbWlseV9uYW1lIjoiVGhhbGF2YWkiLCJlbWFpbCI6InNpdmFzYW5rYXIudGhhbGF2YWlAbWFybGFicy5jb20ifQ.WaRkEaAWJJY8grFX7K_-5hUqWhqUC9gAI1lAn7-Y5ksRfUyZqyDbvj18YTaAf1vZBkM4Afwz3RDPvd_JIfM2psI_2yhTUWJRTfJZ4cTgK_PuyNfyYfcfeIqA6UxxhW6QFAJpw7vBx7vrEZfBmWe8Rk1mZnysRPpTY8pN9YP4vhhxSVkP7f0_XNyXST2Cu35AlB9EcPL_2m4e-s4z3cg9ZsNUMnpaWVea4F1SqKj4wNWr1O16J3zukcKs1_c8OwcWRhLjzXXbANWa_Bf9hshKtZl55b6H8ZDn4JQMg8MunQv0aud_orGAUCQjlZfFGUP8mp0Paz9E-eoxEEXKZKuzOw",
  "expires_in": 300,
  "refresh_expires_in": 1800,
  "refresh_token": "eyJhbGciOiJIUzUxMiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICIzNWZjMzk0MS1kNDU3LTRhMDktYmQ2Mi01YzE1OWI1MzJiYzIifQ.eyJleHAiOjE3NTk5ODU3MDcsImlhdCI6MTc1OTk4MzkwNywianRpIjoiNzNjY2E3ZTUtODUwYS1mNmQ0LWE2YjAtZmI3MDQ5NTMwZGU4IiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MTgwL3JlYWxtcy9pc19hcHBzIiwiYXVkIjoiaHR0cDovL2xvY2FsaG9zdDo4MTgwL3JlYWxtcy9pc19hcHBzIiwic3ViIjoiOTg3NjgyMTEtZDVhYS00MTE4LWJkOWEtZmVmMjNlMDJkNjU1IiwidHlwIjoiUmVmcmVzaCIsImF6cCI6ImV4dGVybmFsX3VzZXJzX2NsaWVudCIsInNpZCI6IjQyZmYxZGEzLTBlYzktOTgwMC1kMzJiLTI4NGY4YzRhNzc4ZCIsInNjb3BlIjoib3BlbmlkIGVtYWlsIGJhc2ljIHByb2ZpbGUgcm9sZXMgYWNyIHdlYi1vcmlnaW5zIn0.m7p81cs69z8e_KfjLWaNE46IuxHc8GkEZWsvSpr8WlqYu--KNE25Sl2BhgFCH-ycJsZ0DNB5L_vrs5bwf8kIiw",
  "token_type": "Bearer",
  "id_token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJfSVlHdVBpeGJCbXBHWEZYNDhfNnJPLWlXSWNfY3U1cTg3WUlqSlE2cFpnIn0.eyJleHAiOjE3NTk5ODQyMDcsImlhdCI6MTc1OTk4MzkwNywiYXV0aF90aW1lIjoxNzU5OTgzNjUxLCJqdGkiOiI4Y2JkNDZhZS1lYWVlLWI4Y2QtNzRkNi01ODcyNjBhNzU4MTkiLCJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjgxODAvcmVhbG1zL2lzX2FwcHMiLCJhdWQiOiJleHRlcm5hbF91c2Vyc19jbGllbnQiLCJzdWIiOiI5ODc2ODIxMS1kNWFhLTQxMTgtYmQ5YS1mZWYyM2UwMmQ2NTUiLCJ0eXAiOiJJRCIsImF6cCI6ImV4dGVybmFsX3VzZXJzX2NsaWVudCIsInNpZCI6IjQyZmYxZGEzLTBlYzktOTgwMC1kMzJiLTI4NGY4YzRhNzc4ZCIsImF0X2hhc2giOiJ1MGc3d3piRXUzbUFDakxhUzA1eWJRIiwiYWNyIjoiMCIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwibmFtZSI6IlNpdmFzYW5rYXIgVGhhbGF2YWkiLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJzaXZhc2Fua2FydGhhbGF2YWkiLCJnaXZlbl9uYW1lIjoiU2l2YXNhbmthciIsImZhbWlseV9uYW1lIjoiVGhhbGF2YWkiLCJlbWFpbCI6InNpdmFzYW5rYXIudGhhbGF2YWlAbWFybGFicy5jb20ifQ.VfF_IGUmJllCnJz6Ch0MF7WB5pHpgJqmBovKmlA8Cun-Y3aLe7n7SxifXyD3NQ_uSmXVLJ-lWxKsYj_EP-Zz2LXIMPg-pBxE3dwdg5ILqRRUTa3wfV5iarG86bfi-zDUgKk1NgEiMlzQQPeM-9Knu5wm-vKr4VHoOXQ3GGsJKfR8M2-zGfxHWL3GWRop2vwH8yzNQL5lSpbua1wlugXfBqkNqfqm1R6REUxsiQgsyhr5aq5zEbE9-8JLVrv_d4YKgo6ZjB1RN9BoSh0ESsYVz6LPkP8mkw4jb6ZlHNpeU51bmT8BuvNZCw9siBmZjx7bvWB3fCqyjPxzgNNoOIk0zw",
  "not-before-policy": 0,
  "session_state": "42ff1da3-0ec9-9800-d32b-284f8c4a778d",
  "scope": "openid email profile"
}
```
---

# Token Verification in Azure APIM (Keycloak OIDC)

## Step 1: Get Keycloak’s OpenID Configuration

Keycloak exposes its metadata at:

```Code
http://<keycloak-host>/realms/<realm-name>/.well-known/openid-configuration
```

Example for your setup:

```Code
http://localhost:8180/realms/is_apps/.well-known/openid-configuration
```

From this, extract:

- issuer

- jwks_uri (for public key verification)

- token_endpoint

## Step 2: Configure Azure APIM JWT Validation Policy

In your API policy (e.g., inbound section), add:

```xml
<validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized">
  <openid-config url="http://localhost:8180/realms/is_apps/.well-known/openid-configuration" />
  <required-claims>
    <claim name="aud">
      <value>external_users_client</value>
    </claim>
    <claim name="iss">
      <value>http://localhost:8180/realms/is_apps</value>
    </claim>
  </required-claims>
</validate-jwt>
```

This ensures APIM validates the token’s signature, issuer, and audience.

## Step 3: Test with a Valid Token
Send a request to your APIM endpoint with:

```http
Authorization: Bearer <access_token>
```

If the token is valid and matches the claims, APIM will allow the request to proceed.

----

# Azure APIM Policy: Keycloak Token Verification

## 1. Validate JWT from Keycloak

Place this inside your API’s inbound policy:

```xml
<validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized">
  <openid-config url="http://localhost:8180/realms/is_apps/.well-known/openid-configuration" />
  <required-claims>
    <claim name="aud">
      <value>external_users_client</value>
    </claim>
    <claim name="iss">
      <value>http://localhost:8180/realms/is_apps</value>
    </claim>
  </required-claims>
</validate-jwt>
```

✅ This verifies the token’s signature, issuer, and audience using Keycloak’s JWKS.

## 2. Optional: Role-Based Access Control

Add this after validate-jwt to restrict access by role:

```xml
<choose>
  <when condition="@(context.Principal.Claims.Any(c => c.Type == 'realm_access.roles' && c.Value.Contains('admin')))">
    <!-- Allow access -->
  </when>
  <otherwise>
    <return-response>
      <set-status code="403" reason="Forbidden" />
      <set-body>Access denied: insufficient role</set-body>
    </return-response>
  </otherwise>
</choose>
```

Adjust admin to match your Keycloak role mappings.

## 3. Optional: Conditional Routing Based on Claims

```xml
<choose>
  <when condition="@(context.Principal.Claims.Any(c => c.Type == 'preferred_username' && c.Value == 'siva'))">
    <set-backend-service base-url="https://internal-api.siva.local" />
  </when>
  <otherwise>
    <set-backend-service base-url="https://public-api.default.local" />
  </otherwise>
</choose>
```

## 4. Testing Tips

Use access_token (not id_token) for API calls

Inspect tokens at jwt.ms

Confirm aud, iss, and exp match your policy
