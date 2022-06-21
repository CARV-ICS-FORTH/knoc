
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
$ minikube start -p knoc --kubernetes-version=v1.19.13 --apiserver-ips=$(curl ipinfo.io/ip)
```

## connect your clis (kubectl,docker, etc.) with the newly-created-cluster's context
```bash
$ eval $(minikube -p knoc docker-env) 
```
  
    
# Installation steps using minikube
```bash
# Download the source code
$ git clone git@github.com:CARV-ICS-FORTH/KNoC.git
$ cd KNoC


# setup argo in our cluster (Argo 3.0.2) that uses a slightly modified version of k8sapi-executor
kubectl create ns argo
kubectl apply -n argo -f deploy/argo-install.yaml

# Expose Kubernetes API server from minikube, using socat. 
# This is required for the argo executor that needs connection to the K8s Api server
# Then socat forwards traffic from the <system_that_hosts_minikube> to the ip of minikube
# redirect web traffic from one server to the local
socat TCP-LISTEN:38080,fork TCP:$(minikube -p knoc ip):8443 &

# And now you can run this

helm install knoc chart/knoc --namespace default \
    --set knoc.k8sApiServer=https://<system_that_hosts_minikube>:38080 \
    --set knoc.remoteSecret.address="<target-system-ip>" \
    --set-string knoc.remoteSecret.port="<target-system-port>" \
    --set knoc.remoteSecret.user="<target-system-username>" \
    --set knoc.remoteSecret.privkey="private key" \
    --set knoc.remoteSecret.kubeContext="current-kubernetes-context"

#i.e.
# helm install knoc chart/knoc --namespace default \
#     --set knoc.k8sApiServer=https://139.91.92.71:38080 \
#     --set knoc.remoteSecret.address="139.91.92.100" \
#     --set-string knoc.remoteSecret.port="22" \
#     --set knoc.remoteSecret.user="malvag" \
#     --set knoc.remoteSecret.privkey="$(cat $HOME/.ssh/id_rsa)" \
#     --set knoc.remoteSecret.kubeContext="$(kubectl config current-context)"


# helm install knoc chart/knoc --namespace default \
#     --set knoc.k8sApiServer=https://$(curl ipinfo.io/ip):38080 \
#     --set knoc.remoteSecret.address="139.91.92.100" \
#     --set-string knoc.remoteSecret.port="22" \
#     --set knoc.remoteSecret.user=$(whoami) \
#     --set knoc.remoteSecret.privkey="$(cat $HOME/.ssh/id_rsa)" \
#     --set knoc.remoteSecret.kubeContext="$(kubectl config current-context)"

# now we can test our KNoC deployment
kubectl create -f examples/argo-workflow-sample.yaml
```
