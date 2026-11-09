Hii Welcome to my Kubernetes repository.


In this Documentation i will be walking through my journey of understanding everything about kubernetes. From the problem it is built to solve, a view into architectural diagram and the each component part that makes up the integral system of distributed Highly Available, self healing system that makes orchestration of Workflows easier. 

This documentation is prepared to expose components, concepts, logics and the right iplementation of Kubernetes. in the end a reader is not just aware of the capabilities of kubernetes but also have deep knowledge of how the system operates under the hood. lreference to actual documentation links will also be shared along the way for reference and more deep dive into any part 

**What is Kubernetes **

Most people know kubernetes as a workflow orchestration tool(data workflow)thats a vague definition because by practical implementation there are tons of orchestration tools going by that definition alone. So what is an orchestration tool? Google defines an orchestration tool as a software that automates, coordinates, and manages complex, multi-step workflows across multiple computer systems, applications, and services. this means in any case of  workflow such as the popular ones ;like 
* Software Development (Continuous Integration / Continuous Deployment)
* Data Engineering & Analytics (Daily Financial Reporting)
* or even E-Commerce & Retail (Order-to-Delivery)

  these are workflows that require sequence of operations to be managed at a specific time under certain conditions and must not fail at any point or must be successful in the end


  Managing workflows like these requires coordination. This could actually be manually managed  if there are few missinfg parts involved. but what happens when there are 100s or thousands of tasks that need to be presicly coordinated? It definitly requires a system that eliminates manual coordination and must precisly give optimal result while dynamically considering the underlying infrastuctures and ther conditions that must be met for a successflw workflow.
  This is the kind o problem kubernetes solves.

  Kubernetes accordinfg the docunmentation is a **portable**, **extensible**, open source platform for managing **containerized **workloads and services that facilitate both **declarative configuration and automation.**

  Not to worry we will upack these concepts in a bit
  first we start with the word containerized.

  Containerization is the process of packaging an applications or workloads together with its dependencies, libraries, and required runtime into a container image so that it can run consistently across different environments. Think of Docker( This assumes you have basic knowledge of Docker and the problem it is created to solve). To further break this down, imagine you have an ETL python application that is required to extract transform and load data into a Postgres DB both tools require dependencies such a s libraries and credentials for this to work. If these dpependencies change at any given time due to some differences then the Whole workflow fails. to achieve these the application and database are both containerized that is, packaged alonsisde their dependencies or credentials to be shipped anywhere. lol i guess the reason why docker has a shark logo.

  One may wonder where does Kubernetes come in then if were here talking about Docker. 
  Well we al know a Syetm or tool is created to solve the shortcominfg of another . In this case Docker

  Docker can build container images and run containers. But imagine having hundreds of containers distributed across multiple servers.
Servers, in this context, can be likened to different computers, each having its own computing resources, such as CPU, memory (RAM), storage, and networking capabilities.

You would need to determine where they should run, manage failed containers, scale applications, and manage communication between them. This is alot of Hats to wear for Dockwr or worse case an ordinary human. 
This is the problem Kubernetes was built to solve 

Tying this back to our definition from the official Kubernetes documentation, Kubernetes is a portable and extensible platform that can run across different computing environments and can accommodate additional capabilities.

it helps us manage the complexities and unpleasant surprises that come with running hundreds of containerized applications across multiple servers.

It ensures that these workloads are properly coordinated, it supports communication between these workloads, responds to failures, and continuously works towards maintaining the desired state of our applications.

**In simpler terms, Kubernetes takes on much of the responsibility of managing and coordinating our containerized workloads, so we don't have to do everything manually.**

We can go on and on about how amazing Kubernetes is but if we do not get a mental pictur of how Kubernetes works we may not fully grasp the full capabilitues of this tool. 
therefore lets take a view of the architecture of Kubernetes

**Kubernetes architectural View**

On a high level, there are two major components of Kubernetes
* The Master (Control plane) and
* Worker

The combinatiomn of these components make uop the Kubernetes Cluster

