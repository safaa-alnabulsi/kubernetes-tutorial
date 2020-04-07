# kubernetes-tutorial

## Intro 

### What is Kubernetes?
System for running many differet containers over multiple different machines.
It is a system to deploy **containerized** apps.


### Why use Kubernetes?
When you need to run many different containers with different images

### The master

It is a machine (vm) with a set of programs to manage nodes. It gets our desired states from the deployment file.
It keeps always watching the nodes and make sure the list of the responsibilities is fulfilled. 

### Node vs. Pod vs. Service

#### Node

- it is the machine/ vm where we run the cluster components on.

#### Pod

- Pod lives inside a node.
- It is a smallest thing we can deploy to run a container.
- We cannot run a container directly on a cluster in k8s world.
- Containers needed to run together (coupled together) are deployed to one Pod.
- A pod can run one or more container inside of it.
 
#### Service

- used to setup networking in the k8s cluster.
- Sub types:
    - ClusterIP
    - NodePort: exposes continaer to the outside world (e.g. access continaer from broswer). Used for dev.
    - LoadBalancer
    - Ingress

### Config file attributes

#### apiVersion 

It is the scope of the limit of objects that we can create

- v1: allows us to create objects of type [componentStatus, configMap, Endpoints, Event, Namespace, Pod]

- apps/v1: allows us to create objects of type [ControllerRevision, StatefulSet]

#### kind 
it is the object type. Examples:

- StatefulSet
- ReplicaController
- Pod (good fo dev only, rarely used directly in production)
- Service
- Deployment (good for dev and production)

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

##### simple-k8s

- to start the minikube

        $ minikube start

- to feed a config file (deployment file) to the Kubectl (this goes to the kube-apiserver, the master and not to the nodes!)
        
        $ kubectl apply -f simple-k8s/client-pod.yaml
          pod/client-pod created

        $ kubectl apply -f simple-k8s/client-node-port.yaml
        service/client-node-port created

        
- to check the status of any object that havve been created

        $ kubectl get pods
        NAME         READY   STATUS    RESTARTS   AGE
        client-pod   1/1     Running   0          107s
        
        
        $ kubectl get services
        NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
        client-node-port   NodePort    10.108.124.40   <none>        3050:31515/TCP   3m17s
        kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP          46h

- to access the multi-client pod from broswer, we need to ask for the IP for the k8s node vm which was created by minikube.

No local host!! http://localhost:31515/ won't work. Instead you have to run:

        $ minikube ip
        192.168.64.2

Then you can access: http://192.168.64.2:31515/

- to get the information of events of lifescycle of an object, run

        $ kubectl describe pod client-pod

- to delete an existing pod

        $ kubectl delete -f simple-k8s/client-pod.yaml
        pod "client-pod" deleted

- to feed deployment config to kubectl, then get information

        $ kubectl apply -f simple-k8s/client-deployment.yaml
        deployment.apps/client-deployment created

        $ kubectl get pods 
        NAME                                 READY   STATUS    RESTARTS   AGE
        client-deployment-755b85c884-jv25x   1/1     Running   0          15s
        
        $ kubectl get deployments
        NAME                READY   UP-TO-DATE   AVAILABLE   AGE
        client-deployment   1/1     1            1           37s

        $ kubectl get pods -o wide
        NAME                                 READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
        client-deployment-755b85c884-jv25x   1/1     Running   0          3m4s   172.17.0.4   minikube   <none>           <none>

- it is so challanging to update the image to the latest version, so we do the following imperative way:

        $ kubectl set image <Object-type>/<object-name> <container-name from deployment config file>=<full-path to image with version>

like so:

        $ kubectl set image Deployment/client-deployment client=safaa1001/multi-client:v1
         

##### multi-k8s

- to update the existing pod to use the multi-worker image:
  
    - Imperative approach: run a command to update the pod
   
    - Declarative approach: update the config file and apply it into kubectl! (preferred)
    



### Production

EKS for AWS and GKE