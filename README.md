# KNoC - A Kubernetes Node to manage container lifecycle on HPC clusters

<p align="center">
    <img src="./media/Dark.png#gh-light-mode-only" height="420" width="420" style="background-position: center center;background-repeat: no-repeat;">
    <img src="./media/Light.png#gh-light-mode-only"  height="420" width="420" style="background-position: center center;background-repeat: no-repeat;">
</p>

KNoC is a [Virtual Kubelet](https://github.com/virtual-kubelet/virtual-kubelet) Provider implementation that manages real pods and containers in a remote container runtime by supporting the lifecycle management of pods, containers and other resources in the context of Kubernetes.

[Virtual Kubelet](https://github.com/virtual-kubelet/virtual-kubelet) is an open source [Kubernetes](https://kubernetes.io/) kubelet implementation that masquerades as a kubelet for the purposes of connecting Kubernetes to other APIs.

Remote environments include [Singularity](https://sylabs.io/singularity/) container runtime utilizing [Slurm's](https://slurm.schedmd.com/) resource management and job scheduling

## Features
- Create, delete and update pods
- Container logs and exec
- Get pod, pods and pod status
- Support for EmptyDirs, Secrets and ConfigMaps

![diagram](media/knoc-env.png)

## Documentation
You can find all relative information in [Documentation](https://github.com/CARV-ICS-FORTH/KNoC/blob/master/doc/README.md)