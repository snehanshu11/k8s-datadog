
Install datadog agent using helm chart
```
eksctl get cluster
 eksctl utils write-kubeconfig --cluster=<name>
 kubectl config current-context 
 eksctl create cluster --name kube-cluster --nodegroup-name ng-default --version 1.28 --node-type t2.micro --nodes 6 --region us-west-2
```

```
helm version
helm repo add datadog https://helm.datadoghq.com
helm repo update
```

```
datadog:
  clusterName: k8s-cluster-agent
  apm:
    portEnabled: true
  logs:
    enabled: true
    containerCollectAll: true
  processAgent:
    processCollection: true
  kubelet:
    tlsVerify: false
```
```
helm install datadog-agent \
  -f custom-values.yaml \
  --set datadog.apiKey=$DD_API_KEY \
  datadog/datadog
```

```
kubectl get secret datadog-agent \
  --template='{{index .data "api-key"}}' | base64 -d
```
```
kubectl get daemonset datadog-agent
kubectl get pods \
  -l app=datadog-agent \
  -o custom-columns=NAME:.metadata.name,NODE:.spec.nodeName
kubectl get nodes kubernetes -o jsonpath="{.spec.taints}"|jq
```
```
datadog:
  clusterName: k8s-cluster-agent
  apm:
    portEnabled: true
  logs:
    enabled: true
    containerCollectAll: true
  processAgent:
    processCollection: true
  kubelet:
    tlsVerify: false
agents:
  tolerations:
    - key: node-role.kubernetes.io/control-plane
      effect: NoSchedule
```

```
helm upgrade datadog-agent \
  -f custom-values.yaml \
  --set datadog.apiKey=$DD_API_KEY \
  datadog/datadog
```

```
kubectl get daemonset datadog-agent
kubectl get pods \
  -l app=datadog-agent \
  -o custom-columns=NAME:.metadata.name,NODE:.spec.nodeName
```
```
kubectl exec -ti $( \
  kubectl get pods \
  -l app=datadog-agent \
  --field-selector spec.nodeName=worker \
  -o name \
  ) -- agent status

kubectl exec -ti $( \
  kubectl get pods \
  -l app=datadog-agent \
  --field-selector spec.nodeName=kubernetes \
  -o name \
  ) -- agent status
```

```
kubectl get deployments
kubectl get pods \
  -l app=datadog-agent-cluster-agent \
  -o custom-columns=NAME:.metadata.name,NODE:.spec.nodeName
 kubectl exec -ti $( \
   kubectl get pods \
   -l app=datadog-agent-cluster-agent \
   -o name \
   ) -- agent status
```
```
eksctl delete cluster --name=kube-cluster --region=us-west-2
```
