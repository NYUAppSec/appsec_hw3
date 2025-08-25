# Homework 3: Deployment Gone Wrong
## Module 1: Setup

## Introduction

Right when you thought things couldn't get worse, your company decided to
re-hire Shoddycorp's Cut-Rate Contracting again. They say that you've
done a fantastic job cleaning up their code, and that they're sure you
can handle whatever problems may occur in the next project. It seems they will
never learn.

Now that the web application is fixed and ready, your company wants it
deployed in a scalable, reliable, and secure manner. To do this, your
company hired Shoddycorp's Cut-Rate Contracting to containerize your
application, then deploy it in a way that ensures availability and security.
What they delivered, as usual, falls quite short of the mark.

Like last time, what Shoddycorp's Cut-Rate Contracting provided was a deployment
that *almost* works.
They containerized the application, the database, and a
Nginx reverse proxy all in Docker.
They then created Kubernetes yaml files to
run these containers in a Kubernetes cluster, and configured them to talk to
each other as needed.
They even began adding some event monitoring for a
monitoring software called Prometheus, though they didn't finish it.

However, upon further inspection, we can see that they didn't quite do things
right. They left secrets lying around out in the open, they only create one
replica of each pod, and there are passwords getting logged all over the
place. All-in-all, it's a mess.

It looks like the job to fix this falls to you again.
Luckily, Kevin Gallagher (KG) has read through the files already and pointed out some of the things that
are going wrong, and provided a list of things for you to fix.
Before you can work on that, though, let's get your environment set up.

Just a disclaimer, in case it needs to be said again: 
Like with all Shoddycorp's Cut-Rate Contracting deliverables, this is not code
you would like to mimic in any way.

## Frequently Asked Questions

Kubernetes is a fairly complicated beast.
To help you get oriented,
we've created a [Frequently Asked Questions](FAQ.md) document that should help with common questions.
As always, please make use of office hours and ask questions on Ed Discussion.

## Part 1: Setting up Your Environment

To complete this assignment, you will need Docker, minikube, and kubectl.
Installation is not simple, and is highly platform-dependent.
We recommend you look at the setup scripts we've created to [help you along.](https://github.com/NYUAppSec/appsec-env-setup-script)

Every operating system environment is different, we recommend you try and perform the assignment work within a Linux distribution. 
Rather than detail how to install this software on different platforms, below are links to relevant information on how to install these tools (if you chose not to use the scripts)
at their official sites, as well as how to operate them. 

