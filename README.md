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
    - ClusterIP: exposes a set of pods to other objects in the cluster
    - NodePort: exposes continaer (set of pods) to the outside world (e.g. access continaer from broswer). Used for dev.
    - LoadBalancer: legacy way of getting network traffic into a cluster
    - Ingress: exposes a set of services to the outside world

### Config file attributes

#### apiVersion 

It is the scope of the limit of objects that we can create

- v1: allows us to create objects of type [componentStatus, configMap, Endpoints, Event, Namespace, Pod]

- apps/v1: allows us to create objects of type [ControllerRevision, StatefulSet]

- extensions/v1beta1: 

#### kind 
it is the object type. Examples:

- StatefulSet
- ReplicaController
- Pod (good fo dev only, rarely used directly in production)
- Service: setup networking in k8s cluster
- Deployment: pods (good for dev and production)
- Volume: an object that allows a container to store data at all the pod level
- Persistent Volume Claim: it's like an ad of options [statically provisioned PV (already created), dynamically provisioned PV (created on the fly)] 
  Access Modes:
    - ReadWriteOnce: can be used by a single node 
    - ReadOnlyMany: multiple nodes can read from this
    - ReadWriteMany: can be read and written to by many nodes
- Secret: securely stores a piece of information into a cluster e.g. db password
- Ingress

### Imperative vs. Declarative approach

to update the existing pod to use the another image:
  
- Imperative approach: run a command to update the pod

- Declarative approach: update the config file and apply it into kubectl! (preferred)  

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

#### simple-k8s

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
         
- to configure the shell to use the vm docker server (only current terminal window)
        
        $ eval $(minikube docker-env)
        $ docker ps # will let you see the copy of docker inside the minikube vm 
 
check: 
       
        $ minikube docker-env
        export DOCKER_TLS_VERIFY="1"
        export DOCKER_HOST="tcp://ipa-ddress:2376"
        export DOCKER_CERT_PATH="/Users/myuser/.minikube/certs"
        # Run this command to configure your shell:
        # eval $(minikube docker-env)
and then you can check the logs and login into the container shell. 

- to get logs of a pod with kubectl        
        
        $ kubectl get pods
        NAME                                 READY   STATUS    RESTARTS   AGE
        client-deployment-7f95ccd89c-xv9c8   1/1     Running   0          60m
        
        $ kubectl logs client-deployment-7f95ccd89c-xv9c8

- to start a shell inside the pod with kubectl

        kubectl exec -it client-deployment-7f95ccd89c-xv9c8
        
#### multi-k8s

- first you have to delete the previous testing deployment

        $ kubectl get deployments
        NAME                READY   UP-TO-DATE   AVAILABLE   AGE
        client-deployment   1/1     1            1           25h
        
        $ kubectl delete deployment client-deployment
        deployment.apps "client-deployment" deleted
     
        $ kubectl get services
        NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
        client-node-port   NodePort    10.108.124.40   <none>        3050:31515/TCP   2d
        kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP          3d22h
        
        $ kubectl delete service client-node-port
        service "client-node-port" deleted

