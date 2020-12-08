# k8s-autoscaling-mirror

Mirror of `k8s.gcr.io/autoscaling/`.

[![DockerHub Badge](http://dockeri.co/image/zhangsean/vpa-recommender)](https://hub.docker.com/r/zhangsean/vpa-recommender/)

gcr.io | docker hub
---|---
k8s.gcr.io/autoscaling/vpa-updater:0.9.0 | [zhangsean/vpa-updater:0.9.0](https://hub.docker.com/r/zhangsean/vpa-updater/)
k8s.gcr.io/autoscaling/vpa-recommender:0.9.0 | [zhangsean/vpa-recommender:0.9.0](https://hub.docker.com/r/zhangsean/vpa-recommender/)
k8s.gcr.io/autoscaling/vpa-admission-controller:0.9.0 | [zhangsean/vpa-admission-controller:0.9.0](https://hub.docker.com/r/zhangsean/vpa-admission-controller/)

## Usage

### Prerequisites

* `kubectl` should be connected to the cluster you want to install VPA in.
* The metrics server must be deployed in your cluster. Read more about [Metrics Server](https://github.com/kubernetes-incubator/metrics-server).
* If you already have another version of VPA installed in your cluster, you have to tear down the existing installation first with:

```sh
./hack/vpa-down.sh
```

### Install

```sh
# Set env REGISTRY and TAG
export REGISTRY=zhangsean
export TAG=0.9.0
# Get autoscaling scripts
git clone https://github.com/kubernetes/autoscaler.git
cd vertical-pod-autoscaler
# To print YAML contents with all resources that would be understood by kubectl diff|apply|... commands, you can use
./hack/vpa-process-yamls.sh print
# To install VPA
./hack/vpa-up.sh
# Check if all system components are running:
kubectl --namespace=kube-system get pods|grep vpa
# The above command should list 3 pods (recommender, updater and admission-controller) all in state Running.
# Check if the system components log any errors. For each of the pods returned by the previous command do:
kubectl --namespace=kube-system logs [pod name]| grep -e '^E[0-9]\{4\}'
# Check that the VPA Custom Resource Definition was created:
kubectl get customresourcedefinition|grep verticalpodautoscalers
```

### Quick start

A simple way to check if Vertical Pod Autoscaler is fully operational in your cluster is to create a sample deployment and a corresponding VPA config:

```sh
kubectl create -f examples/hamster.yaml
```

Example VPA configuration:

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind:       Deployment
    name:       my-app
  updatePolicy:
    updateMode: "Auto"
```

To see VPA config and current recommended resource requests run:

```sh
kubectl describe vpa my-app-vpa
```

### Tear down

> Note that if you stop running VPA in your cluster, the resource requests for the pods already modified by VPA will not change

Tear down VPA components:

```sh
./hack/vpa-down.sh
```

## Goldilocks

By using the kubernetes vertical-pod-autoscaler in recommendation mode, we can see a suggestion for resource requests on each of our apps. This tool creates a VPA for each deployment in a namespace and then queries them for information.

### Installation

```sh
helm repo add fairwinds-stable https://charts.fairwinds.com/stable
helm install goldilocks --namespace kube-system fairwinds-stable/goldilocks
# check pods status
kubectl get po -n kube-system | grep goldilocks

# Enable goldilocks vpa in namespace kube-system
kubectl label ns kube-system goldilocks.fairwinds.com/enabled=true
# After that you should start to see VPA objects in that namespace.
kubectl get vpa -n kube-system
# Get vpa recommended for nginx-ingress-controller
kubectl describe vpa -n kube-system nginx-ingress-controller
```

### Viewing the Dashboard

The default installation creates a ClusterIP service for the dashboard. You can access via ingress:

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: goldilocks-dashboard
  namespace: kube-system
spec:
  rules:
  - host: goldilocks.example.com
    http:
      paths:
      - backend:
          serviceName: goldilocks-dashboard
          servicePort: 8080
```

Then open your browser to http://goldilocks.example.com

Once your VPAs are in place, you'll see recommendations appear in the Goldilocks dashboard:

![Goldilocks Screenshot](https://github.com/FairwindsOps/goldilocks/blob/master/img/screenshot.png)

## More info

* [Vertical Pod Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)
* [Goldilocks](https://github.com/FairwindsOps/goldilocks)