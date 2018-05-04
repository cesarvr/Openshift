# Openshift

### Getting Started

They are various options to get started with OpenShift: 

<!--ts-->
   * [Openshift Interactive](#interactive)
   * [OC Cluster Up](#ocup)
   * [Minishift](#minishift)
   * [Login And First Project/Namespace](#first)
   * [Your First Pod](#pod)
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


Consider this template like a recipe, it basically tells Openshift what to do for you, in this case it specifies a resource of the **kind** Pod, and put some name to it *myapp-pod*, then we describe what we want to run inside, we said, we want a **busybox** container and we want to run to run some shell commands in it.  

Then we need to pass this to Openshift: 

```
 oc create -f pod.yml       #pod "myapp-pod" created
 
 # or you can grab the template from the cloud.
 
 oc create -f https://raw.githubusercontent.com/cesarvr/Openshift/master/templates/pod.yml  #pod "myapp-pod" created
``` 

Then we can ask Openshift, about the state of the resource by doing: 

```
oc get pods                                                               

```

We should see our Pod up and running. 

```
  NAME        READY     STATUS    RESTARTS   AGE
  myapp-pod   1/1       Running   0          4d
```





[![asciicast](https://asciinema.org/a/cLPabsEksE1J7GcD8VruXMp6b.png)](https://asciinema.org/a/cLPabsEksE1J7GcD8VruXMp6b)




