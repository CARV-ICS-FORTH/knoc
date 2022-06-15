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

# Install the kubeconfig("default-config" file in project's root directory) inside the remote .kube directory and
# include the files below in the remote .kube directory
#     certificate-authority: local:$HOME/.minikube/ca.crt                 -> remote:~/.kube/ca.crt
#     client-certificate: local:$HOME/.minikube/profiles/knoc/client.crt  -> remote:~/.kube/client.crt
#     client-key: local:$HOME/.minikube/profiles/knoc/client.key          -> remote:~/.kube/client.key

# the remote .kube directory must look like this:
# -rw-r--r--  1 whoever users  1111 Jun 15 20:43 ca.crt
# -rw-r--r--  1 whoever users  1148 Jun 15 20:44 client.crt
# -rw-r--r--  1 whoever users  1676 Jun 15 20:44 client.key
# -rw-r--r--  1 whoever users   701 Jun 15 20:45 config
# 
# 

make skaffold

# now we can test our KNoC deployment
kubectl apply -f examples/argo_workflow_sample.yaml

```