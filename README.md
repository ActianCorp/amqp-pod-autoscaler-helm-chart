# amqp-pod-autoscaler-helm-chart

## Introduction

This chart installs an application responsible for managing the number of worker replicas that are running according
to the destination configurations for Integration Manager

Add the following repository to helm:  
```helm repo add actian-datacloud https://s3.amazonaws.com/actian-datacloud-helm-charts```

To install the chart, first configure a override-values.yaml file specific for your environment:  
```helm install actian-datacloud/amqp-pod-autoscaler -f override-values.yaml```

## Configuration

The following table lists the configurable parameters of the amqp-pod-autoscaler chart and their default values.
  
| Parameter | Description | Default|
| -----  | ----- | ------|
| `imagePullSecrets` | name of Secret resource containing private registry credentials | [] |
| `image` | image to pull | actian/k8s-rabbit-pod-autoscaler:2.0.5.270 |
| `imagePullPolicy` | when to pull image | IfNotPresent |
| `affinity` | node/pod affinities | {} |
| `livenessProbe` | pod liveness probe | { "initialDelaySeconds": 120, "periodSeconds": 10, "timeoutSeconds": 5, "successThreshhold": 1, "failureThreshhold": 3, "httpGet": { "scheme": "HTTP", "path": "/health", "port": 8080 }} |
| `revisionHistoryLimit` | the number of old history to retain to allow rollback | 10 |
| `existingAutoscalerSecret` | existing autoscaler secret to use | nil |
| `existingRabbitSecret` | existing  rabbitmq secret to use | nil |
| `replicaCount` | number of pods to run | 1 |
| `pdb` | pod disruption budget | {} |
| `nodeSelector` | set nodeSelector | {} |
| `extraLabels` | additional labels to add | {} |
| `extraConfig` | additional properties to include in the config map | {} |
| `podAnnotations` | pod annotations | {} |
| `resources` | set resource limits | {} |
| `amqp.managementUrl` | rabbitmq management url ex: http://rabbitmq-rabbitmq-ha.default.svc.cluster.local:15672 | nil |
| `amqp.host` | rabbitmq host | nil |
| `amqp.username` | rabbitmq username | nil |
| `amqp.password` | rabbitmq password | nil |
| `worker.deploymentLabel` | identifier label used for the worker deployment(s) | integration-manager-worker |
| `autoscale.monitoringRate` | the rate at which to check queue depth in ms | 30000 |
| `actian.im.url` | endpoint for the integration manager APIs | nil |
| `actian.im.clientId` | client id associated with this application | nil |
| `actian.im.clientSecret` | client secret associated with this application | nil |
| `actian.im.deviceCode` | device code associated with this application | nil |
| `actian.im.userCode` | user code associated with this application | nil |
