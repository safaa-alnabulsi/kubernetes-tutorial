# kubernetes-tutorial

## Intro 

### What is Kubernetes?
System for running many differet containers over multiple different machines

### Why use Kubernetes?
When you need to run many different containers with different images



## Working with Kubernetes

### Development 
we use `minikube` to run the cluter and `kubectl` to play with the cli

#### Prerequisites locally

- Install following libraries on Mac:

        brew install minikube
        brew install kubectl

- Download and install virtualbox

- You can also use Docker Desktop's built-in Kubernetes instead of minikube

    1. Click the Docker icon in the top macOS toolbar
    
    2. Click Preferences
    
    3. Click "Kubernetes" in the dialog box menu
    
    4. Check the “Enable Kubernetes” box
    
    5. Click "Apply"
    
    6. Click Install to allow the cluster installation (This may take a while).


#### Play locally

- to start minikube

        minikube start

- to check the status
        
        $ minikube status
        host: Running
        kubelet: Running
        apiserver: Running
        kubeconfig: Configured

        
        $ kubectl cluster-info
        Kubernetes master is running at https://192.168.64.2:8443
        KubeDNS is running at https://192.168.64.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

 
 
### Production

EKS for AWS and GKE