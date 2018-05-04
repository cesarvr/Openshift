## Openshift

### Trying Openshift

They are various options to get started with OpenShift: 

#### OC Cluster Up

##### Instructions for MacOSX 

First we need to install [Docker](https://download.docker.com/mac/stable/1.13.1.15353/Docker.dmg), at the moment of writing this document *oc-client* work best with this old version.

Add insecure registry: 
![Openshift UI](https://github.com/cesarvr/Openshift/blob/master/assets/insecure.png?raw=true)


Install Socat using [Homebrew](https://brew.sh/), ``` brew install socat```


Now download [oc-client](https://github.com/openshift/origin/releases), extract and add it into your path. 

If everything is fine you can try: 

```sh 
  oc version

  #oc v3.9.0+191fece
  #kubernetes v1.9.1+a0ce1bc657
  #features: Basic-Auth
```