- then we start applying the new config files we have created inside `k8s folder:

        $ kubectl apply -f k8s
        service/client-cluster-ip-service created
        deployment.apps/client-deployment created
        service/server-cluster-ip-service created
        deployment.apps/server-deployment created
        deployment.apps/worker-deployment created

_Note:_ config files can be combined in one with `---` in the middle between objects.

- to check all created objects

        $ kubectl get pods
        NAME                                 READY   STATUS    RESTARTS   AGE
        client-deployment-f6bfd4576-fc58n    1/1     Running   0          20m
        client-deployment-f6bfd4576-r6n4k    1/1     Running   0          20m
        client-deployment-f6bfd4576-x4xgl    1/1     Running   0          20m
        server-deployment-676666b8df-4knx9   1/1     Running   0          9m33s
        server-deployment-676666b8df-pb6c4   1/1     Running   0          9m34s
        server-deployment-676666b8df-rlnm9   1/1     Running   0          9m33s
        worker-deployment-7976d9d9f8-vdpv7   1/1     Running   0          89s
        
        
        $ kubectl get deployments
        NAME                READY   UP-TO-DATE   AVAILABLE   AGE
        client-deployment   3/3     3            3           20m
        server-deployment   3/3     3            3           9m44s
        worker-deployment   1/1     1            1           99s
        
        
        $ kubectl get services
        NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
        client-cluster-ip-service   ClusterIP   10.96.8.77     <none>        3000/TCP   20m
        kubernetes                  ClusterIP   10.96.0.1      <none>        443/TCP    3d23h
        server-cluster-ip-service   ClusterIP   10.96.182.56   <none>        5000/TCP   9m48s

Note: i did the same after creating the redis and postgres parts

- **Volumes**: to avoid losing all the data we have inside postgres as soon as the pod or the container crashes, 
we need to have a **volume** on the host machine that have a consistent file system that can be accessed by a database like postgres.
If the pod craches, the deployment will create a new one and it will be pointed to that volume. 
Be aware of not having two replicas accessing the same volume. You need to add more configuration for that.

        $ kubectl get storageclass 
        NAME                 PROVISIONER                AGE
        standard (default)   k8s.io/minikube-hostpath   5d3h
        
        $ kubectl describe storageclass
        Name:                  standard
        IsDefaultClass:        Yes
        Annotations:           storageclass.kubernetes.io/is-default-class=true
        Provisioner:           k8s.io/minikube-hostpath
        Parameters:            <none>
        AllowVolumeExpansion:  <unset>
        MountOptions:          <none>
        ReclaimPolicy:         Delete
        VolumeBindingMode:     Immediate
        Events:                <none>

-  after adding the persistent volume claim, apply it to local cluster:

        $ kubectl apply -f k8s
        service/client-cluster-ip-service unchanged
        deployment.apps/client-deployment unchanged
        persistentvolumeclaim/database-persistent-volume-claim created
        service/postgres-cluster-ip-service unchanged
        deployment.apps/postgres-deployment configured
        service/redis-cluster-ip-service unchanged
        deployment.apps/redis-deployment unchanged
        service/server-cluster-ip-service unchanged
        deployment.apps/server-deployment unchanged
        deployment.apps/worker-deployment unchanged

- get information about the created pv and pvs
        
        $ kubectl get pv
        NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                      STORAGECLASS   REASON   AGE
        pvc-83145502-8132-4bd7-8653-55bfc563b309   2Gi        RWO            Delete           Bound    default/database-persistent-volume-claim   standard                18s

        $ kubectl get pvc
        NAME                               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
        database-persistent-volume-claim   Bound    pvc-83145502-8132-4bd7-8653-55bfc563b309   2Gi        RWO            standard       43s

- create a secret locally by running:

        $ kubectl create secret generic <secret-name> --from-literal key=value

like so:

        $ kubectl create secret generic pgpassword --from-literal POSTGRES_PASSWORD=password123
        secret/pgpassword created

        $ kubectl get secrets
        NAME                  TYPE                                  DATA   AGE
        default-token-lqcw2   kubernetes.io/service-account-token   3      5d4h
        pgpassword            Opaque                                1      17s

#### Traffic

We won't use `Load balancer` because we need to expose two pods to the outside: server and client. 
Rather we will use "Ingress config" which will create "Ingress controller". The ingress controller will make something that accept incoming traffic. 
Following this [docs](https://kubernetes.github.io/ingress-nginx/deploy/#prerequisite-generic-deployment-command):

1. Make sure you executed the mandatory generic script that was discussed in the lecture:


        $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml
        namespace/ingress-nginx created
        configmap/nginx-configuration created
        configmap/tcp-services created
        configmap/udp-services created
        serviceaccount/nginx-ingress-serviceaccount created
        clusterrole.rbac.authorization.k8s.io/nginx-ingress-clusterrole created
        role.rbac.authorization.k8s.io/nginx-ingress-role created
        rolebinding.rbac.authorization.k8s.io/nginx-ingress-role-nisa-binding created
        clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-clusterrole-nisa-binding created
        deployment.apps/nginx-ingress-controller created
        limitrange/ingress-nginx created

        $ minikube addons enable ingress
        ✅  ingress was successfully enabled

2. Execute the provider specific script to enable the service:


        $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud-generic.yaml
        service/ingress-nginx created

3. Verify the service was enabled by running the following:


        $ kubectl get svc -n ingress-nginx
        NAME            TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
        ingress-nginx   LoadBalancer   10.106.173.62   <pending>     80:32553/TCP,443:32460/TCP   5m18s

- apply the ingress config with kubectl
        
        $ kubectl apply -f k8s

Then get the ip and access it from browser without port.

- A very interesting command, to see the K8S UI

        $ minikube dashboard

### Production

EKS for AWS and GKE for Google.

## Refernces
- [Docker and Kubernetes: The Complete Guide Udemy Course](https://www.udemy.com/course/docker-and-kubernetes-the-complete-guide/)
- [Storage Classes Provisioner](https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner)
- [NGINX Ingress Controller for Kubernetes Repo](https://github.com/kubernetes/ingress-nginx)
- [Studying the Kubernetes Ingress system, article by Hongli Lai ](https://www.joyfulbikeshedding.com/blog/2018-03-26-studying-the-kubernetes-ingress-system.html)