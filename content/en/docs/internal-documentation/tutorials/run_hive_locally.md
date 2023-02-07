---
title: Run Hive Locally
linkTitle: Run Hive Locally
---

This guide describes how to deploy a Hive environment in your local machine
using [kind](https://kind.sigs.k8s.io/).

For the managed clusters, this guide covers both
[kind](https://kind.sigs.k8s.io/) clusters and
[CRC](https://developers.redhat.com/products/codeready-containers/overview)
clusters.

## Preparation

Setup your `GOPATH`. Add to your `~/.bashrc`:

```bash
export GOPATH=$HOME/go
export PATH=${PATH}:${GOPATH}/bin
```

Install `kind`:

```bash
~$ curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.10.0/kind-linux-amd64
~$ chmod +x ./kind
~$ mv ./kind ~/.local/bin/kind
```

> This guide was created using kind version: `kind v0.14.0 go1.18.2 linux/amd64`

Install the dependencies:

```bash
~$ GO111MODULE=on go get sigs.k8s.io/kustomize/kustomize/v3
~$ go get github.com/cloudflare/cfssl/cmd/cfssl
~$ go get github.com/cloudflare/cfssl/cmd/cfssljson
~$ go get -u github.com/openshift/imagebuilder/cmd/imagebuilder
```

Clone OLM and checkout the version:

```bash
~$ git clone git@github.com:operator-framework/operator-lifecycle-manager.git
~$ cd operator-lifecycle-manager
~/operator-lifecycle-manager$ git checkout -b v0.21.2 v0.21.2
~/operator-lifecycle-manager$ cd ..
```

Clone Hive and checkout the version:

```bash
~$ git clone git@github.com:openshift/hive.git
~$ cd hive
~/hive$ git checkout 56adaaacf5f8075e3ad0896dac35243a863ec07b
```

Edit the `hack/create-kind-cluster.sh`, adding the apiServerAddress pointing
to your local `docker0` bridge IP. This is needed so the Hive cluster, which
runs inside a docker container, can reach the managed cluster, running inside
another docker container:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  apiServerAddress: "172.17.0.1"  # docker0 bridge IP
containerdConfigPatches:
  - |-
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."localhost:${reg_port}"]
      endpoint = ["http://${reg_name}:${reg_port}"]
```

## Hive

Export the Hive kubeconfig filename (it will be created later):

```bash
~$ export KUBECONFIG=/tmp/hive.conf
```

Enter the `hive` directory:

```bash
~$ cd hive
```

Create the Hive cluster:

```bash
~/hive$ ./hack/create-kind-cluster.sh hive
Creating cluster "hive" ...
Creating cluster "hive" ...
 ✓ Ensuring node image (kindest/node:v1.24.0) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-hive"
You can now use your cluster with:

kubectl cluster-info --context kind-hive

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/

```

The `/tmp/hive.conf` file is created now. Checking the installation:

```bash
~/hive$ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED         STATUS         PORTS                                       NAMES
901f215229a4   kindest/node:v1.24.0   "/usr/local/bin/entr…"   2 minutes ago   Up 2 minutes   172.17.0.1:41299->6443/tcp                  hive-control-plane
0d4bf61da0a3   registry:2             "/entrypoint.sh /etc…"   3 hours ago     Up 3 hours     0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   kind-registry
```

```bash
~/hive$  kubectl cluster-info
Kubernetes control plane is running at https://172.17.0.1:41299
KubeDNS is running at https://172.17.0.1:41299/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

Build Hive and push the image to the local registry:

```bash
~/hive$ CGO_ENABLED=0 IMG=localhost:5000/hive:latest make docker-dev-push
```

Deploy Hive to the hive cluster:

```bash
~/hive$ IMG=localhost:5000/hive:latest make deploy
```

Because we are not running on OpenShift we must also create a secret with
certificates for the hiveadmission webhooks:

```bash
~/hive$ ./hack/hiveadmission-dev-cert.sh
```

If the hive cluster is using node image `kindest/node:v1.24.0` or later, you will
have to additionally run:

```bash
~/hive$ ./hack/create-service-account-secrets.sh
```

because starting in Kubernetes 1.24.0, secrets are no longer automatically generated
for service accounts.

Tip: if it fails, check `kubectl` version. The `Client` and `Server` versions
should be in sync:

```bash
~/hive$ kubectl version --short
Client Version: v1.24.0
Kustomize Version: v4.5.4
Server Version: v1.24.0
```

Checking the Hive pods:

```bash
~/hive$ kubectl get pods -n hive
NAME                                READY   STATUS    RESTARTS   AGE
hive-clustersync-0                  1/1     Running   0          26m
hive-controllers-79bbbc7f98-q9pxm   1/1     Running   0          26m
hive-operator-69c4649b96-wmd79      1/1     Running   0          26m
hiveadmission-6697d9df99-jdl4l      1/1     Running   0          26m
hiveadmission-6697d9df99-s9pv9      1/1     Running   0          26m
```

## Managed Cluster - Kind

Open a new terminal.

Export the Hive kubeconfig filename (it will be  created later):

```bash
~$ export KUBECONFIG=/tmp/cluster1.conf
```

Enter the `hive` directory:

```bash
~$ cd hive
~/hive$
```

Create the managed cluster:

```bash
~/hive$ ./hack/create-kind-cluster.sh cluster1
Creating cluster "cluster1" ...
 ✓ Ensuring node image (kindest/node:v1.24.0) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-cluster1"
You can now use your cluster with:

kubectl cluster-info --context kind-cluster1

Have a nice day! 👋
 😊
```

The `/tmp/cluster1.conf` file is created now. Checking the installation:

```bash
~/hive$ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED             STATUS             PORTS                                       NAMES
267fa20f4a0f   kindest/node:v1.24.0   "/usr/local/bin/entr…"   2 minutes ago       Up 2 minutes       172.17.0.1:40431->6443/tcp                  cluster1-control-plane
901f215229a4   kindest/node:v1.24.0   "/usr/local/bin/entr…"   About an hour ago   Up About an hour   172.17.0.1:41299->6443/tcp                  hive-control-plane
0d4bf61da0a3   registry:2             "/entrypoint.sh /etc…"   5 hours ago         Up 5 hours         0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   kind-registry
```

```bash
~/hive$ kubectl cluster-info
Kubernetes control plane is running at https://172.17.0.1:40431
KubeDNS is running at https://172.17.0.1:40431/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

Before we install OLM, we have to edit the install scripts to use `cluster1`. Go into `scripts/build_local.sh`
and replace

```bash
  if [[ ${#CLUSTERS[@]} == 1 ]]; then
    KIND_FLAGS="--name ${CLUSTERS[0]}"
    echo 'Use cluster ${CLUSTERS[0]}'
  fi
```

with

```bash
  KIND_FLAGS="--name cluster1"
```

Now enter the OLM directory:

```bash
~/hive$ cd ../operator-lifecycle-manager/
~/operator-lifecycle-manager$
```

Install the CRDs and OLM using:

```bash
~/operator-lifecycle-manager$ make run-local
```

OLM pods should be running now:

```bash
~/git/operator-lifecycle-manager$ kubectl get pods -n olm
NAME                                READY   STATUS    RESTARTS   AGE
catalog-operator-54bbdffc6b-hf8rz   1/1     Running   0          87s
olm-operator-6bfbd74fb8-cjl4b       1/1     Running   0          87s
operatorhubio-catalog-gdk2d         1/1     Running   0          48s
packageserver-5d67bbc56b-6vxqb      1/1     Running   0          46s
```

With the cluster1 installed, let's create a `ClusterDeployment` in Hive
pointing to it.

Export the Hive kubeconfig filename and enter the `hive` directory (or just
switch to the first terminal, the one used to deploy Hive):

```bash
~$ export KUBECONFIG=/tmp/hive.conf
~$ cd hive
```

Attention: hiveutil wants you to have default credentials in ~/.aws/credentials
You can fake them like this:

```bash
~/hive$ cat ~/.aws/credentials
[default]
aws_access_key_id = foo
aws_secret_access_key = bar
```

Because Hive will not provision that cluster, we have can use the `hiveutil`
to adopt the cluster:

```bash
~/hive$ bin/hiveutil create-cluster \
--base-domain=new-installer.openshift.com kind-cluster1  \
--adopt --adopt-admin-kubeconfig=/tmp/cluster1.conf \
--adopt-infra-id=fakeinfra \
--adopt-cluster-id=fakeid
```

Checking the `ClusterDeployment`:

```bash
~/hive$ kubectl get clusterdeployment
NAME            PLATFORM   REGION      CLUSTERTYPE   INSTALLED   INFRAID   VERSION   POWERSTATE   AGE
kind-cluster1   aws        us-east-1                 true        infra1                           48s
```

Checking the `ClusterDeployment` status:

```bash
~/hive$ kubectl get clusterdeployment kind-cluster1 -o json | jq .status.conditions
[
  ...
  {
    "lastProbeTime": "2021-02-01T14:02:42Z",
    "lastTransitionTime": "2021-02-01T14:02:42Z",
    "message": "SyncSet apply is successful",
    "reason": "SyncSetApplySuccess",
    "status": "False",
    "type": "SyncSetFailed"
  },
  {
    "lastProbeTime": "2021-02-01T14:02:41Z",
    "lastTransitionTime": "2021-02-01T14:02:41Z",
    "message": "cluster is reachable",
    "reason": "ClusterReachable",
    "status": "False",
    "type": "Unreachable"
  },
  ...
]
```

## Managed Cluster - CRC

Export the CRC kubeconfig filename to be created:

```bash
~$ export KUBECONFIG=/tmp/crc.conf
```

Login to your CRC cluster with the `kubeadmin` user:

```bash
~$ oc login -u kubeadmin -p **** https://api.crc.testing:6443
```

The `/tmp/crc.conf` file should now contain the dockerconfig for your
CRC cluster.

Export the Hive kubeconfig filename and enter the `hive` directory (or just
switch to the first terminal, the one used to deploy Hive):

```bash
~$ export KUBECONFIG=/tmp/hive.conf
~$ cd hive
```

Because Hive will not provision that cluster, we have can use the `hiveutil`
to adopt it:

```bash
~/hive$ bin/hiveutil create-cluster \
--base-domain=crc.openshift.com crc  \
--adopt --adopt-admin-kubeconfig=/tmp/crc.conf \
--adopt-infra-id=fakeinfra \
--adopt-cluster-id=fakeid
```

Checking the `ClusterDeployment` status:

```bash
~/hive$ kubectl get clusterdeployment crc -o json | jq .status.conditions
[
  {
    "lastProbeTime": "2021-02-02T14:21:02Z",
    "lastTransitionTime": "2021-02-02T14:21:02Z",
    "message": "cluster is reachable",
    "reason": "ClusterReachable",
    "status": "False",
    "type": "Unreachable"
  },
  {
    "lastProbeTime": "2021-02-02T01:45:19Z",
    "lastTransitionTime": "2021-02-02T01:45:19Z",
    "message": "SyncSet apply is successful",
    "reason": "SyncSetApplySuccess",
    "status": "False",
    "type": "SyncSetFailed"
  }
]
```

Tip: in case the cluster status is "unreachable", that's because Hive runs
in a Kubernetes cluster deployed inside a container, and it is trying to access
the CRC virtual machine that is controlled by libvirt. You will have to figure
out you firewall, but this is what worked for me:

```bash
~/hive$ firewall-cmd --permanent --zone=trusted --change-interface=docker0
success
~/hive$ firewall-cmd --reload
success
```

## SelectorSyncSet

Export the Hive kubeconfig:

```bash
~$ export KUBECONFIG=/tmp/hive.conf
```

Create a test `SelectorSyncSet`. Example:

```yaml
apiVersion: v1
kind: List
metadata: {}
items:
  - apiVersion: hive.openshift.io/v1
    kind: SelectorSyncSet
    metadata:
      name: cso-test
    spec:
      clusterDeploymentSelector:
        matchLabels:
          api.openshift.com/cso-test: 'true'
      resourceApplyMode: Sync
      resources:
        - apiVersion: v1
          kind: Namespace
          metadata:
            annotations: {}
            labels: {}
            name: cso
```

Apply it:

```bash
~$ kubectl apply -f cso-test.yaml
selectorsyncset.hive.openshift.io/cso-test created
```

Now edit the `ClusterDeployment` of a cluster:

```bash
~$ kubectl edit clusterdeployment kind-cluster1
```

Adding the label `api.openshift.com/cso-test: 'true'` to it. Save and exit.

The `cso` namespace should be now created in the target cluster:

```bash
$ export KUBECONFIG=/tmp/cluster1.conf
$ oc get namespace cso
NAME   STATUS   AGE
cso    Active   81s
```

## Cleanup

To clean-up, delete the two clusters and surrounding clutter:

```bash
~/hive$ kind delete cluster --name hive
~/hive$ kind delete cluster --name cluster1
~/hive$ docker rm -f kind-registry
~/hive$ docker network rm kind
```
