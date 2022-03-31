# KNoC - A Kubernetes Node to manage container lifecycle on HPC clusters

KNoC is a [Virtual Kubelet](https://github.com/virtual-kubelet/virtual-kubelet) Provider implementation that manages real pods and containers in a remote container runtime by supporting the lifecycle management of pods, containers and other resources in the context of Kubernetes.

[Virtual Kubelet](https://github.com/virtual-kubelet/virtual-kubelet) is an open source [Kubernetes](https://kubernetes.io/) kubelet implementation that masquerades as a kubelet for the purposes of connecting Kubernetes to other APIs.

Remote environments include [Singularity](https://sylabs.io/singularity/) container runtime utilizing [Slurm's](https://slurm.schedmd.com/) resource management and job scheduling

## Features
- Create, delete and update pods
- Container logs and exec
- Get pod, pods and pod status
- Support for EmptyDirs, Secrets and ConfigMaps

![diagram](media/knoc-env.png)

## Installation
- First, install [skaffold from here](https://skaffold.dev/docs/install/)
- Then install go dependencies for KNoC and build:
```bash
go get github.com/sfreiberg/simplessh
go get golang.org/x/crypto/ssh/terminal@v0.0.0-20201221181555-eec23a3978ad
#builds the KNoC("virtual-kubelet") binary
make build
```

- Optionally, make a container:
```bash
make container
```

- Prepare the secret needed for the communication with the remote system:
```bash
# generate a secret with ssh keys and remote-user
kubectl create secret generic remote-secret \
    --from-file=ssh-privatekey=$HOME/.ssh/id_rsa \
    --from-file=ssh-publickey=$HOME/.ssh/id_rsa.pub \
    --from-literal=remote_user="<USER>" \
    --from-literal=host="<HOST>" \
    --from-literal=port="<PORT>"
```

*[OPTIONAL] Alternatively KNoC uses bitnami's [kubeseal](https://github.com/bitnami-labs/sealed-secrets) that encrypts the above secret for your cluster and creates a sealed-secret. KNoC requests the secrets values and kubeseal decrypts them upon request. Then KNoC uses the produced k8s secret and uses these values as enviromental variables inside the [KNoC's yaml](https://github.com/CARV-ICS-FORTH/knoc/blob/master/deploy/pod.yml)*

- Create and add the sealed-secret inside the skaffold manifests:
```bash
# install bitnami's kubeseal
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.15.0/kubeseal-linux-amd64 -O kubeseal
sudo install -m 755 kubeseal /usr/local/bin/kubeseal

# deploy kubeseal's controller in our cluster
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.15.0/controller.yaml

#encrypt it only for your k8s controller
kubectl get secret remote-secret -o yaml > deploy/remote-secret.yml
kubeseal < deploy/remote-secret.yml > deploy/sealed-remote-secret-crd.yaml

#add the correct sealed-secret-crd file to the manifests of skaffold.yaml
# i.e.
# deploy:
#  kubectl:
#     manifests:
#      - deploy/sealed-remote-secret-crd.yaml # <--------
```

## Running
```bash
#Builds a container for the knoc-vkubelet and deploys it on your k8s
#using the existing k8s context and config
#(skaffold: by default in dev mode)
make skaffold
```

## Setup in minikube
```bash
minikube start --kubernetes-version=v1.19.13 --apiserver-ips=<host-ip>
# then copy your local certificates and configs from ~/.minikube to your "knoc-targetted" system

# --apiserver-ips flag is used so that minikube apiserver can be accesed outside the local network it resides.
```

After setting up argo you can run a sample workflow located in examples/
