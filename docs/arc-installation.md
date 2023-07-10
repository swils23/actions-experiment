## Before you begin

##### Helpful aliases (***Optional***)
```shell
grep -qxF 'alias k="kubectl"' ~/.bashrc || echo 'alias k="kubectl"' >> ~/.bashrc
grep -qxF 'alias kg="kubectl get"' ~/.bashrc || echo 'alias kg="kubectl get"' >> ~/.bashrc
source ~/.bashrc
```

## Environment Setup

#### Install Required Packages

```shell
sudo snap install kubectl --classic
sudo snap install helm --classic
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER && newgrp docker
```

#### Install minikube

<details>
  <summary>Linux (ARM64)</summary>
    <blockquote>
        curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-arm64 && sudo install minikube-linux-arm64 /usr/local/bin/minikube
    </blockquote>
</details>
<details>
  <summary>Linux (x86_64)</summary>
    <blockquote>
        curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && sudo install minikube-linux-amd64 /usr/local/bin/minikube
    </blockquote>
</details>

<sub>Additional installation options can be found [here](https://minikube.sigs.k8s.io/docs/start/).</sub>


#### Start minikube
```shell
minikube start
```

## Deploying ARC
#### Install cert-manager
```shell
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.yaml
```

#### Kubectl deployment
```shell
kubectl create -f https://github.com/actions/actions-runner-controller/releases/download/v0.27.4/actions-runner-controller.yaml
```

#### Authenticate (via GitHub Personal Access Token)
```shell
kubectl create secret generic controller-manager -n actions-runner-system --from-literal=github_token={GITHUB_PERSONAL_ACCESS_TOKEN}
```
<sub>Instructions for creating a GitHub Personal Access Token can be found
[here](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens).</sub>


## Controller Deployment (Manual)

Or skip to automatic deployment by clicking [here](#controller-deployment-automatic)!

#### Create config
```shell
nano runnerdeployment.yaml
```
Documentation and more examples [here](https://github.com/actions/actions-runner-controller/blob/master/docs/automatically-scaling-runners.md).

#### Example config
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

#### Apply config
```shell
kubectl apply -f runnerdeployment.yaml
```


## Controller Deployment (Automatic)

This downloads the sample [runnerdeployment.yaml](https://raw.githubusercontent.com/swils23/utils/main/actions-experiment/runnerdeployment.yaml) then applies it to the cluster.

```shell
curl -LO https://raw.githubusercontent.com/swils23/utils/main/actions-experiment/runnerdeployment.yaml && kubectl apply -f runnerdeployment.yaml
```

## Additional Controller Configuration (***Optional***)

### Working with the controller deployment

```shell
kubectl get deployment actions-runner-controller -n actions-runner-system -o yaml > arc-controller.yaml # Get current config
vi arc-controller.yaml # Edit config, feel free to use nano... vim is scary :)
kubectl apply -f arc-controller.yaml -n actions-runner-system # Apply changes
```
<sub>**Note:** Controller launch args are found under `spec.template.spec.containers.args` in the the `arc-controller.yaml` file.</sub>

### Some helpful controller launch args

```yaml
# Interval between each check for new workflows/jobs
# NOTE: If this is too low, you will get rate limited by GitHub, if it is too high
#       you will have a greater delay auto-scaling up/down for new jobs.
--sync-period=10s # default 1m
```

View current rate limit with 

```shell
curl -i -u <GITHUB_USERNAME>:<GITHUB_PERSONAL_ACCESS_TOKEN> https://api.github.com/rate_limit
```