To install Docker, please see the following Website and select [Docker Desktop.](https://www.docker.com/get-started)

To install Kubectl, please see the following [Website.](https://kubernetes.io/docs/tasks/tools/)

To install Minikube, please see the following [Website.](https://minikube.sigs.k8s.io/docs/start/)

As in the previous assignments, we will be using Git and GitHub for submission,
so please ensure you still have Git installed. Remember to continue to follow git
best practices.

When you are ready to begin the project, please use GitHub Classroom to create
your repository for this assignment, and do your work in that repository. The
repository name will look like `NYUAppSec/appsec-homework-3-[username]`.

## Get Latest updates

It is always good to pull the latest updates for your repository before you continue your work.

Use the following commands to pull the latest updates.
```bash
git remote add upstream https://github.com/NYUAppSec/appsec_hw3
git fetch upstream
git merge upstream/main --allow-unrelated-histories
git push
```

### Part 1.1: Rundown of Files

This repository has a lot of files. The following are files you will likely be
modifying throughout this assignment.

* GiftcardSite/GiftcardSite/settings.py
* GiftcardSite/LegacySite/views.py
* GiftcardSite/k8/django-deploy.yaml
* db/Dockerfile
* db/k8/db-deployment.yaml

In addition, you may need to make new files in order to work with Prometheus (see Part 2).

### Part 1.2.a: Getting it to Work 

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
 
 Finally, you might also need to run the following command to configure your shell to use Docker with Minikube:
 
 ```bash
 eval $(minikube docker-env)
 ```

### Part 1.2.b: Getting it to Work 

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


### Part 1.3: Git Signature and Pushing to DockerHub
To remain consistent with our other coding assignments, please complete the following:

* At least one signed git commit 
* Use GitHub Actions to automate deploying your Django container to [DockerHub](https://hub.docker.com/).
  * Create an account on DockerHub. Ideally, you would store your login values in GitHub secrets. If you cannot, just use environment variables, but assume this account can be compromised, do not put anything sensitive on your DockerHub.
  * Use this [GitHub Action](https://github.com/docker/build-push-action) on how to set up an action to push an image to DockerHub. Push your Django Docker Image to a DockerHub repository.

To submit this part, push the `hw3p1handin` tag with the following:
```commandline
git tag -a -m "Completed hw3 part1." hw3p1handin
git push origin main
git push origin hw3p1handin
```

## Part 2: Securing Secrets.

Unfortunately, there are many values that are supposed to be secret floating
around in the source code and in the yaml files.
Typically, we do not want this.
Secret values should be protected so that we can move the source code to GitHub
and put the docker images on Dockerhub and not compromise any secrets.
In addition to keeping secrets secret, this method also allows for changing secrets
more easily.

For this part, your job will be to find some of the places in which secrets are
used and replace them with a more secure way of doing secrets. Specifically,
you should look into Kubernetes [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets), 
how they work, and how they can be used with both kubernetes yaml files and how they may be accessed via Python
(hint: they end up as environment variables). You may want to read the
[Kubernetes documentation on secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/).

To complete this assignment, determine which two values must be secrets in your sealed secret YAML file. 
On the root of your repository, ensure the sealed secret YAML file is called `part1.yaml`.

For this portion of the assignment, you should submit:

1. All kubernetes yaml files created/modified to use sealed secrets
2. All changes necessary to the Web application (limited to 
   settings.py as mentioned above) needed to use the passed secrets.
3. A file, called secrets.txt, which demonstrates how you added the sealed secrets.
   This must include all commands used, etc.

Finally, rebuild your Docker container for the Django application, and then
update your pods using the kubectl apply commands specified earlier.

To submit this part, push the `hw3p2handin` tag with the following:
```commandline
git tag -a -m "Completed hw3 part2." hw3p2handin
git push origin main
git push origin hw3p2handin
```

## Part 3: Monitoring with Prometheus

It seems the DevOps employee at Shoddycorp's Cut-Rate Contracting decided to add
some monitoring to the Web application using Prometheus. However, while they do
seem to know how to use the Python Prometheus client to perform monitoring, they
seem to struggle with understanding what your company may want to monitor. Moreover,
they seem to be using Prometheus' monitoring to monitor things that you want to
remain secret!

As if that wasn't bad enough, it seems that the employee also didn't complete
the Prometheus setup! While there is some monitoring there, there is no
Prometheus service to collect the information that's being exposed on the site.

In this section of the assignment, you will be fixing this situation by removing
problematic monitoring done using Prometheus' python client, and expanding the
reasonable monitoring with a few more metrics. Then you will create a Prometheus
pod and service for Kubernetes, so it can monitor your application.

Specifically, in this part, you must:

### Part 3.1: Remove unwanted monitoring.

There exists some unsafe monitoring of sensitive data in views.py. Remove all
monitoring that exposes any sensitive secrets.

All changes in this section should occur in the GiftcardSite/LegacySite/views.py
file.

### Part 3.2: Expand reasonable monitoring.

There are things we may want to monitor using Prometheus. In this part of the
 assignment, you should add a Prometheus counter that counts all the times we 
purposely return a 404 message in views.py. These lines are caused by Database 
errors, so you should name this counter database_error_return_404.

All changes in this section should occur in the GiftcardSite/LegacySite/views.py
file.

### Part 3.3: Add Prometheus

All of this data is pointless if it is not being collected. In this section, you
should add Prometheus to your Kubernetes cluster and use it to automatically
monitor the metrics from your Web application. Information about how to add
Prometheus to Kubernetes can be found [here](https://prometheus.io/docs/introduction/overview/).

For this section you will submit all the yaml files that you needed to run
Prometheus, as well as a writeup called `prometheus.txt` describing the steps you
took to get it running.


Hints:

* You probably want to look into `helm`, a package manager for kubernetes that makes it easy to install services like Prometheus. Not required, but recommended due to ease of use.

* To configure Prometheus, you probably want to use `configmaps`, which are a way of providing configuration information to running pods. You can see what configmaps are available by using `kubectl get configmaps`, and output their current configuration by doing `kubectl get configmap <service_name> -o yaml`. You can also directly edit the configuration with `kubectl edit configmap <service_name>`.

* Each running service gets a DNS name that corresponds to the service name. So to refer to the proxy running on port 8080, you would use `proxy-service:8080`. Remember every DNS name maps to a corresponding IP. 

* You can see what the final result of Part 2 will look like [here](./FAQ.md#how-do-i-know-if-i-have-finished-part-2).

To submit this part, push the `hw3p3handin` tag with the following:
```commandline
git tag -a -m "Completed hw3 part3." hw3p3handin
git push origin main
git push origin hw3p3handin
```

## Grading

Total points: 100

Part 1 is worth 20 points:

* 10 points for signed commits.
* 10 points for GitHub Actions configuration.

Part 2 is worth 40 points:

* 20 points for the yaml files that use Kubernetes sealed secrets.
* 10 points for the changes to the Django code.
* 10 points for the writeup covering your work in Part 1 (mainly GitHub Actions) and Part 2.

Part 3 is worth 40 points:

* 10 points for removing dangerous monitoring
* 10 points for expanding monitoring
* 10 points for all yaml files for Prometheus
* 10 points for the writeup covering your work in Part 3.

## What to Submit
To submit your code, please only submit a file called `git_link.txt` that contains the name of your repository. 
For example, if your GitHub account username is exampleaccount, you would submit a text file named `git_link.txt` to 
Gradescope with only one line that reads the following:

    appsec-homework-3-exampleaccount

The TA will also be looking for the following files on your Gradescope:

    secrets.txt
    prometheus.txt

Having the write-ups uploaded makes it easier for the TA to grade the write-up as it saves them time traversing your GitHub repository. 
Please be sure to have your written parts in your repository too, and the files are expected to be exactly the same as what is uploaded to Gradescope. 
This will be verified by the autograder hashing the write-up found in the root of your repository and uploaded write-ups.

The repository should contain:

* Part 1
  * At least one signed git commit
  * A GitHub Actions YML that uploads a docker image to Docker Hub
* Part 2
  * Your yaml file, `part1.yaml` using Kubernetes sealed secrets.
  * All files you changed from the GiftcardSite/ directory.
  * A writeup called secrets.txt on the root of your repository.
* Part 3
  * A modified GiftcardSite/LegacySite/views.py file.
  * Your config map yaml file for running Prometheus. The file should be at the root of your repository and called `prometheus.yaml`.
  * A writeup called prometheus.txt on the root of your repository.

## Concluding Remarks

With the changes you made in this assignment, your company is a lot closer to a
decent deployment solution. However, even with the changes, there are a lot of
things that are still lacking.

One of the benefits of using Kubernetes is the ability to create replicas that
are load balanced to avoid overwhelming one instance of the application. The
same can be done with other microservices such as the database, though this
would require database syncing across the difference database instances. These
solutions do not currently exist in this version of the assignment.

For more experience working with cloud security and deployment, consider taking
this one step further and replicating these microservices.
Attempt to load balance over many replicas, and syncing databases.
Try using Prometheus to gather more metrics from all of your different microservices.
Try adding logging and other useful tools.

Though these attempts will not be graded, and should not be submitted as part of
the assignment, they should help you learn a lot about how using cloud
deployment helps you preserve the availability of your service (and the
microservices that comprise it) and how good monitoring and logging can help you
spot errors in the application before they become serious issues.
