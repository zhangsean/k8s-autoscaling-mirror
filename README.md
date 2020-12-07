# k8s-autoscaling-mirror

Mirror of `k8s.gcr.io/autoscaling/`.

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

## More info

[Vertical Pod Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)
