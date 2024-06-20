# Microservices with Node React Docker and Kubernetes

## Fundamental ideas

### What is a microservice?

A monolith contains routing, middleware, business logic and db access to implement all features of our app
A single microservice contains routing, middleware, business logic and db acces to implement one feature of our app

### Data in microservices

The biggest challenge is **data management between services**
How we store data and how we access it
Each service gets its own database
Services will never ever reach into another database

Database per service:

- we want each service to run independently of others
  Otherwise if anything bad happens to a service A, service B will not work either
- data/structure might change between services
- some service might function more efficiently with differents type of db (sql vs nosql)

### Big problems with Data

2 general strategies to solve this issue

Sync or Async (not same as in javascript)

Sync: service communicate with another using direct requests
Async: services communicate with another one with events

### Sync Communication between services

With sync, you may not need a database
But:
you introduce dependencies between services
And if any inter service request fails, the request fails
And it's only as fast as the slowest request
And it can introduce a web of requests

### Event-Based communication

Introduce something called Event Bus to handle events or notifications from different services
Each service will connect to the Event Bus and emit or receive notifications
A service will emit an event to the Event Bus:
event { type: UserQUery, data: { id: 1} }
this event will be catched by the user service and will emit a new event to the bus:
event { type: UserQueryResult, data: { id: 1, name: 'Jim' }}
And our service will receive this event and handle it

But this system has many issues, a bit like the sync cons

### Storing data

A bit bizarre and inefficient way of storing data:
Define the exact goal of the service.
Instead of "code to show products ordered by a user":
Given the id of the user, show the title and image for every product the user have ever ordered
The idea of a db to solve this request:
A record of User (id, product ids)
A record of Products (id, title, image)

So as we create a product, we'll at the same time send an event to the bus event that a product has been created, and our service will catch it and add this product in its db
And same thing when a user signup or orders a product

### Async comm. pros and cons

- no direct dependencies on other service
- very fast service

* data duplication, paying extra storage
* harder to understand

## Mini app

Blog project: Posts and comments to start
So 2 resources = 2 services
A Post service and a Comment service
Post: create a post and list all posts
Comment: create a comment and list all comments (dependent to creating / listing all posts)

Create posts with express app
Use postman to test your requests are working

Create react app and call localhosts created with express
Fix for Cors errors because ports are different:
app.use(cors())

In client app, how to minimize the number of requests?
Make each post request its comments
So how to Make one request for one post and its comments?
Two possible solutions: sync and async

### Event buses

Many implementations exist: rabbitmq, kafka, nats
They receive events and publish them to listeners

An easy solution is that all services have a post to the route /events and the event bus will get them and send along the event to all services even the one that emitted the event.

If you then want to add a new feature like comment moderation, approved, pending and rejected:
you can create a new service and modify the comment by adding a new status property and then flag the status

option 1: Moderation service communicates event creation to query service
like a type CommentModerated with data + status
Then the query service will receive the CommentModerated event
The con is that user will not see the comment immediately, only when moderated

option 2: Moderation updates status at both comments and query services (with default status pending)
The issue is: does it make sense for a presentation service like query to understand how to process a very precise update?
Comments can have many type of updates (downvote, upvote, adv, anonymous, searchable, promoted...) so not the best solution to create service for each type of update

option 3: let the query handle a CommentUpdated type event that takes the data and copy them
the event of type CommentModerated will be handle by the comments service and emits a CommentUpdated event to the bus that will be handled by the query service

If a service crashes how to deal with missing events?
option1: sync requests: the query service needs to know all posts, all comments and store
not ideal at all

option2: direct db access, but not what we want

option3: store events: event bus emit the event and store it at the same time

## Running services with Docker

To solve the many possible deployment issues, we need something that can track all the services, loadbalance them and more
Docker and Kubernetes

### Docker

Docker will create containers to run our programs/Services

Docker will solve two issues
Running our app makes big assumptions about our environment (node / npm)
Running our app requires precise knowledge on how to start it

Containers wrap up everything we need: environment and how to start and run our program

## Kubernetes

It is a tool for running a bunch of containers
We give it configuration to describe how we want our containers to run and interact with each other

KB creates clusters (sets of virtual machines).
All these machines are refered as Nodes (VMs) and they are managed by a Master
KB eases the communication between all the nodes

KB setup
for windows/mac, easy, activate it in DockerDesktop

Open a new terminal window
> kubectl version -> should give you something

