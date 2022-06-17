# Installation steps using minikube
```bash
# Create a minikube single-node-cluster
minikube start -p knoc --kubernetes-version=v1.19.13 --apiserver-ips=<system_that_hosts_minikube-ip>

# connect your clis (kubectl,docker, etc.) with the newly-created-cluster's context
eval $(minikube -p knoc docker-env) 


# setup argo in our cluster (Argo 3.0.2) that uses a slightly modified version of k8sapi-executor
kubectl create ns argo
kubectl apply -n argo -f deploy/argo-install.yaml


# given that we have an ssh public key on the side we re going to connect to

kubectl create secret generic remote-secret \
    --from-file=ssh-privatekey=$HOME/.ssh/id_rsa \
    --from-file=ssh-publickey=$HOME/.ssh/id_rsa.pub \
    --from-literal=remote_user="<REMOTE_USER>" \
    --from-literal=host="<REMOTE_HOST>" \
    --from-literal=port="<REMOTE_PORT>"

#i.e.
# kubectl create secret generic remote-secret \ 
#     --from-file=ssh-privatekey=$HOME/.ssh/id_rsa \
#     --from-file=ssh-publickey=$HOME/.ssh/id_rsa.pub \
#     --from-literal=remote_user="malvag" \
#     --from-literal=host="139.91.92.100" \
#     --from-literal=port="22"

# Expose Kubernetes API server from minikube, using socat. 
# This is required for the argo executor that needs connection to the K8s Api server
# Then socat forwards traffic from the <system_that_hosts_minikube> to the ip of minikube
socat TCP-LISTEN:38080,fork TCP:$(minikube -p knoc ip):8443


# you should change the CLUSTER_SERVER variable from the deploy/setup_kubeconfig.yaml
# and after that you should run

make skaffold

# now we can test our KNoC deployment
kubectl apply -f examples/argo_workflow_sample.yaml

```