Started by user 23203615
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /app/jenkins/workspace/Infra-automation/Development/DevOps_Automation/keda_toggle_test
[Pipeline] {
[Pipeline] withEnv
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Extract Jenkins User)
[Pipeline] cleanWs
[WS-CLEANUP] Deleting project workspace...
[WS-CLEANUP] Deferred wipeout is used...
[WS-CLEANUP] done
[Pipeline] script
[Pipeline] {
[Pipeline] sh
+ echo 'Reading log: /app/jenkins/jobs/Infra-automation/jobs/Development/jobs/DevOps_Automation/jobs/keda_toggle_test/builds/60/log'
Reading log: /app/jenkins/jobs/Infra-automation/jobs/Development/jobs/DevOps_Automation/jobs/keda_toggle_test/builds/60/log
+ cat /app/jenkins/jobs/Infra-automation/jobs/Development/jobs/DevOps_Automation/jobs/keda_toggle_test/builds/60/log
++ cat log.txt
++ grep -i Started
++ sed 's@Started by user @@'
+ user_id='23203615'
+ echo '23203615'
[Pipeline] readFile
[Pipeline] echo
✅ Clean USER_ID: 23203615
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Set Kubernetes Master IP)
[Pipeline] script
[Pipeline] {
[Pipeline] echo
🌐 Selected Kubernetes master IP: 172.27.98.166
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Keda Toggle)
[Pipeline] script
[Pipeline] {
[Pipeline] echo
🔁 Applying KEDA toggle config
[Pipeline] sh
+ git clone https://bitbucket.airtel.africa/bitbucket/scm/atlaf/keda.git keda-repo
Cloning into 'keda-repo'...
[Pipeline] dir
Running in /app/jenkins/workspace/Infra-automation/Development/DevOps_Automation/keda_toggle_test/keda-repo
[Pipeline] {
[Pipeline] pwd
[Pipeline] echo
🔍 Original YAML:
[Pipeline] sh
+ cat ug-keda-scaler/default/collective_keda_scaledobject.yaml

apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: collective-keda-scaledobject
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: collective
  pollingInterval: 5
  cooldownPeriod: 300
  minReplicaCount: 6
  maxReplicaCount: 8
  fallback:
    failureThreshold: 3
    replicas: 6
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
        query: sum(irate(container_network_receive_bytes_total{cluster="",namespace=~"default"}[5m]) * on (namespace,pod) group_left(workload,workload_type) namespace_workload_pod:kube_pod_owner:relabel{cluster="",namespace=~"default", workload=~"collective", workload_type="deployment"})
        threshold: '1870000'
        ignoreNullValues: "true"
[Pipeline] sh
+ sed -i 's/^  minReplicaCount: .*/  minReplicaCount: 1/' ug-keda-scaler/default/collective_keda_scaledobject.yaml
+ sed -i 's/^  maxReplicaCount: .*/  maxReplicaCount: 5/' ug-keda-scaler/default/collective_keda_scaledobject.yaml
+ sed -i 's/^  replicas: .*/  replicas: 1/' ug-keda-scaler/default/collective_keda_scaledobject.yaml
/app/jenkins/workspace/Infra-automation/Development/DevOps_Automation/keda_toggle_test/keda-repo@tmp/durable-7176fbd9/script.sh.copy: line 5: unexpected EOF while looking for matching `''
[Pipeline] }
[Pipeline] // dir
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Declarative: Post Actions)
[Pipeline] cleanWs
[WS-CLEANUP] Deleting project workspace...
[WS-CLEANUP] Deferred wipeout is used...
[WS-CLEANUP] done
[Pipeline] echo
❌ KEDA pipeline failed.
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
ERROR: script returned exit code 2
/app/jenkins/jobs/Infra-automation/jobs/Development/jobs/DevOps_Automation/workspace/keda_toggle_test@tmp/jfrog/60/.jfrog deleted
Finished: FAILURE
