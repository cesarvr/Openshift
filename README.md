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
        - [Exporting Images](#exporting_images)
        - [Deploying Server Application](#server_application)
     - [Exposing Services](#expose)
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

Umm, we have finish the deployment of our server app, but we still are not comunnicating with our server from the outside, to be honest we don't have even access from our guest machine. For this we need a combination of two Openshift objects Service and Router, in the next section we are going to explore how route request to our Pods.




<a name="expose"/>


### Exposing Services

Before we start exposing our server application to external traffic, we need to talk about some concepts:

- Label's: labels provide a easy way to organize our objects in the cluster, think of it as a way to group a set of objects, in the example above we choose to setup the label ```app: nodejs-app```.

- Services: OpenShift object that is in charge to redirect the traffic to our Pods, it work at cluster level, saying this, you should never target the IP of the Pod directly, always use a Service.   
  - **Why?** Because the Pod entities are ephemeral objects designed to die, moved around nodes, etc.. . Services works as an entity that keep tracks of them and offer a single point endpoint to contact your Pod.  

- Routers: This object redirect traffic from the outside to our Service, we need this object when we want to expose our Services to the exterior.


![Service-Router](https://github.com/cesarvr/Openshift/blob/master/assets/deploy-server.gif?raw=true)


To create a Service we need to create a definition in a template:

```yml
kind: Service
apiVersion: v1
metadata:
  name: HelloWorld-service
spec:
  selector:
    app: nodejs-app

  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```
