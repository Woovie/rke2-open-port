# PostgreSQL Deployment in RKE2 (Or, how to open arbitrary ports with RKE2 Ingress Controller)

## Overview

This repository contains Kubernetes manifests and configuration files for deploying PostgreSQL in a Rancher RKE2 environment. The setup includes secure credential management and external access configuration through the built-in RKE2 ingress controller.

## Important considerations

From the included `ingress-nginx-tcp-config.yaml`, we can see this data:

```yaml
spec:
  valuesContent: |-
    tcp:
      "5432": "postgre/postgre:5432" 
```

The first port, represented as the key, is the incoming port to this machine. The value is in the format `<namespace>/<service>:<port>` and will direct traffic to that service. The service should then use a selector to find the right deployed application. In my case, my PostgreSQL objects all share a simple `app=postgresql` keyval to use for lookups.

This functionality also works for UDP ports, see the [Ingress Nginx](https://kubernetes.github.io/ingress-nginx/user-guide/exposing-tcp-udp-services/) documentation.

## Install

1. Install PostgreSQL with your values of choice, I've given a bare minimum example showing how the port can be configured

```bash
helm install postgresql bitnami/postgresql --create-namespace -n <namespace> --values values.yaml
```

2. Install the service

```bash
kubectl apply -f service.yaml
```

3. Apply the TCP configuration to expose PostgreSQL through the ingress controller

```bash
kubectl apply -f ingress-nginx-tcp-config.yaml
```

4. Test accessing one of your public endpoint IPs of your cluster with the `psql` client and you should get prompted for a password:

```bash
➜ jbanasik@woovie-desktop ~/projects/cursor-testing/k8s/postgresql git:(main) ✗ psql -h postgres.woovie.net
Password for user jbanasik:
```
