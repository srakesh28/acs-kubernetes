# Lab2 - Containers Orchestrator hands-on lab with Kubernetes 

# Contents / Agenda

a.	Kubernetes Installation on Azure

b.	Hello-world on Kubernetes

c.	Experimenting with Kubernetes Features
    i.	Placement
    ii.	Reconciliation
    iii.	Rolling Updates
    
d.	Deploying a Pod and Service from a public repository 

e.	Create Azure Container Service Repository (ACR) 

f.	Enable OMS monitoring of containers

g.	Create and deploy into Kubernetes Namspaces

# Lab Goals:

•	To get a primarily hands-on experience with Docker, Kubernetes and Kubernetes’ main features on Azure.

•	To set up a Kubernetes deployment in your subscription for future customer-demo purposes. 

•	To write and run a basic hello-world program on Docker

# A.	Kubernetes Installation on Azure

Note: A number of resources created in this demo have names that must be globally unique (e.g. ACR endpoints). In those cases the commands will include a placeholder value noted within angle brackets <> to signal that a value specific to your environment needs to be provided.
Outline:

1.	Download and install Azure CLI
2.	Logging in to your Azure subscription
3.	Create the Kubernetes cluster
4.	Install Kubectl command line
5.	Download the K8s cluster configuration
6.	Test your Kubernetes installation
Details: 

o	You can create a cluster through CLI v2 or the Azure Portal. If you use the portal you will need to have created and provide a SSH key and Service Principal. If you use the command line you can have the SSH key and Service Principal created for you. For this tutorial, we will use the command line. 
o	For production deployments you would want to create and managed keys and Service Principals for specific purposes. To quickly create a cluster for demo/development purposes you can use the command line which will auto create:
	SSH keys - in your home/.ssh directory
	Service Principal - in your home/.azure directory

o	Note: If you already have ssh keys in your home directory, then you should use those keys on the command line rather then allowing the CLI to create new keys which will overwrite any existing keys in your home/.ssh directory.
The following steps will create the Kubernetes cluster using command line commands: 

1.	Download and install Azure CLI (if you don’t have it installed locally): 
Follow the guide at < https://azure.microsoft.com/en-us/documentation/articles/xplat-cli-install/> to install Azure CLI v2 on your local machine.

2.	Login to your Azure subscription and create a new resource group
    
    $> az login
    
    $> az group create --name my-k8-clusters --location westus

 
3.	Create the Kubernetes cluster
 
    $> az acs create --name my-k8-cluster --resource-group my-k8-clusters --orchestrator-type Kubernetes --dns-prefix <my-k8> --generate-ssh-keys
 
The above command will use ACS to create a new Kubernetes cluster named “my-k8-cluster” within the newly created resource group. 

The orchestrator-type parameter indicates to ACS that you are creating a kubernetes cluster with a dns parameter and to generate new ssh keys and service principals. 
    
    
    
    
4.	Install Kubectl command line 

If not already installed, you can use the cli to install the k8 command line utility (kubectl). Note:

On Windows you need to have opened the command windows with Administrator rights as the installation tries write the program to “C:\Program Files\kubectl.exe”.
You may also have to add “C:\Program Files” to your PATH
 
    $> az acs kubernetes install-cli

5.	Download the k8s cluster configuration (including credentials): 
The kubectl application requires configuration data which includes the cluster endpoint and credentails. 
The credentails are created on the cluster admin server during installation and can be downloaded to your machine using the get-credential subcommand.

    az acs kubernetes get-credentials --resource-group=my-k8-clusters --name=my-k8-cluster
    

6.	Test your Kubernetes Installation:
    
    kubectl cluster-info

After downloading the cluster configuration you should be able to connect to the cluster using kubectl. For example the cluster-info command will show details about your cluster.
 
# B. Hello-World on Kubernetes

(Source): http://kubernetes.io/docs/user-guide/quick-start/

1.	To run a simple app on Kubernetes, we can use: kubectl run 

Let us start by running a simple HTTP server: nginx. Nginix is a popular HTTP server, with a pre-built container on Docker hub. The kubectl run commands below will create two nginx replicas, listening on port 80, and a public IP address for your application.

$> kubectl run my-nginx --image=nginx --replicas=2 --port=80
deployment "my-nginx" created

2.	To expose your service to the public internet, use:

$> kubectl expose deployment my-nginx --target-port=80 --type=LoadBalancer
service "my-nginx" exposed

3.	You can see that they are running by: 

$> kubectl get po
NAME                                READY     STATUS    RESTARTS   AGE
my-nginx-3800858182-h9v8d           1/1       Running   0          1m
my-nginx-3800858182-wqafx           1/1       Running   0          1m

4. Kubernetes will ensure that your application keeps running, by automatically restarting containers that fail, spreading containers across nodes, and recreating containers on new nodes when nodes fail.
To find the public IP address assigned to your application, execute:

