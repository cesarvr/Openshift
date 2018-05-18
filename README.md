# Openshift Developers Guide


The purpose of this guide is to introduce developers, in a practical way, to the concepts, components and tooling of the OpenShift ecosystem.

### Getting Started

They are various options to get started with Openshift:

<!--ts-->
   * Getting Started
     - [Openshift Interactive](#interactive)
     - [OC Cluster Up](#ocup)
     - [Minishift](#minishift)
   * Application Development
     - [Login And First Project/Namespace](#first)
     - [Your First Pod](#pod)
     - [Deployment Config](#deploy)
        - [Exporting Images](#exporting_images)
        - [Deploying Server Application](#server_application)
     - [Exposing Our Application](#expose)
        - [Service](#service)
        - [Router](#router)
     - [Openshift Application Templates](#oat)
        - [Deploying Java](#java)
        - [Deploying NodeJS](#node)
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
Starting Openshift using openshift/origin:v3.7.1 ...
Openshift server started.

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

### Before We Start

Openshift offers three ways to communicate, calling HTTP request, Openshift console which is an admin portal where you can graphically specify what you want and the [oc-client](https://github.com/openshift/origin/releases) which is a robust terminal command line tool where you can summit the actions you want to accomplish.

In this guide we are going to use mostly the [oc-client](https://github.com/openshift/origin/releases), just download the client and make it available to your PATH and you should be ready to go.    

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
Here we describe the state we want for our application, we specify we want two replicas, once the  Deployment Config is created, scheduler will check the current state of the system (0 replicas) versus the expected state (2 replicas), then it will increase the number of Pods to achieve the desired state.   

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

![deployment](https://github.com/cesarvr/Openshift/blob/master/assets/oc-deployment-2.gif?raw=true)


<a name="exporting_images"/>

### Exporting Images
We are going to deploy simple [Node.js](https://nodejs.org/en/) application, the best way to do this is to use a BuilderConfig but for now and for the sake of learning we going use a external image and update our Deployment Config.  

To achieve this goal we are going to export an image to our cluster, for this we need a Openshift object called **ImageStream**, which allow us to monitor and perform actions based in images tag changes.

To export the image we run the following code

```sh
oc import-image alpine-node:latest --from=docker.io/mhart/alpine-node --confirm
```

We can check the status of the ImageStream by running:

```sh
oc get is

# NAME          DOCKER REPO                               TAGS      
# alpine-node   172.30.1.1:5000/hello-world/alpine-node   latest
```

Some explanations:
 - **Name**
   - **Name**: Name of the ImageStream object.  
 - **Docker Repo URL**
   - **172.30.1.1:5000**: This is the URL for the Docker registry in your Openshift installation.
   - **hello-world**: This is the project/namespace, this means that this image is pullable from this project/namespace only, which is good, because we don't want to clutter the Docker registry for others.
   - **alpine-node**: It's the actual image our ImageStream is pointing to.  
 - **Tags**
   - **tags**: This is the [Docker tag](https://docs.docker.com/engine/reference/commandline/tag/), that our ImageStream is monitoring for changes.  


Now that we have exported our image, we can update our deployment configuration:  


<a name="server_application"/>

### Deploying Server Application

Once we got our image imported into our cluster is ready to use, next step is to modify our previously created deployment template.

We start with this template:

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

Here we need to modify the image section, where we going to replace **busybox** with the URL of our imported image, to get the URL just write ```oc get is``` and copy/paste the URL that appear in the section called **Docker Repo**. Next, we need to change the command and add some instruction to execute our Node.js server.

We should change the container:

```yml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: hello-dev
spec:
  selector:
    matchLabels:
      app: nodejs-app
  replicas: 2
  template:
    metadata:
      labels:
        app: nodejs-app
    spec:
      containers:
      - name: myapp-container
        image: 172.30.1.1:5000/hello-world/alpine-node
        command: ['node', '-e']
        args:
          - require('http').createServer((req, res) => { res.end('Hello World') }).listen(8080)
        ports:
        - containerPort: 8080
```

This will execute ```node -e``` with those **args**, that snippet its just a simple Node.js HTTP server program that would listen in port 8080 and will respond a ```Hello World```. We also going to open a port for our Pod, in this case **8080**.

We save this modification as ``` deploy-server.yml ```.

Then we load the template:

```
oc create -f deploy-server.yml
```

![deployment server](https://github.com/cesarvr/Openshift/blob/master/assets/deploy-server.gif?raw=true)

If you have the old version of the deployment, you need to delete it first:

```
oc delete deployment hello-dev
```

Umm, we have finish the deployment of our server app, but we still are not communicating with our server from the outside, to be honest we don't have even access from our guest machine. For this we need a combination of two Openshift objects Service and Router, in the next section we are going to explore how route request to our Pods.


<a name="expose"/>


## Exposing Our Application

Before we start exposing our server application to external traffic, we need to talk about some concepts:

- Labels: labels provide a easy way to organize our objects in the cluster, think of it as a way to group a set of objects, in the example above we choose to setup the label ```app: nodejs-app```.

- Services: Openshift object that is in charge to redirect the traffic to our Pods, it work at cluster level, saying this, you should never target the IP of the Pod directly, always use a Service.   
  - **Why?** Because the Pod entities are ephemeral objects designed to be disposable, moved on-demand around the cluster. Services works as an entity that keep tracks of them and offer a single point endpoint to contact your Pod.  

- Routers: This object redirect traffic from the outside to our Service. We need this object when we want to expose our Services to the exterior.

<a name="service"/>

### Services

To create a Service we need to create a definition in a template:

```yml
kind: Service
apiVersion: v1
metadata:
  name: helloworld
spec:
  selector:
    app: nodejs-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

In this definition we are telling Openshift that we want a Service object that send traffic to objects with the tag ```app:nodejs-app```, also we want to take traffic from **port 80** and we want to forward the traffic to port 8080.

Pay special attention to the **selector app:nodejs-app**, this basically tell the Service object to query objects in the cluster that match those labels, once he find it, it will start to direct traffic between them.

To create the service:

```sh
oc create -f service.yml
oc get service

#NAME         CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
#helloworld   172.30.106.249   <none>        80/TCP    4h
```

<a name="router"/>

### Router

The [Router](https://docs.openshift.com/container-platform/3.7/install_config/router/index.html), as mentioned before is the object to direct traffic into the cluster, creating it is very simple, we can explicitly expose the service using ```oc expose```.

```sh
oc expose svc helloworld
oc get route

#NAME         HOST/PORT                                 PATH      SERVICES     PORT      TERMINATION   WILDCARD
#helloworld   helloworld-hello-world.127.0.0.1.nip.io             helloworld   8080                    None
```

Now we can talk with our Pods from the outside.

```sh
curl helloworld-hello-world.127.0.0.1.nip.io
#Hello World%
```

![Service-Router](https://github.com/cesarvr/Openshift/blob/master/assets/expose-modified-.gif?raw=true)

<a name="oat"/>

## Openshift Application Templates

Now that we know the fundamental building blocks to deploy our application, is time to introduce an alternative way to deploy applications. Instead of creating sophisticated scripts to create the objects your self, you can use Openshift application templates.

This templates is a way to automatise the creation of Objects, we just need to follow the rules of our particular programming language. In this section I'll explain how to deploy applications for two programming languages Java and NodeJS as this are the languages I know most, if you know how to deploy in another one feel free to contribute.

<a name="java"/>

### Deploying A Java Application

For deploying in Java at the moment of writing this you project need to use [Maven Package Manager](https://maven.apache.org/), and is easy, for this particular example I would use this [Hello World](https://github.com/cesarvr/Spring-Boot) Spring Boot Project, that I copy from [this place](https://spring.io/guides/gs/spring-boot/).

To make it work is necessary just to modify the [pom.xml](https://github.com/cesarvr/Spring-Boot/blob/master/pom.xml#L23) and add Tomcat as the embedded servlet container and we need to tell maven that we want to build a [WAR](https://github.com/cesarvr/Spring-Boot/blob/master/pom.xml#L10) file.   

If your project is store in a git repo in the cloud you can use the Openshift console:

![Deploying Java](https://raw.githubusercontent.com/cesarvr/Spring-Boot/master/docs/hello.gif)


<a name="node"/>

### Deploying A NodeJS Application

For NodeJS is less drama, we just need to define two entries in our package.json ```test``` and ``` start ```.

We need a **package.json**.
```js

{
  "name": "hello",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "node app.js",
    "test": "echo \"Error: no test specified\" && exit 0"
  },
  "author": "",
  "license": "ISC"
}
```

And the source code.

```js
require('http').createServer((req, res) => {
    res.end('Hello World')
}).listen(8080)
```

We go to the console, create a new project and choose Node.JS then we point to our git repo and thats it. If you want a project to test you can use [this project](https://github.com/cesarvr/hello-world-nodejs).


![Deploying Java](https://github.com/cesarvr/Openshift/blob/master/assets/new-app-nodejs.gif?raw=true)


In the command line we can create project using ``` oc new-app ``` let see the example:

```sh
oc new-app nodejs:6~https://github.com/cesarvr/hello-world-nodejs # NodeJS

oc new-app wildfly:10.0~https://github.com/cesarvr/Spring-Boot --name=spring-boot # Java
```

It will create the exact same components and auto magically deploy your application.
