### Optional
##### Alias
```shell
echo "alias k=kubectl" >> ~/.bashrc && source ~/.bashrc
```

### Prerequisites

##### Install Required Packages

```shell
sudo snap install kubectl --classic
sudo snap install helm --classic
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER && newgrp docker
```

##### Install minikube
##### Linux (ARM64)
```shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-arm64
sudo install minikube-linux-arm64 /usr/local/bin/minikube
```

##### Linux (x86_64)
```shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```



##### Start minikube
```shell
minikube start
```


## Deploying ARC
##### Install cert-manager
```shell
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.yaml
```

##### Kubectl deployment
```shell
kubectl create -f https://github.com/actions/actions-runner-controller/releases/download/v0.27.4/actions-runner-controller.yaml
```


##### Authenticate (GitHub Personal Access Token)
```shell
kubectl create secret generic controller-manager -n actions-runner-system --from-literal=github_token={GITHUB_PERSONAL_ACCESS_TOKEN}
```


## Configuration (Manual)

##### Create config
```shell
nano runnerdeployment.yaml
```
##### Sample config
```yaml
apiVersion: actions.summerwind.dev/v1alpha1
kind: RunnerDeployment
metadata:
  name: actions-runner
spec:
  template:
    spec:
      repository: swils23/actions-experiment
---
apiVersion: actions.summerwind.dev/v1alpha1
kind: HorizontalRunnerAutoscaler
metadata:
  name: actions-runner-autoscaler
spec:
  scaleDownDelaySecondsAfterScaleOut: 60
  scaleTargetRef:
    kind: RunnerDeployment
    name: actions-runner
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: TotalNumberOfQueuedAndInProgressWorkflowRuns
    repositoryNames:
    # A repository name is the REPO part of `github.com/OWNER/REPO` = swils23/actions-experiment
    - actions-experiment
```

##### Apply config
```shell
kubectl apply -f runnerdeployment.yaml
```



## Configuration (Automatic)

Copies the sample config [runnerdeployment.yaml](https://raw.githubusercontent.com/swils23/actions-runner-controller/main/runnerdeployment.yaml) to a file called `runnerdeployment.yaml` in the current directory, then applies it to the cluster.

```shell
curl -LO https://raw.githubusercontent.com/swils23/actions-runner-controller/main/runnerdeployment.yaml && kubectl apply -f runnerdeployment.yaml
```
