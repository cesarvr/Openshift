# Openshift

### Getting Started

They are various options to get started with OpenShift: 

<!--ts-->
   * [Openshift Interactive](#interactive)
   * [OC Cluster Up](#ocup)
   * [Minishift](#minishift)
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








