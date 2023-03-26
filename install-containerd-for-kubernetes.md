# Install containerd in Ubuntu 22.04 for Kubernetes  

Docker isn't mandatory to run kubernetes since 1.20, this guid list docs about how to install containerd instead of docker  

## main starting point:  
https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd

## run containerd with systemd  
The containerd.service can be used for launching containerd, the script is provided by containerd github rep:  
https://github.com/containerd/containerd/blob/main/containerd.service
