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

- **Standard Flow Enabled:** (for Authorization Code Flow)

- **Implicit Flow Enabled:** (deprecated)

- **Direct Access Grants Enabled:** (optional for testing)

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

---

# Full inbound Policy Block

````xml
<inbound>
  <!-- Validate JWT from Keycloak -->
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

  <!-- Role-Based Access Control -->
  <choose>
    <when condition="@(context.Principal.Claims.Any(c => c.Type == 'realm_access.roles' && c.Value.Contains('admin')))">
      <!-- Access granted to 'admin' role -->
    </when>
    <otherwise>
      <return-response>
        <set-status code="403" reason="Forbidden" />
        <set-body>Access denied: insufficient role</set-body>
      </return-response>
    </otherwise>
  </choose>

  <!-- Conditional Routing Based on Claims -->
  <choose>
    <when condition="@(context.Principal.Claims.Any(c => c.Type == 'preferred_username' && c.Value == 'siva'))">
      <set-backend-service base-url="https://internal-api.siva.local" />
    </when>
    <otherwise>
      <set-backend-service base-url="https://public-api.default.local" />
    </otherwise>
  </choose>
</inbound>
```
---


