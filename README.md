# Get Hands-on with Cluster API - openSUSE 2023

This is a companion to the tutorial at openSUSE 2023.

## Pre-reqs

To complete the tutorial yourself you will need to install the following:

- [kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
- [clusterctl](https://github.com/kubernetes-sigs/cluster-api/releases/tag/v1.4.2)
- [Rancher Desktop](https://rancherdesktop.io/) or Docker Engine/Desktop
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- (optional)[k9s](https://k9scli.io/topics/install/)

## Tutorial

### Create a management cluster

- Download [kind-cluster-with-extramounts.yaml](./kind-cluster-with-extramounts.yaml)
- Run the following command in a terminal:

```bash
kind create cluster --config kind-cluster-with-extramounts.yaml
```

### Configure clusterctl

- Download [clusterctl.yaml](./clusterctl.yaml)
- Move it to the following directory: **~/.cluster-api/clusterctl.yaml**

### Install Cluster API (and providers)

We need to install the Docker and RKE2 providers.

- In a terminal run the following:

```bash
clusterctl init -i docker -b rke2 -c rke2
```

### Create workload/child cluster

- Open a terminal and run the following:

```bash
export CABPR_CP_REPLICAS=1
export CABPR_WK_REPLICAS=1
export KUBERNETES_VERSION=v1.26.4

clusterctl generate cluster test1 --from https://github.com/capi-samples/opensuse-23/blob/main/templates/online-default.yaml > cluster.yaml
```

- Apply the cluster yaml to your management cluster

```bash
kubectl apply -f cluster.yaml
```

- Watch the **Cluster** being provisioned:

```bash
kubectl get clusters -A -w
```

- Watch the **Machine** being provisioned:

```bash
kubectl get machines -A -w
```

### Connect to the new cluster

- In your terminal run the following to get the kubeconfig for the child cluster:

```bash
clusterctl get kubeconfig test1 > test.kubeconfig
```

- Then use it to explore the cluster:

```bash
kubectl --kubeconfig test.kubeconfig get pods -A
```

### Scaling nodes

Lets scale the control plane to 2 nodes:

- Open 2 terminals
- In the first terminal watch the nodes of the child cluster:

```bash
kubectl --kubeconfig test.kubeconfig get nodes -w
```

- In the second terminal edit the control plane:

```bash
kubectl edit rke2controlplane/test1-control-plane
```

- In the editor find **replicas** and change it to 2
- Save and exit from editor
- Watch the node be added to the child cluster

### Clear up

- Delete the child cluster by running:

```bash
kubectl delete cluster test1
```

-- Delete the management cluster by running:

```bash
kind delete cluster --name=capi-test
```
