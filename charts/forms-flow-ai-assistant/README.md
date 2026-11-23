# Formsflow.ai AI Assistant

formsflow.ai AI Assistant provides intelligent assistance, retrieval-augmented generation (RAG), and vector search capabilities for formsflow.ai.

## Introduction

This chart bootstraps a forms-flow-ai-assistant deployment on a [Kubernetes](https://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

## Installing the Chart

To install the chart with the release name `forms-flow-ai-assistant`:

```console
helm install forms-flow-ai-assistant forms-flow-ai-assistant
```

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example:

```console
helm install forms-flow-ai-assistant forms-flow-ai-assistant --set ingress.ingressClassName=INGRESS_CLASS --set ingress.hostname=HOSTNAME
```

> **Tip**: List all releases using `helm list`

### Resource requests and limits

Forms-flow-ai-assistant charts allow setting resource requests and limits for all containers inside the chart deployment. These are inside the `resources` value (check parameter table).

```yaml
resources:
  limits:
    cpu: "2"
    memory: 2Gi
  requests:
    cpu: 250m
    memory: 512Mi
```

### Change Forms-flow-ai-assistant version

To modify the Forms-flow-ai-assistant version used in this chart, specify a valid image tag using the `image.tag` parameter. For example, `image.tag=v8.4.0-alpha`.

```yaml
image:
  registry: docker.io
  repository: formsflow/forms-flow-ai-assistant-ee
  tag: v8.4.0-alpha
```

## Sidecar Configuration

The chart includes an unprivileged Nginx sidecar container to route and proxy requests to the AI Assistant service.

## API Path

The `forms-flow-ai-assistant` is accessible at the `/assistant` route. Ensure that all ingress configurations and client requests reference this path.

For example:

```
https://<HOSTNAME>/assistant/
```

## Parameters

| Parameter                                             | Description                                                          | Default Value                                |
|-------------------------------------------------------|----------------------------------------------------------------------|----------------------------------------------|
| `replicaCount`                                       | Number of replicas for the deployment                                | `1`                                          |
| `image.registry`                                     | Docker registry for the image                                        | `docker.io`                                  |
| `image.repository`                                   | Repository for the image                                             | `formsflow/forms-flow-ai-assistant-ee`       |
| `image.tag`                                          | Image tag                                                            | `v8.4.0-alpha`                               |
| `image.pullSecrets`                                  | Secrets for pulling images from private registries                   | `[forms-flow-ai-auth]`                       |
| `nameOverride`                                       | Override name for the deployment                                     | `""`                                         |
| `fullnameOverride`                                   | Override full name for the deployment                                | `""`                                         |
| `serviceAccount.create`                              | Create a service account                                             | `true`                                       |
| `serviceAccount.annotations`                          | Annotations for the service account (e.g. AWS IRSA role ARN)         | `{"eks.amazonaws.com/role-arn": "..."}`       |
| `ingress.enabled`                                    | Enable ingress for the service                                       | `true`                                       |
| `ingress.ingressClassName`                           | Ingress class (e.g. alb)                                             | `"alb"`                                      |
| `ingress.path`                                       | Path for ingress routing                                             | `"/assistant/*"`                             |
| `ingress.servicePort`                                | Ingress backend service port                                         | `8080`                                       |
| `containerPorts.http`                                | Port for the AI Assistant application                                | `5007`                                       |
| `service.ports`                                      | Service ports exposing nginx sidecar                                 | `[ { name: http, port: 8080, ... } ]`        |
| `database.url`                                       | PostgreSQL connection string with pgvector support                   | `"postgresql://..."`                         |
| `s3.bucket`                                          | AWS S3 bucket for document assets                                    | `"formsflow-multitenant-assets-dev"`         |
| `s3.region`                                          | AWS S3 region                                                        | `"ca-central-1"`                             |
| `backend.retrievalProfile`                           | Retrieval profile backend                                            | `"vector"`                                   |
| `backend.vectorStoreBackend`                         | Vector store backend                                                 | `"pgvector"`                                 |
| `backend.objectStoreBackend`                         | Object store backend                                                 | `"s3"`                                       |
| `backend.jobQueueBackend`                            | Job queue backend                                                    | `"pgqueuer"`                                 |
| `formsflow.configmap`                                | Name of shared formsflow.ai configmap                                | `forms-flow-ai`                              |
| `formsflow.secret`                                   | Name of shared formsflow.ai secret                                   | `forms-flow-ai`                              |
