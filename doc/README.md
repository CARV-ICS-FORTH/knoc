
# Start MiniKube

## Install
```bash
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

## Run
And then Create a minikube single-node-cluster.
- Profile (-p) : Create a new minikube profile
- Server (--apiserver-ips): Add the local advertised ip. This is for Door to communicate with Knoc.

```bash
$ export MINIKUBE_PROFILE=knoc

$ minikube start -p $MINIKUBE_PROFILE --kubernetes-version=v1.19.13 --apiserver-ips=$(curl ipinfo.io/ip)
```
## Install kubectl

```bash
$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

## connect your clis (kubectl,docker, etc.) with the newly-created-cluster's context

Whenever you want to use kubectl or docker commands, you have to first configure the terminal session you are in, with the command below:
```bash
$ eval $(minikube -p $MINIKUBE_PROFILE docker-env) 
```

    
# Installation steps using minikube
```bash
# we state the name of the helm's release we re going to use later
$ export HELM_RELEASE=knoc

# Download the source code
$ git clone git@github.com:CARV-ICS-FORTH/KNoC.git
$ cd KNoC


# setup argo in our cluster (Argo 3.0.2) that uses a slightly modified version of k8sapi-executor
$ kubectl create ns argo
$ kubectl apply -n argo -f deploy/argo-install.yaml

# Expose Kubernetes API server from minikube, using socat. 
# This is required for the argo executor that needs connection to the K8s Api server
# Then socat forwards traffic from the <system_that_hosts_minikube> to the ip of minikube
# redirect web traffic from one server to the local
$ socat TCP-LISTEN:38080,fork TCP:$(minikube -p $MINIKUBE_PROFILE ip):8443 &

# And now you can run this

$ helm upgrade --install --wait $HELM_RELEASE chart/knoc --namespace default \
    --set knoc.k8sApiServer=https://<system_that_hosts_minikube>:38080 \
    --set knoc.remoteSecret.address="<target-system-ip>" \
    --set-string knoc.remoteSecret.port="<target-system-port>" \
    --set knoc.remoteSecret.user="<target-system-username>" \
    --set knoc.remoteSecret.privkey="private key" \
    --set knoc.remoteSecret.kubeContext="current-kubernetes-context"

#i.e.
#$ helm upgrade --install --wait $HELM_RELEASE chart/knoc --namespace default \
#     --set knoc.k8sApiServer=https://139.91.92.71:38080 \
#     --set knoc.remoteSecret.address="139.91.92.100" \
#     --set-string knoc.remoteSecret.port="22" \
#     --set knoc.remoteSecret.user="malvag" \
#     --set knoc.remoteSecret.privkey="$(cat $HOME/.ssh/id_rsa)" \
#     --set knoc.remoteSecret.kubeContext="$(kubectl config current-context)"


$ helm upgrade --install --wait $HELM_RELEASE chart/knoc --namespace default \
    --set knoc.k8sApiServer=https://$(curl ipinfo.io/ip):38080 \
    --set knoc.remoteSecret.address="139.91.92.100" \
    --set knoc.remoteSecret.user=$(whoami) \
    --set knoc.remoteSecret.privkey="$(cat $HOME/.ssh/id_rsa)" \
    --set knoc.remoteSecret.kubeContext="$(kubectl config current-context)"

# now we can test our KNoC deployment
$ kubectl create -f examples/argo-workflow-sample.yaml
```

# Tear down

## Remove knoc
```bash
helm uninstall $HELM_RELEASE
```
In case you want to clean everything from the remote side:

```bash
# Clean slurm outputs and door executable
rm -f slurm-*.out door
# now let's clean door logs, kubernetes associated files and generated scripts
rm -rf .knoc .tmp
```

## Delete minikube's profile
This command deletes the whole minikube-vm that includes the Kubernetes and the Docker deployments inside the vm.

```bash
minikube stop -p $MINIKUBE_PROFILE
minikube delete -p $MINIKUBE_PROFILE
```

## Remove minikube and its data
```bash
minikube stop -p $MINIKUBE_PROFILE; minikube delete -p $MINIKUBE_PROFILE

# ++ optionally: if you run minikube on docker
docker stop (docker ps -aq)
# ++

rm -r ~/.kube ~/.minikube
sudo rm /usr/local/bin/minikube
systemctl stop '*kubelet*.mount'
sudo rm -rf /etc/kubernetes/

# ++ optionally: if you run minikube on docker
docker system prune -af --volumes
# ++
```