The Control pane initially called Master is the brain of Kubernetes. It is the one responsible for Planning how to sucessfully run these workload while the Worker is responsible for Exxecuting the actual tasks involved. But these two do not work in isolation neither are they just one component.
the Control pane or master  in itself has various components that makes up the master so does the worker 

We will go a step deeper into each of them to uncover how they function under the hood

** Diagram showing worker and mastr nodes **

** Control plane **

The control plane's components are the ones responsible for making  decisions about the cluster as well as detecting and responding to cluster events ( Expected or unexpected )
 
 ### components of the control plane ###

 **etcd** This can be refeered to  as the memory of the control plane
It stores information about the Kubernetes cluster  including every resources kubernetes needs to function such as Pods, Deployments, Services, Nodes, ConfigMaps, and their desired and reported current states.(Dont worry about these we'll explore them in details later on.)  

Now, if you understand what this means, you'll realize how important etcd is to Kubernetes. Lets look at a quic example. Imagine you want Kubernetes to replicate your ETL container in 3 different places. the etcd holds this information in its metastore so that the component part responsible to achieve this rads this information and carrys out the task. The acto f writing your desired state is called declaration of state and etcd is responsible for holding that informatiom

**kube-apiserver**
If youre familair with api servers this shoul be easy one to recognise. fundamenally, an API server acts as a middle man on an errand rather than you making a request directly to an application, the api helps take that request and returns with the result. this is important for security and safety especially for our etcd that holds very special and importan information. the Kube-apiserver is responsible for exposing the Kubernetes API. ot is the first entry point of the control plane. Every request coming to the control plane hits the kube-apiserver first 

**Kube-scheduler**

Before we talk about kube-scheduler its worthy  to mention pod here. it simply reprezens a set of running containers in the cluster so the kube scheduler is the one responsible for watching newly created pods and assign them to worker. there are so many factors that affect this scheduling including avaialable resources. like RAM CPU etc

** kube-controller-manager**
just as we briefly explained pods in the previous session let us also do justice to controller. 
A controller is a control loop that watches the shared state of the cluster( worker annd master) it watches this process through the apiserver and makes changes from the current state to the desired state. Simply put, the controller keeps the whole cluster under survillance looking out for when the actual state of this cluster is changing from the desired state. Infact controller is our Military Guard commander lol

so what then is the work of kube-controller manager?
It simply manages the controller. Because each controller runs as a separate process, the manager controls them as though they are a single unit of process. 

** cloud-controller-manager**
With the mention of the word cloud, we can tell its the cloud manager for kubernetes.  it embeds cloud specific control logic into the process. it allows  you link your cluster into your choice cloud provider's API, and separates out the components that interact with that cloud platform from components that only interact with your cluster.



## Worker or Node components ##

these  components are seen in every node, they help in  maintaining running or active pods and provide the Kubernetes runtime environment.

Again, we're doing a deep dive into each component starting with the kubelet

** Kubelet**  I practically love this one just the way it sounds cute like a baby kube lol
Cute as it may sound it is a powerful component of the node(worker machine) It is an agent that runs on each node in the cluster. 
It makes sure that containers are running in a Pod.
rememer we said pod represents a set of running containers which means ideally containers run inside a pod. The kubelet is responsible making sure this process runs smoothly.
it does this through an operation that takes set of resources (PodSpecs) that are provided through different mechanisms and ensures that the containers described in those resources (PodSpecs) are running and healthy. The kubelet does not manage containers that were not created by Kubernetes. it is Kubernetes-native containers aware

**kube-proxy**
Its just but not insignificant a network proxy that runs on each node in your cluster. It maintains network rules on nodes. These rules allow network communication to the Pods from inside or outside of the cluster.

** Container runtime**
Another of my favourite 
P.S i choose my favourites just based off how cute they sound. Please dont take it personal lol

It is a fundamental component that empowers Kubernetes to run containers effectively.This is not kubelet neither is it pod. It is responsible for managing the execution and lifecycle of containers within the Kubernetes environment.



Some addons worthy of mention here are:





