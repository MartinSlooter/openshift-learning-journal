# 2026-07-02 - Understanding OpenShift Routes

## Goal

Learn how Routes differ from Kubernetes Ingress.

## What I learned

- Routes are OpenShift's native HTTP exposure mechanism.
- TLS termination can happen at the Route.
- Routes are managed by the OpenShift Router.

## Commands

```bash
oc get routes
oc expose service myapp
