# Openshift

### Getting Started

They are various options to get started with OpenShift: 

<!--ts-->
   * Getting Started
     - [Openshift Interactive](#interactive)
     - [OC Cluster Up](#ocup)
     - [Minishift](#minishift)
   * Application Development
     - [Login And First Project/Namespace](#first)
     - [Your First Pod](#pod)
     - [Deployment Config](#deploy)
<!--te-->

<a name="interactive"/>

#### Openshift Interactive

This [portal](https://learn.openshift.com/) is a great start to get familiar with Openshift.

<a name="ocup"/>

#### Cluster Up


##### Instructions for MacOSX 

First we need to install [Docker](https://download.docker.com/mac/stable/1.13.1.15353/Docker.dmg), at the moment of writing this document *oc-client* work best with this old version.

Add insecure registry: 
![Openshift UI](https://github.com/cesarvr/Openshift/blob/master/assets/insecure.png?raw=true)


Install Socat using [Homebrew](https://brew.sh/), ``` brew install socat```


Now download [oc-client](https://github.com/openshift/origin/releases), extract and add it into your path. 

```sh 
export PATH=$HOME/folder-with-oc-client/:$PATH
```

If everything is fine, you should be able to run this: 

```sh 
  oc version

  #oc v3.9.0+191fece
  #kubernetes v1.9.1+a0ce1bc657
  #features: Basic-Auth
```

Now you can try and create your Openshift cluster: 

```sh 
oc cluster up
``` 

We should get the following message:

```sh 
Starting OpenShift using openshift/origin:v3.7.1 ...
OpenShift server started.

The server is accessible via web console at:
    https://127.0.0.1:8443

You are logged in as:
    User:     developer
    Password: <any value>

To login as administrator:
    oc login -u system:admin
```

<a name="minishift"/>

### Minishift 

In some cases using oc-cli method can be complicated to setup, [Minishift](https://github.com/minishift/minishift#getting-started) offers a VM solution that encapsulate all this complexity. 


<a name="first"/>

### Login And First Project/Namespace

Once you have setup your cluster you can start playing with Openshift, first we can start by login in, to do this we need to write the following command: 

``` 
  oc login -u <username>
```

or if you want to get a interactive login: 

```
 oc login
```

Project/Namespaces are the way Openshift divide the cluster between users, basically all your object should be defined under a Openshift namespace, to create a new project you should do.

```
 oc new-project hello-world
```

This will create a new namespace called **hello-world**. 

[![asciicast](https://asciinema.org/a/J0ASJwIMlReglXQpbxz2QDZL7.png)](https://asciinema.org/a/J0ASJwIMlReglXQpbxz2QDZL7)



<a name="pod"/>


### Your First Pod

The Pod is the minimal building blog in Openshift, represent a single application from the perspective of Openshift. But in the inside it can contain multiple resources containers, storage resources, etc.

To create we just need to create a small template: 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Openshift! && sleep 3600']
```


Consider this template like a recipe, it basically tells Openshift what to do for you, in this case it specifies a resource of the **kind** Pod, and put some name to it *myapp-pod*, then we describe what we want to run inside, we said, we want a **busybox** container and we want to run to run some shell **command** in it.  

Then we need to pass this to Openshift: 

```shell
oc create -f pod.yml       #pod "myapp-pod" created

#or you can grab the template from the cloud.
 
oc create -f https://raw.githubusercontent.com/cesarvr/Openshift/master/templates/pod.yml
``` 

Then we can ask Openshift, about the state of the resource by doing: 

```shell
oc get pods                                                               
```

We should see our Pod up and running. 

```bash
  NAME        READY     STATUS    RESTARTS   AGE
  myapp-pod   1/1       Running   0          4d
```





[![asciicast](https://asciinema.org/a/cLPabsEksE1J7GcD8VruXMp6b.png)](https://asciinema.org/a/cLPabsEksE1J7GcD8VruXMp6b)



<a name="deploy"/>


### Your First Deployment

In the last section, we end up running our first pod, but there some catch, we are not able to scale up or down our application, also we are not able to update the state or rolling out a new version. For been able to do that we need another entity called [DeploymentConfig](https://docs.openshift.com/enterprise/3.0/dev_guide/deployments.html). 

[DeploymentConfig] is a way to describe a desired state for our application, we can scale, downscale, pause deployments, roll-back and roll-out.

Let's scale our app, to do this we just need to create a Deployment template: 

```yml 
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: hello-dev
spec:
  selector:
    matchLabels:
      app: hello-dev
  replicas: 2 # tells deployment to run 2 pods matching the template
  template: 
    metadata:
      labels:
        app: hello-dev
    spec:
      containers:
      - name: myapp-container
        image: busybox
        command: ['sh', '-c', 'echo Hello World! && sleep 13600']
```

The template here is very similar to the one used to create the Pod: 

```yml 
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: hello-dev
```
Here we said we want to target the **apps/v1beta** endpoint of Openshift, and inside this endpoint we want the Deployment object. 

```yml 
spec:
  selector:
    matchLabels:
      app: hello-dev
  replicas: 2 # tells deployment to run 1 pods matching the template
  template: 
    metadata:
      labels:
        app: hello-dev
```
Here we describe the state we want for our application, we specify we want two replicas, once the  Deployment Config is created, Openshift will check the current state of the system (0 replicas) versus the expected state (2 replicas), then it will increase the number of Pods to achieve the desired state.   

```
spec:
  containers:
    - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello World! && sleep 13600']
```

This is the exact same code as our previous example, we just want to say hello and sleep. 

To load this Deployment template, is similar to what we did with our Pod last time. 

```
 oc create -f deploy.yml
```


#### Deployment

![Image of Yaktocat](https://github.com/cesarvr/Openshift/blob/master/assets/deploy.gif?raw=true)



### Preparation 
We are going to deploy simple [Node.js](https://nodejs.org/en/), the best way to do this is to use a BuilderConfig but for know and for the sake of learning we going to do it with a Deployment template and an external image.  

Exporting an image to your cluster is easy, in this case we want to export a nodejs image an [alpine-node](https://hub.docker.com/r/mhart/alpine-node/) work well for this purpose, to import it we just run: 

```sh
oc import-image alpine-node:latest --from=docker.io/mhart/alpine-node --confirm
```

This will import the image into our cluster. 

Now we need to create our deployment template. 

```sh

```