$> kubectl get service my-nginx
NAME         CLUSTER_IP       EXTERNAL_IP       PORT(S)                AGE
my-nginx     10.179.240.1     25.1.2.3          80/TCP                 8s

5.	To kill the application and delete its containers and public IP address, do: 

$> kubectl delete deployment,service my-nginx
deployment "my-nginx" deleted
service "my-nginx" deleted

# C. Experimenting with Kubernetes Features
We will highlight three main Kubernetes features in this section: placement, reconciliation and application rolling updates. 

A.	Placement 
(Source): http://kubernetes.io/docs/user-guide/configuring-containers/
(Source): http://kubernetes.io/docs/user-guide/node-selection/

This section focuses on containers’ placement using Kubernetes, which addresses two main issues a) placing a container relative to other containers, whether required to be on the same node as a collection (i.e. pod) or on different nodes and b) which nodes to run the collection of containers relative to other collections. We will address each issue at a time.   

# Placing a Container Relative to Other Containers:

Kubernetes uses the concept of Pods as a logical abstract to collect containers in the same collection, which can then be guaranteed to be deployed to the same node. 

1.	Kubernetes executes containers in Pods. A pod containing a simple Hello World container can be specified in YAML as follows:
 
 $> cat hello-world.yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
spec:  # specification of the pod's contents
  restartPolicy: Never
  containers:
  - name: hello
    image: "ubuntu:14.04"
    command: ["/bin/echo", "hello", "world"]

2.	This pod can be created using the create command: 

$ kubectl create -f ./hello-world.yaml

pods/hello-world

3.	You can see the pod you created using the get command:

$ kubectl get pods
NAME          READY     STATUS    RESTARTS   AGE
hello-world   1/1       Running   0          5s

4.	Terminated pods aren’t currently automatically deleted, so you will need to delete them manually using:

$ kubectl delete pod hello-world
pods/hello-world


5.	Let us now start a new pod with two containers. For example, the following configuration file creates two containers: a redis key-value store image, and a nginx frontend image.

$ cat pod_sample.yaml

apiVersion: v1
kind: Pod
metadata:
  name: redis-nginx
  labels:
    app: web
spec:
  containers:
    - name: key-value-store
      image: redis
      ports:
        - containerPort: 6379
    - name: frontend
      image: nginx
      ports:
        - containerPort: 8000

# Placing Pods on Various Kubernetes Nodes:

You can constrain a pod to only be able to run on particular nodes or to prefer to run on particular nodes. There are several ways to do this, and they all use label selectors to make the selection. Generally, such constraints are unnecessary, as the scheduler will automatically do a reasonable placement (e.g. spread your pods across nodes, not place the pod on a node with insufficient free resources, etc.) but there are some circumstances where you may want more control where a pod lands, e.g. to ensure that a pod ends up on a machine with an SSD attached to it, or to co-locate pods from two different services that communicate a lot into the same availability zone.
nodeSelector is the simplest form of constraint. nodeSelector is a field of PodSpec. It specifies a map of key-value pairs. For the pod to be eligible to run on a node, the node must have each of the indicated key-value pairs as labels (it can have additional labels as well).

1.	You need to attach labels to a node using; kubectl label nodes <node-name> <label-key>=<label-value>, as follows:

 Get the names of the nodes on your cluster
      $> kubectl get nodes

 then add a label to your specific node
      $> kubectl label nodes kubernetes-foo-node-1.c.a disktype=ssd

2.	You can then specify the label in your pod config file as a nodeSelector section

$> cat pod2_sample.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx2
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd

# B.	Reconciliation
(Source): http://kubernetes.io/docs/user-guide/replication-controller/
A replication controller ensures that a specified number of pod “replicas” are running at any one time. In other words, a replication controller makes sure that a pod or homogeneous set of pods are always up and available. If there are too many pods, it will kill some. If there are too few, the replication controller will start more. Unlike manually created pods, the pods maintained by a replication controller are automatically replaced if they fail, get deleted, or are terminated. 
For example, your pods get re-created on a node after disruptive maintenance such as a kernel upgrade. A simple case is to create 1 Replication Controller object in order to reliably run one instance of a Pod indefinitely.

1.	Create an example of Replication Controller config. It runs 3 copies of the nginx web server.

$> cat pod_w_replica_sample.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx3
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx

2.	Run the new pod

  $>  kubectl create -f ./pod_w_replica_sample.yaml
  replicationcontrollers/nginx
 
3.	Check the status of the replica controller

$>  kubectl describe replicationcontrollers/nginx
Name:		nginx
Namespace:	default
Image(s):	nginx
Selector:	app=nginx
Labels:		app=nginx
Replicas:	3 current / 3 desired
Pods Status:	0 Running / 3 Waiting / 0 Succeeded / 0 Failed
Events:
  FirstSeen				LastSeen			Count	From
SubobjectPath	Reason			Message
  Thu, 24 Sep 2015 10:38:20 -0700	Thu, 24 Sep 2015 10:38:20 -0700	1
{replication-controller }			SuccessfulCreate	Created pod: nginx-qrm3m
  Thu, 24 Sep 2015 10:38:20 -0700	Thu, 24 Sep 2015 10:38:20 -0700	1
{replication-controller }			SuccessfulCreate	Created pod: nginx-3ntk0
  Thu, 24 Sep 2015 10:38:20 -0700	Thu, 24 Sep 2015 10:38:20 -0700	1
{replication-controller }			SuccessfulCreate	Created pod: nginx-4ok8v

4.	Now, you can increase the number of replicas from 3 to 5 as follows, then query your pod using “kubectl describe replicationcontrollers/nginx”:

$> $ kubectl scale rc nginx --replicas=5

5.	you can experiment with deleting a container and see how it come up again afterwards automatically by the replica controller. 

$> # use cmd line instructions from first part of the lab

# C.	Applications’ Rolling Updates

(source): http://kubernetes.io/docs/user-guide/rolling-updates/ and http://kubernetes.io/docs/user-guide/update-demo/ 
To update a service without an outage, kubectl supports what is called ‘rolling update’, which updates one pod at a time, rather than taking down the entire service at the same time. A rolling update applies changes to the configuration of pods being managed by a replication controller. The changes can be passed as a new replication controller configuration file; or, if only updating the image, a new container image can be specified directly.
This example demonstrates the usage of Kubernetes to perform a rolling update for a new container image on a running group of pods.

1.	Let’s say you were running version 1.7.9 of nginx


$>  cat my-nginx.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx
spec:
  replicas: 5
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9

2.	To update to container image to ngnix 1.9.1, you can use kubectl rolling-update --image to specify the new image:

  $>  $ kubectl rolling-update my-nginx --image=nginx:1.9.1
  Created my-nginx-ccba8fbd8cc8160970f63f9a2696fc46
 
3.	In another window, you can see that kubectl added a deployment label to the pods, whose value is a hash of the configuration, to distinguish the new pods from the old:

    $ kubectl get pods -l app=nginx -L deployment
NAME                                              READY     STATUS    RESTARTS   AGE       DEPLOYMENT
my-nginx-ccba8fbd8cc8160970f63f9a2696fc46-k156z   1/1       Running   0          1m        ccba8fbd8cc8160970f63f9a2696fc46
my-nginx-ccba8fbd8cc8160970f63f9a2696fc46-v95yh   1/1       Running   0          35s       ccba8fbd8cc8160970f63f9a2696fc46
my-nginx-divi2                                    1/1       Running   0          2h        2d1d7a8f682934a254002b56404b813e
my-nginx-o0ef1                                    1/1       Running   0          2h        2d1d7a8f682934a254002b56404b813e
my-nginx-q6all                                    1/1       Running   0          8m        2d1d7a8f682934a254002b56404b813e

4.	If you encounter a problem, you can stop the rolling update midway and revert to the previous version using --rollback:

$ kubectl rolling-update my-nginx --rollback
Setting "my-nginx" replicas to 1
Continuing update with existing controller my-nginx.
Scaling up nginx from 1 to 1, scaling down my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 from 1 to 0 (keep 1 pods available, don't exceed 2 pods)
Scaling my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 down to 0
Update succeeded. Deleting my-nginx-ccba8fbd8cc8160970f63f9a2696fc46
replicationcontroller "my-nginx" rolling updated

This is one example where the immutability of containers is a huge asset, in rolling new updates and in devops. 
For more exercises and info on Kubernetes, the Kubernetes website has a wealth of information and easy to follow on its various features (http://kubernetes.io/docs/).  

# D.	Deploying a Pod and Service from a public repository 

The following steps can be used to quickly deploy an image from DockerHub (a public repository) that is made available via Azure external load balancer.
The kubectl get services command will show the EXTERNAL-IP as 'Pending' until a public IP is provisioned for the service on the load balancer. Once the EXTERNAL-IP is assigned you can use that IP to render the nginx landing page.

kubectl run nginx --image nginx
kubectl get pods
kubectl expose deployments nginx --port=80 --type=LoadBalancer
kubectl get services

You can also enable access to the k8 web UI console via local proxy

kubectl proxy

The output will note the port that the proxy binds to. The console will then be available at that port on localhost. e.g. http://localhost:8001/ui

# E.	Create Azure Container Service Repository (ACR) 

In the previous step the image for ngnix was pulled from a public repository. For many customers they want to only deploy images from internal (controlled) private registries.
1.	Create ACR Registry

Note: ACR names are globally scoped so you can check the name of a regsitry before trying to create it

az acr check-name --name <myk8acr>

The minimal parameters to create a ACR are a name, resource group and location. With these paramters a storage account will be created and administrator access will not be created.
Note: the command will return the resource id for the registry. That id will need to be used in subsequent steps if you want to create service principals that are scoped to this registry instance.

az acr create --name <myk8acr> --resource-group my-k8-clusters --location westus

Create a two service principals, one with read only and one with read/write access.
Note:
1.	Ensure that the password length is 8 or more characters
2.	The command will return an application id for each service principal. You'll need that id in subsequent steps.
3.	You should consider using the --scope property to qualify the use of the service principal a resource group or registry

az ad sp create-for-rbac --name my-acr-reader --role Reader --password my-acr-password
az ad sp create-for-rbac --name my-acr-contributor  --role Contributor --password my-acr-password

2.	Push demo app images to ACR

List the local docker images. You should see the images built in the initial steps when deploying the application locally.

docker images

Tag the images for service-a and service-b to associate them with you private ACR instance.
Note that you must provide your ACR registry endpoint

docker tag service-a:latest <myk8acr-microsoft.azurecr.io>/service-a:latest
docker tag service-b:latest <myk8acr-microsoft.azurecr.io>/service-b:latest

Using the Contributor Service Principal, log into the ACR. The login command for a remote registry has the form: 
docker login -u user -p password server
docker login -u <ContributorAppId>  -p <my-acr-password> <myk8acr-microsoft.azurecr.io> 

3.	Push the images

docker push <myk8acr-microsoft.azurecr.io>/service-a
docker push <myk8acr-microsoft.azurecr.io>/service-b

At this point the images are in ACR, but the k8 cluster will need credentails to be able to pull and deploy the images

4.	Create a k8 docker-repository secret to enable read-only access to ACR

kubectl create secret docker-registry acr-reader --docker-server=<myk8acr-microsoft.azurecr.io> --docker-username=<ContributorAppId> --docker-password=my-acr-password --docker-email=a@b.com

Note: If you use a different name for the secret you will have to update the imagePullSecrets name property in the k8-demo-app.yml file

imagePullSecrets:
        - name: acr-reader

5. Deploy the application to the k8 cluster

Review the contents of the k8-demo-app.yml file. It contains the objects to be created on the k8 cluster.

a.	Note that multiple objects can be included within the same file
b.	Note the environment variables that are used to configure endpoints. Since these containers are being deployed to a Pod:

The containers will communicate via localhost. Containers cannot listen on the same port

Update the image references in the k8-demo-app.yml file to reference your ACR endpoint
    spec:
      containers:
        - name: web
          image: <myk8acr-microsoft.azurecr.io>/service-a
          ...
        - name: api
          image: <myk8acr-microsoft.azurecr.io>/service-b
          ...
        - name: mycache  

Deploy the application using the kubectl create command:

kubectl create -f k8-demo-app.yml

# F. Enable OMS monitoring of containers

This expands on the steps described here https://docs.microsoft.com/en-us/azure/container-service/container-service-kubernetes-oms by using a k8 secret to store the workspace id and key.

Go to the OMS portal and get the workspace id and a key (see the page referenced in the prior link).

Create a k8 secret to hold the workspace id and key. The following command creates a generic secret named oms-agent-secret that holds the properties WSID and KEY. Substitute your WorkspaceID and WorkspaceKey for the placholder values.

kubectl create secret generic oms-agent-secret --from-literal=WSID=<WorkspaceID> --from-literal=KEY=<WorkspaceKey>

Note two points in the file k8-demo-enable-oms.yml
	The deployment type is DaemonSet which means one instance will be deployed to each agent Node
	The reference to the secret created in the prior step to hold the OMS WorkspaceID and WorkspaceKey

Deploy the agent with the following command:

kubectl create -f k8-demo-enable-oms.yml

Within a few minutes you should see metrics and logs for containers deployed in the k8 cluster.

# F.	Create and deploy into Namspaces

Kubernetes provides namespaces as a way to create isolated environments within a cluster (e.g. dev,test,prod)

Create a test and prod namespace using the kubectl create command:

kubectl create -f k8-create-namespaces.yml

Use the UI or command line to verify the available namespaces now include test and prod:

kubectl get namespaces


Deploy the demo application into the test namespace
o	Get a list of the current contexts
o	Create a context named test and bind it to the test namespace
o	Set the current context to test
o	Show the current context again and note that test is tagged as CURRENT
o	Deploy the application into the test namespace

kubectl config get-contexts
kubectl config set-context test --cluster=<my-k8-cluster> --user=<my-k8-cluster-admin> --namespace=test 
kubectl config use-context test
kubectl config get-contexts
kubectl create -f k8-demo-app.yml





































 




















