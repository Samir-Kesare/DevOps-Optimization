apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: adapter-service-keda-scaledobject
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: adapter-service
  pollingInterval: 5
  cooldownPeriod: 300
  minReplicaCount: 3
  maxReplicaCount: 5
  fallback:
    failureThreshold: 3
    replicas: 3
  advanced:
    horizontalPodAutoscalerConfig:
      behavior:
        scaleUp:
          stabilizationWindowSeconds: 120
          policies:
            - type: Pods
              value: 1
              periodSeconds: 60
        scaleDown:
          stabilizationWindowSeconds: 120
          policies:
            - type: Percent
              value: 100
              periodSeconds: 60
  triggers:
    - type: prometheus
      metadata:
        serverAddress: http://rancher-monitoring-devops-prometheus.cattle-monitoring-system:9090
        metricName: container_network_receive_bytes_total
        query: sum(irate(container_network_receive_bytes_total{cluster="",namespace=~"default"}[5m]) * on (namespace,pod) group_left(workload,workload_type) namespace_workload_pod:kube_pod_owner:relabel{cluster="",namespace=~"default", workload=~"adapter-service", workload_type="deployment"})
        threshold: '1100'
        ignoreNullValues: "true"
