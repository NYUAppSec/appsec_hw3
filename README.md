# Homework 3: Deployment Gone Wrong
## Module 1: Setup
### Introduction
Right when you thought things couldn't get worse, your company decided to re-hire Shoddycorp's Cut-Rate Contracting again. They say that you've done a fantastic job cleaning up their code, and that they're sure you can handle whatever problems may occur in the next project. It seems they will never learn.

Now that the web application is fixed and ready, your company wants it deployed in a scalable, reliable, and secure manner. To do this, your company hired Shoddycorp's Cut-Rate Contracting to containerize your application, then deploy it in a way that ensures availability and security. What they delivered, as usual, falls quite short of the mark.

Like last time, what Shoddycorp's Cut-Rate Contracting provided was a deployment that almost works. They containerized the application, the database, and a Nginx reverse proxy all in Docker. They then created Kubernetes yaml files to run these containers in a Kubernetes cluster, and configured them to talk to each other as needed. They even began adding some event monitoring for a monitoring software called Prometheus, though they didn't finish it.

However, upon further inspection, we can see that they didn't quite do things right. They left secrets lying around out in the open, they only create one replica of each pod, and there are passwords getting logged all over the place. All-in-all, it's a mess.

It looks like the job to fix this falls to you again. Luckily, Kevin Gallagher (KG) has read through the files already and pointed out some of the things that are going wrong, and provided a list of things for you to fix. Before you can work on that, though, let's get your environment set up.

Just a disclaimer, in case it needs to be said again: Like with all Shoddycorp's Cut-Rate Contracting deliverables, this is not code you would like to mimic in any way.

### Frequently Asked Questions
Kubernetes is a fairly complicated beast. To help you get oriented, we've created a Frequently Asked Questions document that should help with common questions. As always, please make use of office hours and ask questions on Ed Discussion

### Step 1: Setting up Your Environment

To complete this assignment, you will need Docker, minikube, and kubectl.
Installation is not simple, and is highly platform-dependent.
We recommend you look at the setup script we created and is included in this repository, named 'setup.sh'. 

Every operating system environment is different, we recommend you try and perform the assignment work within a Linux distribution, preferable Ubuntu.
Rather than detail how to install this software on different platforms, below are links to relevant information on how to install these tools (if you chose not to use the scripts)
at their official sites, as well as how to operate them. 

