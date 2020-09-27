# Homework
The goal of this homework is to find out how do you approach solving some issues and how do you think when designing new or improving existing solutions. The first three tasks are theoretical, you are expected to write up a fairly-short answer as you would in a standard asynchronous communication channel (e.g. slack, email, issue tracker). The last one is purely practical, the expected output is a git repo (share as public, private or gzipped).

If you have any questions, feel free to reach out.

## ECR image replication
A customer is using Elastic Container Registry (ECR) to host their docker images for their Elastic Kubernetes Service (EKS). Docker image gets built in Gitlab-CI and pushed to ECR’s `us-east-1` endpoint. EKS cluster is running in `eu-east-1` too and uses a VPC endpoint for pulling the images from ECR/S3. The customer created recently a new EKS cluster in `eu-west-1` and wants to know how they will pull images to the new cluster. How would you design a simple and cost-effective solution?

## LunchSharing app
A customer runs a backend web service - for influencers from Spain and Finland to share what they had for lunch - as a kubernetes deployment in Google Kubernetes Engine (GKE) having set 3 replicas for the deployment. GKE cluster runs in `eu-west-1` and is fronted by Google TCP Network LoadBalancer. There are moments during lunch time when the service returns 5xx errors on GET (image serving) and POST (image upload) from its mobile app clients as its fully saturated. Outside of lunch hours the service pretty much only serves the images for the end users and does not fully utilize its reserved compute resources. The stateless backend is backed by Google Filestore to persist all of the images. The app is getting more popular and end users are complaining that image uploads and downloads are very slow, fail a lot and need to be retried. The app owner is concerned about rising cloud costs that are way out of hand already. What solution what would you suggest?

## Increased Backend latency
An engineer is tasked with troubleshooting an increased backend service latency. The engineer is suspecting the Application Load Balancer's (ALB - Elastic Load Balancing product) lack of TLS 1.3 support along with suboptimal certificates from AWS Certificate Manager (ACM) and suggests switching to Network Load Balancer and LetsEncrypt certificates managed with a new cert-manager that would run inside of an existing EKS cluster. You are reviewing this architectural decision - what would you do?

## dkr-ls
Write a small CLI tool “dkr-ls” that can list docker images that you have in your local docker registry (similar to the output of `docker images`). There is one command to list repositories, another command to list tags, one tag per line, with additional filters:
```
--minAge (int) list only images older than minAge days
--prefix (stringArray) list only images with these repository prefixes
--maxTags (int) list only maxTags number of tags of each image
--minSize (string) list only images bigger than minSize in human-readable format
--sum (boolean) sum all image sizes and print them at the end of the listing, default: false
```
Add a root-level (for all commands) flag `--log-level {DEBUG,INFO,WARNING,ERROR..}` and log accordingly.

Example usage:
```
$ dkr-ls repos
REPOSITORY
nginx
mysql/mysql-server
datadog/agent
123456789000.dkr.ecr.us-east-1.amazonaws.com/webapp
awscli
debian
<none>
```

```
$ dkr-ls tags
REPOSITORY                                                              TAG                        IMAGE ID            CREATED             SIZE
nginx                                                                   latest                     7e4d58f0e5f3        2 weeks ago         133MB
nginx                                                                   alpine                     6f715d38cfe0        6 weeks ago         22.1MB
mysql/mysql-server                                                      5.7                        9c31a29b3f30        2 months ago        322MB
datadog/agent                                                           7                          534ebd31f754        3 months ago        540MB
123456789000.dkr.ecr.us-east-1.amazonaws.com/webapp                     20200609-1018              4660474f5c5b        3 months ago        902MB
awscli                                                                  1                          39491bbd3b9a        3 months ago        317MB
debian                                                                  buster-20200514-slim       108d75da320f        4 months ago        69.2MB
<none>                                                                  <none>                     c0dc2538b9fa        4 months ago        661MB
```

```
$ dkr-ls repos --maxTags 3
Error: unknown flag: --maxTags
Usage:
...
```

```
$ dkr-ls tags --prefix nginx --prefix debian
REPOSITORY                                                              TAG                        IMAGE ID            CREATED             SIZE
nginx                                                                   latest                     7e4d58f0e5f3        2 weeks ago         133MB
nginx                                                                   alpine                     6f715d38cfe0        6 weeks ago         22.1MB
debian                                                                  buster-20200514-slim       108d75da320f        4 months ago        69.2MB
```

```
$ dkr-ls tags --maxTags 1 --prefix nginx --prefix debian --sum=true
REPOSITORY                                                              TAG                        IMAGE ID            CREATED             SIZE
nginx                                                                   latest                     7e4d58f0e5f3        2 weeks ago         133MB
debian                                                                  buster-20200514-slim       108d75da320f        4 months ago        69.2MB
                                                                                                                       TOTAL               202.2MB
```

```
$ dkr-ls tags --minAge 20 --minSize 100MB
REPOSITORY                                                              TAG                        IMAGE ID            CREATED             SIZE
mysql/mysql-server                                                      5.7                        9c31a29b3f30        2 months ago        322MB
datadog/agent                                                           7                          534ebd31f754        3 months ago        540MB
123456789000.dkr.ecr.us-east-1.amazonaws.com/webapp                     20200609-1018              4660474f5c5b        3 months ago        902MB
awscli                                                                  1                          39491bbd3b9a        3 months ago        317MB
<none>                                                                  <none>                     c0dc2538b9fa        4 months ago        661MB
```

Choose a language of your preference to solve the task. Try to write readable, testable, reusable (don't go crazy here) code. Don't reinvent the wheel, use well-maintained libraries and modules that can help you with some of the things.

Extras:
- Write a simple bash script called e2e.sh that prepopulates some images to your docker registry, executes all these examples (we want to see the ouput on stdout/stderr) and cleans up the registry at the end.
- Create a simple Makefile so that it is easy for somebody to use your dkr-ls tool (i.e. add something like `make build`, `make test`, `make lint`, `make e2e`).
- Create a Dockerfile for yout dkr-ls tool and add an quickstart example in your README.md to make it easy for people to use your tool without needing to install or compile anything other than Docker.