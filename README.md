# Microsim – Kubernetes Observability Demo

This repository demonstrates a complete **Kubernetes-based observability setup** using
**Microsim**, **OpenTelemetry**, **Jaeger**, and **Grafana**, secured with **TLS via cert-manager and Let’s Encrypt (Cloudflare DNS-01)**.

The goal of this project is **not** to serve an application UI, but to:
- generate distributed traces
- expose observability tooling
- demonstrate Kubernetes, ingress, and TLS best practices

---

## Architecture Overview

- **Microsim** runs as a Kubernetes **Job**
  - Generates synthetic distributed traces
  - Exports traces via OpenTelemetry
- **Jaeger** collects and visualizes traces
- **Grafana** visualizes metrics and traces
- **NGINX Ingress Controller** exposes services
- **cert-manager + Let’s Encrypt (Cloudflare)** provides TLS certificates
- A **static landing page** explains the setup

---

## Public Endpoints

| Service | URL |
|------|-----|
| Landing page | https://microsim.dev.piratesec.com |
| Grafana | https://grafana.microsim.dev.piratesec.com |
| Jaeger | https://traces.microsim.dev.piratesec.com |

All endpoints are secured with valid **Let’s Encrypt TLS certificates**.

---

## Landing Page

The root endpoint serves a **static HTML landing page** explaining the system purpose.

The landing page is deployed via:
- `ConfigMap`
- `nginx:alpine`
- Kubernetes `Deployment`
- Kubernetes `Service`
- NGINX Ingress

No application traffic is served from this endpoint by design.

---

## Microsim

Microsim is an **open-source microservices simulator** by Yuri Shkuro.

- Docker image: `yurishkuro/microsim`
- Runs as a **Kubernetes Job**
- Generates traces continuously
- Exports traces using **OpenTelemetry OTLP**

Microsim **does not expose an HTTP UI**, which is why a landing page is used instead.

---

## Observability Stack

### Jaeger
- Receives OTLP traces from Microsim
- Used to inspect:
  - distributed traces
  - service dependencies
  - latency
- Intentionally exposed **without authentication** (demo environment)

### Grafana
- Used for visualization
- Configured to read from Jaeger
- Authentication enabled (default Grafana auth)

---

## TLS & Certificates

TLS is handled using:

- **cert-manager**
- **Let’s Encrypt**
- **Cloudflare DNS-01 challenge**
- **ClusterIssuer**

Certificates are issued automatically for:
- microsim.dev.piratesec.com
- grafana.microsim.dev.piratesec.com
- traces.microsim.dev.piratesec.com

---

## Security Headers (Important)

All endpoints are protected by **TLS** and served exclusively over **HTTPS**.

However, HTTP security headers (such as `Content-Security-Policy`, `X-Frame-Options`,
`Strict-Transport-Security`, etc.) are **not injected at the ingress level**.

### Reason

The Kubernetes cluster’s **NGINX Ingress Controller is configured with snippet directives disabled**.
This is a **cluster-wide security policy** that prevents the use of annotations such as:

- `nginx.ingress.kubernetes.io/configuration-snippet`
- `nginx.ingress.kubernetes.io/server-snippet`

As a result, it is **not possible** to inject custom security headers via ingress annotations in this environment.

### Implication

- Security scanning tools (e.g. securityheaders.com) report **D-grade scores**
- This is a **known and documented limitation**, not a misconfiguration
- In clusters where ingress snippets or custom NGINX templates are allowed, all endpoints can be hardened to **A+**

This setup reflects real-world constraints commonly found in **managed or shared Kubernetes environments**.

---

## Repository Structure

```
├── docs
├── k8s
│ ├── namespaces
│ │ └── namespaces.yaml
│ ├── ingress
│ │ ├── clusterissuer-letsencrypt.yaml
│ │ ├── microsim-ingress.yaml
│ │ └── observability-ingress.yaml
│ ├── microsim
│ │ ├── microsim.yaml
│ │ ├── microsim-job.yaml
│ │ ├── service.yaml
│ │ ├── landing.yaml
│ │ └── landing-configmap.yaml
│ └── observability
│ ├── grafana.yaml
│ ├── jaeger.yaml
│ └── observability-certificate.yaml
└── README.md
```
## Deployment Order (Important)

Apply manifests in this order:
```
kubectl apply -f k8s/namespaces/
kubectl apply -f k8s/ingress/clusterissuer-letsencrypt.yaml
kubectl apply -f k8s/observability/
kubectl apply -f k8s/ingress/observability-ingress.yaml
kubectl apply -f k8s/microsim/
kubectl apply -f k8s/ingress/microsim-ingress.yaml
```

## Notes

- This is a **demo / lab environment**
- Jaeger is intentionally public
- Grafana authentication is enabled
- No secrets or credentials are stored in Git
- All TLS certificates are managed automatically
