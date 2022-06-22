
# Install Dependencies

## Minikube
```bash
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

## kubectl

```bash
$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

## Helm

```bash
$ ... 
```

## Start Minikube
And then Create a minikube single-node-cluster.
To do so, you need to specify the minikube profile and the ip where the minikube service is listening to.

```bash
$ export MINIKUBE_PROFILE=knoc
$ export ADVERTISED_HOST=$(curl ipinfo.io/ip)
$ export API_SERVER_PORT=8443
$ export PROXY_API_SERVER_PORT=38080
$ export KUBE_PROXY=${ADVERTISED_HOST}:${PROXY_API_SERVER_PORT}
```

```bash
$ minikube start -p ${MINIKUBE_PROFILE} --kubernetes-version=v1.19.13 --apiserver-ips=${ADVERTISED_HOST}
$ eval $(minikube -p $MINIKUBE_PROFILE docker-env) 

# Expose Kubernetes API server from minikube, using socat. 
# This is required for the argo executor that needs connection to the K8s Api server
# Then socat forwards traffic from the <system_that_hosts_minikube> to the ip of minikube
# redirect web traffic from one server to the local
$ socat TCP-LISTEN:${PROXY_API_SERVER_PORT},fork TCP:$(minikube -p $MINIKUBE_PROFILE ip):${API_SERVER_PORT} &
```

```bash
# Create alias because malvag is very very mal...
$ alias kubectl='minikube -p knoc kubectl --'

$ kubectl config view > ~/.kube/config
```

# Run a workflow on Kubernetes

Before using kubectl or docker commands, you have to first configure the terminal session you are in, with the command below:
```bash
$ export HELM_RELEASE=knoc
$ export SLURM_CLUSTER_IP=thegates
$ export SLURM_CLUSTER_USER=$(whoami)
$ export SLURM_CLUSTER_SSH_PRIV=/home/${SLURM_CLUSTER_USER}/.ssh/id_rsa
```
    
```bash
# Download the source code
$ git clone git@github.com:CARV-ICS-FORTH/KNoC.git
$ cd KNoC

# setup argo in our cluster (Argo 3.0.2) that uses a slightly modified version of k8sapi-executor
$ kubectl create ns argo
$ kubectl apply -n argo -f deploy/argo-install.yaml


# And now you can run this

$ helm upgrade --install --debug --wait $HELM_RELEASE chart/knoc --namespace default \
    --set knoc.k8sApiServer=https://${KUBE_PROXY} \
    --set knoc.remoteSecret.address=${SLURM_CLUSTER_IP} \
    --set knoc.remoteSecret.user=${SLURM_CLUSTER_USER}  \
    --set knoc.remoteSecret.privkey="$(cat ${SLURM_CLUSTER_SSH_PRIV})" \
    --set knoc.remoteSecret.kubeContext="current-kubernetes-context"


# now we can test our KNoC deployment
$ kubectl create -f examples/argo-workflow-sample.yaml
```

# Tear down

## Remove knoc
```bash
helm uninstall --wait $HELM_RELEASE
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
