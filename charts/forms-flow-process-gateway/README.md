# Formsflow.ai Process Gateway

formsflow.ai Process Gateway provides integration, adapter routing, and offline process support for formsflow.ai.

## Introduction

This chart bootstraps a forms-flow-process-gateway deployment on a [Kubernetes](https://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

## Installing the Chart

To install the chart with the release name `forms-flow-process-gateway`:

```console
helm install forms-flow-process-gateway forms-flow-process-gateway
```

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example:

```console
helm install forms-flow-process-gateway forms-flow-process-gateway --set ingress.ingressClassName=INGRESS_CLASS --set ingress.hostname=HOSTNAME
```

> **Tip**: List all releases using `helm list`

### Resource requests and limits

Forms-flow-process-gateway charts allow setting resource requests and limits for all containers inside the chart deployment. These are inside the `resources` value (check parameter table).

```yaml
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 256Mi
```

### Change Forms-flow-process-gateway version

To modify the Forms-flow-process-gateway version used in this chart, specify a valid image tag using the `image.tag` parameter. For example, `image.tag=v8.4.2`.

```yaml
image:
  registry: docker.io
  repository: formsflow/forms-flow-process-gateway-ee
  tag: v8.4.2
```

## Sidecar Configuration

The chart includes an unprivileged Nginx sidecar container to route and proxy requests to the Process Gateway service.

## API Path

The `forms-flow-process-gateway` is accessible at the `/gatewayapi/*` route.

## Parameters

| Parameter                                             | Description                                                          | Default Value                                |
|-------------------------------------------------------|----------------------------------------------------------------------|----------------------------------------------|
| `replicaCount`                                       | Number of replicas for the deployment                                | `1`                                          |
| `image.registry`                                     | Docker registry for the image                                        | `docker.io`                                  |
| `image.repository`                                   | Repository for the image                                             | `formsflow/forms-flow-process-gateway-ee`    |
| `image.tag`                                          | Image tag                                                            | `v8.4.2`                                     |
| `image.pullSecrets`                                  | Secrets for pulling images from private registries                   | `[forms-flow-ai-auth]`                       |
| `nameOverride`                                       | Override name for the deployment                                     | `""`                                         |
| `fullnameOverride`                                   | Override full name for the deployment                                | `""`                                         |
| `serviceAccount.create`                              | Create a service account                                             | `true`                                       |
| `ingress.enabled`                                    | Enable ingress for the service                                       | `true`                                       |
| `ingress.ingressClassName`                           | Ingress class (e.g. alb)                                             | `"alb"`                                      |
| `ingress.path`                                       | Path for ingress routing                                             | `"/gatewayapi/*"`                            |
| `ingress.servicePort`                                | Ingress backend service port                                         | `8080`                                       |
| `containerPorts.http`                                | Port for the Process Gateway application                             | `8000`                                       |
| `service.ports`                                      | Service ports exposing app and nginx sidecar                         | `8000 (http), 8080 (nginx)`                  |
| `gateway.workers`                                    | Process gateway workers                                              | `"1"`                                        |
| `gateway.gatewayAdapters`                            | Gateway adapters                                                     | `"camunda"`                                  |
| `formsflow.configmap`                                | Name of shared formsflow.ai configmap                                | `forms-flow-ai`                              |
| `formsflow.secret`                                   | Name of shared formsflow.ai secret                                   | `forms-flow-ai`                              |
