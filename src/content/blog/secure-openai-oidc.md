---
title: "Secure OpenAI Access with OIDC in Kubernetes Using Envoy AI Gateway"
date: 2025-05-28
description: "Learn how to authenticate and control access to OpenAI’s API using OIDC and Envoy AI Gateway in a Kubernetes environment."
---

# Secure OpenAI Access with OIDC in Kubernetes Using Envoy AI Gateway

As generative AI becomes integral to internal tools and user-facing products, securing access to model APIs like OpenAI’s becomes critical. Without proper controls, teams risk:

- Unauthenticated access to high-cost APIs
- Loss of visibility into who is prompting models
- Data exposure from unverified users
- Difficulty enforcing usage limits or quotas

In this guide, you’ll learn how to use **Envoy AI Gateway** with **OIDC (OpenID Connect)** to secure access to OpenAI's API from within your Kubernetes environment.

## Why OIDC?

OIDC is a widely adopted identity layer built on OAuth 2.0. It allows users to authenticate using identity providers like:

- Google Workspace
- Okta
- Auth0
- Azure AD

With OIDC in place, you can:
- Enforce login for any prompt or API request
- Identify users via JWT claims
- Apply policy per identity or group
- Audit access logs for security and billing

## Architecture Overview

```
[ Dev Portal / CLI ]
        |
    [ OIDC Login ]
        |
[ Envoy AI Gateway ] --> [ OpenAI API ]
        |
     Policies
   (rate limits, auth)
```

## Step 1: Configure Envoy AI Gateway with OIDC

Example Gateway configuration using OIDC:

```yaml
authentication:
  - provider:
      name: oidc-auth
      issuer: https://auth.myorg.com
      clientId: envoy-gateway
      jwksUri: https://auth.myorg.com/.well-known/jwks.json
```

You can enforce this policy on any route (e.g., `/v1/chat/completions`) and require a valid bearer token signed by your IdP.

## Step 2: Map JWT Claims to Policy

With OIDC tokens available at the gateway, you can apply advanced controls:

- Rate limits by `email` or `sub` (subject ID)
- RBAC by `groups` claim
- Token quotas by `tenant_id` or `project_id`

Example policy snippet:

```yaml
rateLimits:
  - name: user-limit
    match:
      path: /v1/chat/completions
      claims:
        email: "*@myorg.com"
    requestsPerUnit: 100
    unit: minute
```

## Step 3: Enforce Logging and Auditing

Envoy AI Gateway integrates with Prometheus and OpenTelemetry to expose:

- Which user made each request
- Which endpoints were hit
- Request latency and failure rates
- JWT claims used to authorize traffic

## Bonus: Multi-Tenant Support

OIDC + Envoy AI Gateway makes it easy to isolate access by team, project, or product by parsing tenant IDs from JWT claims or request headers.

Example: apply a token budget per team:

```yaml
tokenBudgets:
  - tenant: marketing
    maxTokens: 100000
    period: month
```

## Conclusion

With just a few lines of configuration, you can enforce secure, authenticated, and observable access to OpenAI—without writing custom middleware or proxies.

Envoy AI Gateway makes it easy to:
- Enforce enterprise SSO and zero-trust policies
- Set per-user or per-group usage controls
- Observe LLM usage patterns at the edge

---

🔐 **Secure your OpenAI usage today**  
📘 [See the full OIDC configuration guide](#)  
🚀 [Deploy Envoy AI Gateway in your cluster](#)
