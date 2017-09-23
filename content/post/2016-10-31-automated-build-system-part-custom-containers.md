---
author: Steve Bargelt
categories:
- devops
date: 2016-10-31T00:00:00Z
excerpt_separator: <!--more-->
featured_image: /assets/azure_docker_jenkins.png
tags:
- jenkins
- cd
- ci
- docker
- azure
- riot games
title: 'Automated Build System Part 02: Custom Jenkins Container and Ephemeral Slave
  Nodes'
url: /2016/10/31/automated-build-system-part-custom-containers/
---
### Introduction

Welcome back to my series where we are creating an Automated Build System with Docker and Jenkins in Azure. In part one we setup a Linux  VM in azure, installed Docker on that VM and setup secure communication to the Docker host. If you missed it, go check it out, this tutorial assumes you've worked through Part 1.  

<!--more-->

In this tutorial we will setup Jenkins in a Docker container, using a custom image and configure Jenkins to spin up slave build environments that are Docker containers, on demand and then remove them to clean up. Very cool stuff! Once again I need to give credit to Maxfield Stewart of Riot Games. I've based this on his [DockerCon 2016 Talk]() and his [tutorial on github](). 

Here is a diagram of what we will have at the end of this part of the tutorial:

{{< blog-img src="/assets/buildSystem_02" >}}

The video tutorial:
{{< youtube NN3d_fZO87w >}}

### The bootstrap code