To install Docker, please see the following Website and select [Docker Desktop.](https://www.docker.com/get-started)

To install Kubectl, please see the following [Website.](https://kubernetes.io/docs/tasks/tools/)

To install Minikube, please see the following [Website.](https://minikube.sigs.k8s.io/docs/start/)

As in the previous assignments, we will be using Git and GitHub for submission,
so please ensure you still have Git installed. Remember to continue to follow git
best practices.

#### Image Versions

You may see something similar to the below:
```
🆕  Kubernetes 1.32.0 is now available. If you would like to upgrade, specify: --kubernetes-version=v1.32.0
✨  Using the docker driver based on existing profile
👍  Starting "minikube" primary control-plane node in "minikube" cluster
🚜  Pulling base image v0.0.46 ...
🔄  Restarting existing docker container for "minikube" ...
❗  Image was not built for the current minikube version. To resolve this you can delete and recreate your minikube cluster using the latest images. Expected minikube version: v1.34.0 -> Actual minikube version: v1.35.0
```

Version mismatches like this are common! The ❗ symbol indicates a version mismatch warning—in this case, the image was built for Minikube v1.34.0, but you're running Minikube v1.35.0. This can lead to compatibility issues. Such mismatches often occur when teams upgrade to newer versions without properly managing dependencies. The fix? You may need to delete and recreate the Minikube cluster to align versions - if the versions even align, either way, worthy of a discussion in your writeups in future modules. Looks like Shoddy Corp didn’t handle their update policies very well!

When you are ready to begin the project, please use GitHub Classroom to create
your repository for this assignment, and do your work in that repository. The
repository name will look like `NYUAppSecCF/assignment-3-module-1-[username]`.

### Step 1.1: Rundown of Files

This repository has a lot of files. The following are files you will likely be
modifying throughout this assignment.

* GiftcardSite/GiftcardSite/settings.py
* GiftcardSite/LegacySite/views.py
* GiftcardSite/k8/django-deploy.yaml
* db/Dockerfile
* db/k8/db-deployment.yaml

In addition, you may need to make new files in order to work with Prometheus (see Part 2).

### Step 1.2.a: Getting it to Work 

Once you have installed the necessary software, you are ready to run the whole thing
using minikube. First, start minikube.

```bash
minikube start
```

#### Troubleshooting

 If you encounter issues such as "Unable to pick a default driver" or if Docker is not healthy, read the error messages carefully.
 
 Verify that Docker is running correctly. This command is very useful:
 
 ```bash
 docker info
 ```
 
 If you have a permission issue you may need add your user to the Docker group:

 ```bash
 sudo usermod -aG docker $USER
 newgrp docker
 ```
Insight: '-a' stands for "append" and adds the user to the specified group without removing them from other groups they might already belong to. 'G' specifies the group that you want to add the user to. '$USER' is an environment variable that automatically holds the name of the currently logged-in user. You should know all about environment variables from Assignment 2. 
 
 It is good practice to check 'docker info' to see if there are still issues after the above permissioning. If not, try 'minikube start' again.
 
 If you still have issues still you may need to explicitly set Minikube to use the Docker driver (virtualbox for example is 'minikube config set driver virtualbox'):
 ```bash
 minikube config set driver docker
 ```

Remember, hints are there for a reason! ```bash❗  These changes will take effect upon a minikube delete and then a minikube start```.
 
 Finally, you might also need to run the following command to configure your shell to use Docker with Minikube:
 
 ```bash
 eval $(minikube docker-env)
 ```

### Step 1.2.b: Getting it to Work 

Next, we need to build the Dockerfiles Kubernetes will use to create the
cluster. This can be done using the following lines, assuming you are in the
root directory of the repository. These take time to build, be patient.

```
docker build -t nyuappsec/assign3:v0 .
docker build -t nyuappsec/assign3-proxy:v0 proxy/
docker build -t nyuappsec/assign3-db:v0 db/
```

Then use kubectl to create the pods and services needed for our project. Again,
these commands assume you are in the root directory of the repository.

```
kubectl apply -f db/k8
kubectl apply -f GiftcardSite/k8
kubectl apply -f proxy/k8
```
Verify that the pods and services were created correctly.

```
kubectl get pods
kubectl get service
```

There should be three pod entries:

* One that starts with assignment3-django-deploy
* One that starts with mysql-container
* One that starts with proxy

They should each have status RUNNING after approximately a minute, this depends on the resources your local machine has available.

There should also be four service entries:

* One called kubernetes
* One called assignment3-django-service
* One called mysql-service
* One called proxy-service

To see if you can connect to the site, run the following command:

```
minikube service proxy-service
```

This should open your browser to the deployed site. You should be able to view the first page of the site and navigate around. If this worked, you are ready to move on to the next part. If you are using WSL (Windows) or another virtual machine without graphical user interface, keep in mind how your localhost loopback works for access in your native environment (or research it). 


### Step 1.3: Git Signature and Pushing to DockerHub
To remain consistent with our other coding assignments, please complete the following:

* At least one signed git commit 
* Use GitHub Actions to automate deploying your Django container to [DockerHub](https://hub.docker.com/).
  * Create an account on DockerHub. Ideally, you would store your login values in GitHub secrets. If you cannot, just use environment variables, but assume this account can be compromised, do not put anything sensitive on your DockerHub.
  * Use this [GitHub Action](https://github.com/docker/build-push-action) on how to set up an action to push an image to DockerHub. Push your Django Docker Image to a DockerHub repository.


### Ready for Grading

Feel free to start submitting on gradescope to see how you would score. Once you want to lock in your grade push the `assign3mod1handin` tag with the following:

To submit this part, push the `assign3mod1handin` tag with the following:
```commandline
git tag -a -m "Completed hw3 part1." assign3mod1handin
git push origin main
git push origin assign3mod1handin
```
**DO NOT PUSH THIS TAG UNTIL YOU WANT TO BE GRADED**