Give a config file to the master (K8 cluster) (1/run 2 copies of Posts 2/allow copies of Posts to be access. from network)
A pod can be a container or contains multiple ones
K8 will create a Deployment for the pods.
If one of the pods crash, the **Deployment** will make sure it restarts
K8 create something called **Service** (allow copies of Posts to be accesssible from network)
(running pods inside the cluster with networking)
Then if you add an EventBus pod, it will pass by the Service to reach out the container.

### Kubernetes terminology

Cluster: collection of nodes + a master to manage them
Node: a Virtual Machne that wil run our containers
Pod: More or less a running container, can contain multiple containers
Deployment: Monitors a set of pods running & restarting
Service: Provides an easy url to access a running container

### K8 config file

yaml syntax
tells about Deployments, Pods and Services (Objects)
Use config files to create Objects! Even if the doc tells different

### Create a pod

A pod running the Posts image
First rebuild the image
> docker build -t fffjacquier/posts:0.0.1 .

Then create an /infra/k8s/posts.yaml file
apiVersion: v1
kind: Pod
metadata: 
  name: posts
spec: 
  containers: 
    - name: posts
      image: fffjacquier/posts:0.0.1
      
Then in your terminal, tell to use this config to create a new Object
> cd ../infra/k8s
> kubectl apply -f posts.yaml
It will return: "pod/posts created"

to inspect the state of the cluster:
> kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
posts   1/1     Running   0          97s

If your pods are showing ErrImagePull, ErrImageNeverPull, or ImagePullBackOff errors after running kubectl apply, the simplest solution is to provide an imagePullPolicy to the pod.

First, run kubectl delete -f infra/k8s/

Then, update your pod manifest:

    spec:
      containers:
        - name: posts
          image: cygnet/posts:0.0.1
          imagePullPolicy: Never

Then, run kubectl apply -f infra/k8s/

This will ensure that Kubernetes will use the image built locally from your image cache instead of attempting to pull from a registry.

apiVersion : k8s is extensible, this specifies the set of objects we want k8s to look at
kind: the kind of object we want to create
metadata: name: config options for the kind of Object we want
spec: the exact attrs to apply to the object
  containers: we want many containers in a pod
    - name: posts :the - means it's an array
    image: the image we want to use

### Common kubernetes commands

docker is for runnin individual containers, and K8s is for running a bunch of containers together

kubectl get pods
kubectl exec -it podname cmd -> execute the cmd in a pod
kubectl logs podname -> prints logs from pod
kubectl delete pod podname -> delete a pod
kubectl apply - posts.yaml -> create a pod by processing the config
kubectl describe pd podname -> log info about a pod

> kubectl exec -it posts sh

Alias: k for kubectl
add alias k="kubectl" in your .bahsrc or .zshrc

### Deployments

K8s objects that manage a set of identical pods
like restart if crash or handle change version

apiVersion: apps/v1
kind: Deployment
metadata:
  name: posts-depl
spec:
  replicas: 1 # nb of pods we want
  selector:
    matchLabels:
      app: posts
  template:
    metadata:
      labels:
        app: posts
    spec:
      containers:
        - name: posts
          image: fffjacquier/posts:0.0.1

> k apply -f posts-depl.yaml 
deployment.apps/posts-depl created


> k get deployments
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
posts-depl   1/1     1            1           47s

if we delete the second pods, it is deleted and restarted after

> k describe deployment <depl name>

> k delete deployment posts-depl

### Update the Image used by a deployment

method 1: make changes, rebuild the docker image with a new version (0.0.2), updat the version in the deployment yaml file
then run
> k apply -f posts-depl.yaml 
deployment.apps/posts-depl configured

method 2: avoid to update the depl yaml, always use latest version
in the post-depl, do use :latest or nothing for version number
update your code
rebuild the docker image
push the image to docker hub
run: > k rollout restart deployment <depl.name>

### Networking with services

Services provide networking between pods or from outside requests

types of service: cluster ip node port, load balancer, external name
Cluster IP: easy to remember url to access a pod in the cluster
Node port makes a pod accessible from outside the cluster (dev only)
Load Balancer: makes a pod accessible from outside the cluster
Ext name: redirects an in-cluster to a CNAME url

#### Node Port:

apiVersion: v1
kind: Service
metadata: 
  name: posts-srv
spec:
  type: NodePort
  selector:
    app: posts
  ports:
    - name: posts # whatever name you want
      protocol: TCP
      port: 4000
      targetPort: 4000 # the actual port your app expects


Browser page -> port 3xxx -> Node[ port 4000 -> node port service -> port 4000 -> Pod]
                nodePort             port from the service           targetPort 

> k apply -f post-srv.yaml 
service/posts-srv created

> k get services
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          6h13m
posts-srv    NodePort    10.99.165.249   <none>        4000:30833/TCP   5s

The nodePort here is 30833

> k describe service posts-srv
...
Selector:                 app=posts
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.99.165.249
IPs:                      10.99.165.249
Port:                     posts  4000/TCP
TargetPort:               4000/TCP
NodePort:                 posts  30833/TCP
Endpoints:                10.1.0.15:4000
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

To access a post pod now you can use: localhost:30833/posts (for dev purpose)

#### Cluster IP

To expose a pod inside a cluster to other pods
In our case, useful for the event bus we created.

create a pod for the bus
create a cluster ip service for each pod?

the posts pod will request the the cluster ip service of the event bus
the cluster ip service will ask to event bus pod
the event bus will send the answer to the cluster ip service for the posts pod
and the cluster ip service for posts will give it back to the posts pod

Build an image for the event bus
push image to docker hub
create a deployment for event bus

create a cluster ip service for event bus
you can colocate it in the deployment file by adding:
---
apiVersion: v1
kind: Service
metadata:
  name: event-bus-srv
spec:
  # type: ClusterIP # by default
  selector:
    app: event-bus
  ports:
    - name: event-bus
      protocol: TCP
      port: 4005
      targetPort: 4005

then apply it
> k apply -f event-bus-depl.yaml 
deployment.apps/event-bus-depl unchanged
service/event-bus-srv created

create the cluster ip service for the posts Pod by adding in the posts-depl
---
apiVersion: v1
kind: Service
metadata:
  name: posts-cip-srv # make sur the name is different from the one on nodePort
spec:
  selector:
    app: posts
  ports:
    - name: posts
      protocol: TCP
      port: 4000
      targetPort: 4000

**How to communicate between the services?**
the old event bus url was localhost:4005
now we need the address of the clusterIP service for the event bus

> k get services
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
event-bus-srv   ClusterIP   10.96.59.146    <none>        4005/TCP         7m12s
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP          6h40m
posts-cip-srv   ClusterIP   10.109.190.16   <none>        4000/TCP         13s
posts-srv       NodePort    10.99.165.249   <none>        4000:30833/TCP   26m

we'll request to http://event-bus-srv:4005
and from event bus to cluster ip posts: http://posts-cip-srv:4000

Apply to multiple at same time
> k apply -f .

Debug
> k describe pod nameofpod

#### Load balancer

For our client app, we will put it in a pod too.

How our app will reach out our posts pods?
Use a Load Balancer service that will access the cluster ip services

A LBS load balancer service tells K8s to reach out to its provider and provision
a load balancer. The goal is to get traffic into a single pod.

#### Ingress or Ingress Controller

A pod with a set of routing rules to distribute traffic to other services

cloud provider (AWS, GC, Azure):
  K8s:
    Some pod in
    The cluster
  
Config file for a load balancer service -> K8s
A LBS will tell our cluster to reach out to the cloud provider to provision a load balancer (outside the cluster)
with the goal of getting traffic into our pod.
We want rules in order to not directly request the Pod, but the cluster ID services we created
These rules can be written in an Ingress Controller (in the cluster) that will handle where to send the requests

#### Installing Ingress Nginx

with helm:
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace

without:
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml

This (ingress-nginx) will create a LoadBalancer Serice and an Ingress for us

#### Writing ingress rules

ingress-srv.yaml:
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-srv
spec:
  ingressClassName: nginx
  rules:
    - host: posts.com
      http:
        paths:
          - path: /posts #/posts/?(.*)/comments
            pathType: Prefix #ImplementationSpecific
            backend:
              service:
                name: posts-cip-srv
                port: 
                  number: 4000
                  

#### hosts file tweak

With K8s we can host many different apps at different domains
- host: posts.com -> we are telling that our app is at posts.com/posts
To make to work locally, we have to trick the computer with the hosts file.
in the hosts file, add at the end:
127.0.0.1 posts.com

The Ingress Controller have no idea about POST or GET requests, only on routes
so if we have a GET /posts -> Pod Query
if we have a POST /posts -> Pod Posts

How to have a unique path for this?
We have to change the POST /posts to /posts/create in the client

Painful to rebuild and repush every repo each time you make a change. Solution?