My [jenkinsDocker](https://github.com/stevebargelt/jenkinsDocker) project out on github  is what we'll be using to bootstrap our system. It includes Dockerfile image definitions for all of the images we need to run our system. I'm not going to deep-dive into the Docker files in this tutorial but here's a quick overview of the contents. 

* Jenkins Master is a custom Jenkins image that we will use as the backbone of our automated build system
* Jenkins Data (for persistent data storage or our Jenkins settings)
* NGINX for our web proxy
* letsencrypt we will use to setup TLS/SSL on our system in part 3
* Finally two sample ephemeral slave node images jenkins-slave and jenkins-dotnetcore-slave

Remember, in our final system software developers or more likely, development teams, will be responsible for creating their own build environment definitions but in order to make sure our system is functioning I've included these two starter examples. 

I encourage you to check out the code and also please do check out Maxfield Stewart's excellent tutorial if you want more detail. 

### Our Custom Jenkins Docker Container

SSH into our VM and clone the jenkinsDocker repo:

```shell
ssh dockeruser@dockerbuild.hairbrained-apps.com

git clone https://github.com/stevebargelt/jenkinsDocker.git

Cloning into 'jenkinsDocker'...
remote: Counting objects: 376, done.
remote: Total 376 (delta 0), reused 0 (delta 0), pack-reused 376
Receiving objects: 100% (376/376), 70.71 KiB | 0 bytes/s, done.
Resolving deltas: 100% (196/196), done.
Checking connectivity... done.

cd jenkinsDocker
```

Then use docker-compose to build and startup our base containers

```shell
docker-compose -p jenkins up -d nginx data master
```

```shell
docker ps -a
```

{{< blog-img src="/assets/abs-02-containers-ps-a" alt="shell showing results of docker ps -a" >}}

Pop open a browser with an address (domain, IP address) that points to your VM (http://dockerbuild.harebrained-apps.com for me) and you will see:

{{< blog-img src="/assets/abs-02-unlock-jenkins" alt="Initial jenkins web view asking for initial password">}}

Our *custom* Jenkins image is running in Docker on our VM in Azure. Great, but how do we get that initial password from the container?

The message tells us that the password can be found at /var/jenkins_home/secrets/initialAdminPassword.
Makes sense - if we inspect the jenkins-master Dockerfile we see that JENKINS_HOME is mapped to /var/jenkins_home

{{< blog-img src="/assets/abs-02-dockerfile-jenkins-home" alt="Image showing mapping of JENKINS_HOME to var jenkins home" >}}

How do we get to that folder in the container? We can docker exec against the container - using the cat command to output the password to the console. Our container is jenkins-master_1:

```shell
docker exec jenkins_master_1 cat /var/jenkins_home/secrets/initialAdminPassword
```

{{< blog-img src="/assets/abs-02-term-get-init-password" >}}

Copy and paste that password into out web browser and click Continue

### Jenkins Initialization

Click on Install Suggested Plugins

{{< blog-img src="/assets/abs-02-install-suggested-plugins" >}}

Think it's a big enough button? Jenkins will proceed to install the suggested plugins

{{< blog-img src="/assets/abs-02-plugins-installing" >}}

After the plugin install is complete, click Continue and create the initial admin user:
{{< blog-img src="/assets/abs-02-jenkins-create-first-admin" >}}

Click Save and Finish

Click Start Using Jenkins

### Jenkins TLS Certificate Credentials

So now we need to add those client certs to Jenkins so it can securely talk to our Docker host. Jenkins will be talking to our dockerhost to spin up and remove containers as needed so it need to be able to communicate securly with the host.

On the jenkins landing page click on “Credentials”

Then:

1. Click "System"
1. Click on “Add credentials”
1. Click "Global credentials (unrestricted)"
1. Click "Add Credentials"
1. Select "Docker Host Certificate Authentication" in the "Kind" dropdown

We need to copy and paste the Client Key, Client Certificate and Server CA Certificate into the web interface. In Part 1 we created all of our certs in ~/tldBuild. I'm going to use pbcopy to copy from the files onto my clipboard, one at a time and paste them into the web interface.

```shell
cd ~/tlsBuild 
pbcopy < ~/tlsBuild/key.pem
pbcopy < ~/tlsBuild/cert.pem
pbcopy < ~/tlsBuild/ca.pem
```

> NOTE: an earlier version of this post incorrectly had server-cert.pem in that last slot. It should be ca.pem. In Part 05 this will cause issues if not changed. (Updated 2017-08-23)

I'm going to add an ID of dockerTLS and a Description of Docker TLS Certs. This is what my final TLS credential screen looks like:

{{< blog-img src="/assets/abs-02-jenkins-paste-tls" >}}

Click OK

### Add Cloud Config

Our custom Jenkins_Master Dockerfile automatically installs [Yet Another Docker Jenkins plugin](https://github.com/KostyaSha/yet-another-docker-plugin) which it what we will use to control our ephemeral slaves.   

In jenkins-master Dockerfile:
{{< blog-img src="/assets/abs-02-docekrfile-plugins" >}}

Plugins.sh is in the github repository (along with plugins.txt):
{{< blog-img src="/assets/abs-02-install-plugins" >}}
If you want to auto-install Jenkins plugins add them to plugins.txt:
{{< blog-img src="/assets/abs-02-plugins-txt" >}}

Next we need to add a Cloud Config to Jenkins

1. Manage Jenkins
1. Configure System
1. Cloud 
1. From the Add a New Cloud dropdown select Yet Another Docker
{{< blog-img src="/assets/abs-02-jenkins-yad" >}}

Cloud Settings: 

1. Name: AzureJenkins
1. Docker URL: tcp://10.0.0.4:2376 (or the internal IP of your VM - you can find in the Azure portal) 
1. Docker API Version: 1.23
1. Host Credentials: dockerTLS (the certs credentials we added in the previous step)

{{< blog-img src="/assets/abs-02-jenkins-cloud" >}}

Click Test Connection

{{< blog-img src="/assets/abs-02-jenkins-cloud-test" >}}

You should see a confirmation that Jenkins was able to talk to your Docker host. 

Click Apply

### Another Inbound Security Rule

Jenkins will use the secure connection we setup to communicate to the dockerhost but once the slave node(s) are up Jenkins will use JNLP to do the actually build-job work so we need to open a port in our system to allow that traffic.

Back in the Azure portal:

1. Click on dockerBuild (Resource Group) 
1. dockerbuild-nsg (Network Security Group)
1. Inbound Security Rules
1. Add 

We need to open a port Jenkins to communicate with slaves over JNLP. We'll call this **allow-jenkins-jnlp** and since this is not a preconfigured service we have to select: 

* Any as the protocol
* Port 50000

{{< blog-img src="/assets/abs-02-azure-new-inbound" >}}

Click Ok

### Add a Docker Template

In Part 4 we will automate the creating of Docker templates but for now lets do it manually to test our system and to get the feel for exactly what we need to autoamte. Back in our Jenkins web interface, Manage Jenkins, Configure System... at the bottom of the Could Config we created...

* Click on the “Add Docker Template” drop down
* Select “Docker Template”
* For the Docker Image Field enter: “jenkins_slave”
* Pull Never
* For “Labels” add “testslave”
* For “usage” change the selection to “Only build jobs with label restrictions matching this node”
* Under "Remove Container Settings" check "Remove Volume"
* Click “Save” at the bottom of the configuration page

{{< blog-img src="/assets/abs-02-jenkins-dockertemplate" >}}

You can think of the Docker Template as linking an Jenkins slave node Docker container/image  to a label. So basically what we are saying here is when a build job with the label "testslave" gets kicked off, start a container with the Docker image jenkins_slave. This is how developers will link thier builds to their Docker images. So one team may have the labels "teamSteam, dotnetcore1.0" that uses the Docker image they created named "jenkins-slave-steam-dotnetcore1.0" 

### Testing Our System
So lets test out our system by creating a very simple build job.

On the Jenkins landing page click “create new jobs”
For Item name enter “testjob”
Select “Pipeline”

{{< blog-img src="/assets/abs-02-jenkins-newjob" >}}

The only thing we are going to change is to add the following for the pipeline script:

```groovy
node ('testslave') {

  stage ('Stage 1') {
  	sh 'echo "Hello World from an Ephemeral Jenkins node!"'
  }
}
```

{{< blog-img src="/assets/abs-02-testjob-pipeline" >}}

The "node" in a pipeline script is the label for that part of the script. So we are telling Jenkins to label this "testslave" which we configured to kick off the jenkins_slave docker container. 

Click “Save”

### Build our Test Save Image

We need to build our jenkins-slave image so that Jenkins can startup a container based off that image once it is called upon by our test build job. SSH'd into our VM, 

```shell
cd ~/jenkinsDocker
docker-compose -p jenkins build slave
```

NOTE: These are not lightweight production containers, they are build environments as containers... often with the full JDK, full .Net Core framework, test application, test frameworks, and the like. These containers can be beasts!

### Build with an ephemeral container

Back in the Jenkins Web Interface click on "testjob"

Click Build Now

We can pop over to our terminal / SSHd into our host to see if we can catch the Docker container in action:

```shell
$ docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                  
b64359bc2a31        jenkins_slave       "/bin/sh -cxe 'cat <<"   1 seconds ago       Up Less than a second       
e65e5807788f        jenkins_nginx       "nginx"                  52 minutes ago      Up 51 minutes
286624f336d3        jenkins_master      "/bin/tini -- /usr/lo"   53 minutes ago      Up 53 minutes
```

You can see the ContainerID matched the ID of the Jenkins node:
{{< blog-img src="/assets/abs-02-ephemeral-container" >}}

Build is complete and the jenkins-slave container is not running and has been removed:
{{< blog-img src="/assets/abs-02-ephemeral-container-done" >}}

In the detail for our build we can see our "Hello world..." output:
{{< blog-img src="/assets/abs-02-jenkins-build-output" >}}

### Conclusion and Next Steps

So in this part of the tutorial we've configured Jenkins to spin up Docker container slaves based on build labels. Very cool stuff. 

In the next installment we are going to setup a private Docker registry, and secure it with TLS/SSL from letsencrypt. This way we (or our development teams can push their build environment images to our private registry so our build system can have access to them. 

### Resources

Video Tutorial: [https://youtu.be/NN3d_fZO87w](https://youtu.be/NN3d_fZO87w)
