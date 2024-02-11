---
title: "Installing Docker CE on an Air Gapped RHEL system"
date: 2017-11-30
description: "The free Docker CE is not compatible with Red Hat, so how can we install it, resolve dependencies, and transfer images - all on a RHEL server with no internet access?"
#cover: banner.png
tags: [guides, docker]
comments: false
toc: true
---

# Introduction
I recently completed a deployment inside a customer's lab involving several containerized services, in which the server instances provided were not permitted to have direct access to the internet. To add to the issue, the instances themselves were running RHEL 7.3 with no active subscription.

There were several problems in play here:

1. Docker only make their Enterprise Edition (EE) compatible with RHEL. The usual free Community Edition (CE) is not officially supported.
2. Attempting to circumvent #1 by installing the RPM for Centos resulted in dependency issues.
3. Normally these dependency issues would be trivial to resolve with internet access, however in this case, no internet access was available on the instances.
4. Attempting to manually download the dependencies on another machine and transfer them to the servers resulted in more dependency issues (it was a tree of dependencies!)
4. How can the required container images be pulled with no internet access?

The below writeup should cover the solution to all of these issues, so no matter what your issue is - I hope this is useful to you.

_As always, proceed at your own risk. I will not be held responsible for any problems that result as a result of you following this guide._

---

# Part 1 - Configure a local repository
_Skip this step if you have internet access on your target server as any issues with dependencies should resolve themselves automatically._

To resolve the dependency issues on RHEL, we are going to create a local Yum repository from the Red Hat installation media which contains resolutions to our dependency issues.

## 1. Download the RHEL ISO
- Download the RHEL ISO for your applicable version (7.2, 7.3 etc).
- RHEL ISOs can be downloaded from the [RedHat developer page][rheldev] (free account required).
- You can use the below command to check your version if unsure:

```shell
cat /etc/redhat-release
```


## 2. Copy the ISO to the required servers
- The ISO will need to be copied to all the servers which Docker is to be installed on.

Example:
```shell
scp rhel-server-7.X-x86_64-dvd.iso <user>@<hostname>:
```

## 3. Mount the ISO
- Mount the ISO on your target server:

```shell
mount -o loop rhel-server-7.X-x86_64-dvd.iso /mnt
```

## 4. Copy the repository locally
- Copy the repository from the mounted ISO and set the correct permissions.

```shell
cp /mnt/media.repo /etc/yum.repos.d/rhel7dvd.repo
chmod 644 /etc/yum.repos.d/rhel7dvd.repo
```

## 5. Edit the repo file
```shell
vi /etc/yum.repos.d/rhel7dvd.repo
```
_If you haven't used vi before, press `a` to enter edit mode, `ESC` to exit edit mode, and `:wq` to save and quit._

- Change the `gpgcheck=0` parameter to `1`.
- Add the following 3 lines to the end of file (but before the `~` characters in the editor):
```shell
enabled=1
baseurl=file:///mnt/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```
Save and exit vi.

## 6. Clean the repositories
```shell
yum clean all && subscription-manager clean
```

## 7. Verify the repo works
This should generate a very long packages list if working correctly.
```shell
yum  --noplugins list
```

---

# Part 2 - Install Docker CE
Attempting to install the RPM for Centos will fail due to a missing dependency. In this part, we will download Docker CE and the dependency manually and install.

## 1. Download the Docker CE RPM for Centos
_**Note 1:** At the time of this post, Docker 17.09.0 CE was the latest version available for Centos. You may wish to check for a more up-to-date version if applicable._

_**Note 2:** The Container SELinux dependency below was for RHEL 7.3. Your version may differ. Please confirm before proceeding._
On your internet-enabled machine, download the Docker 17.09.0 CE package and its dependency.

If your instance does not have internet access, you will need to complete this step on a connected machine and then transfer the files across.

Docker CE:
```shell
curl https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-17.09.0.ce-1.el7.centos.x86_64.rpm -o docker.rpm
```

Dependency:
```shell
curl http://mirror.centos.org/centos/7.3.1611/extras/x86_64/Packages/container-selinux-2.9-4.el7.noarch.rpm -o containerselinux.rpm
```

## 2. Install
Make sure to run the command with both RPMs in the specific order below! The dependency should be installed first!

```shell
yum install -y containerselinux.rpm docker.rpm
```

If the above command succeeds: congratulations! You have successfully installed Docker CE on Red Hat!

## 3. [OPTIONAL] Start Docker
Start Docker and set it to start at boot.
```shell
systemctl start docker
systemctl enable docker
```

---
# Part 3 - Transfer Docker images
_This step is not required if you have internet access available on your server_

Now that Docker is installed, there is the issue of: how do we actually get our Docker images onto the server if there is no internet access available?

The answer is: You will pull the images on another (internet-connected) machine with Docker, save the images, and then transfer them across to your server. Docker will always attempt to use a local image __first__ before pulling the image from a remote repo.

## 1. Pull the required Docker images
In this example, we will be pulling the images for both Zookeeper and Kafka (your images might differ).

On an internet-enabled machine with Docker installed, run the following:

```shell
docker pull my_docker_image
save -o my_docker_image.docker my_docker_image
```
Replacing `my_docker_image` with the same of the image you want to pull and save.

For example, for both Kafka and Zookeeper:
```shell
docker pull wurstmeister/zookeeper
docker pull wurstmeister/kafka:latest
docker save -o zookeeper_image.docker wurstmeister/zookeeper
docker save -o kafka_image.docker wurstmeister/kafka
```

Copy the `.docker` file(s) you saved across to your target server.

## 2. Load the Docker images
On your target (non-internet connected) server, load the Docker images into inventory with:
```shell
docker load -i my_image_name
```

For example, again for both Kafka and Zookeeper:
```shell
docker load -i kafka_image.docker
docker load -i zookeeper_image.docker
```

## 3. Done!
You can now use Docker as you normally would with the images loaded above.

[rheldev]: https://www.gitbook.com/book/nathancatania/telemetry-backbone-installation/edit#
