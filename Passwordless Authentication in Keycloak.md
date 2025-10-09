# Passwordless Authentication in Keycloak — Table of Contents

## 1. Overview
- What is passwordless authentication?
- Benefits for onboarding, security, and UX

## 2. Supported Methods
- Magic Link (email-based login)
- WebAuthn (biometric/hardware keys)
- OTP via Email/SMS
- Social Login (Google, Microsoft, etc.)

## 3. Magic Link Setup
- Installing `keycloak-magic-link` extension
- Customizing authentication flow
- SMTP configuration and email templates
- Required actions after login

## 4. WebAuthn Setup (Built-in)
- Enabling WebAuthn in realm settings
- Creating a custom browser flow
- Registering devices (YubiKey, fingerprint, etc.)
- Fallback strategies

## 5. OTP via Email/SMS (Custom SPI)
- Overview of SPI-based OTP authenticators
- Integration with Twilio, SendGrid, etc.
- Custom flow design and validation

## 6. Social Login as Passwordless
- Configuring identity providers
- Mapping federated identities
- Auto-linking and required actions

## 7. Testing & Debugging
- Token inspection ([jwt.ms](https://jwt.ms))
- Flow tracing and event logs
- Common errors and fixes

## 8. Modular Onboarding Scaffolds
- Markdown guides for each method
- Visual diagrams of login flows
- Sample curl/Postman flows
- Role-based access and conditional routing

## 9. Security & Compliance
- PKCE, DPoP, and token binding
- Audit logging and introspection
- Session expiration and revocation

## 10. Deployment & Scaling
- Docker and Kubernetes setup
- Realm export/import
- Multi-tenant and multi-realm strategies

---
