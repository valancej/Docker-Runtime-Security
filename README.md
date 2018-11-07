# Docker Runtime Security

## Introduction

Previously, in our Docker Security Best Practices series, we took a deeper look into Docker Image security, and what best practices to follow. This post will continue the series, focusing on Docker container runtime, the challenges that come with securing them, and what countermeasures can be taken to achieve a better container runtime security stance. Left out from this discussion will be any considerations that touch on host or static image security.

## Background on containers

We previously defined a Docker image as is a collection of data that includes all files, software packages and metadata needed to create a running instance of a Docker container. What are Docker containers? Containers are a method of operating system virtualization that allow you to run an application and its dependencies in resource-isolated processes. These isolated processes can be running on a single host without visiability into each others' processes, files, and network. Oftentimes, each container instance provices a single service or piece of an application (commonly known as a microservice). 

Containers themselves are immutable, meaning any changes made to a running container instance will be made on the image, and then reployed. This allows to more streamlined development, and higher degree of confidence when deploying. 

**Why runtime container security is important?** One of the last pieces of a container's lifecycle is deployment to production, for many organizations, this stage is the most critical. Oftentimes, a production deployment is the longest period of a container lifecyle, and therefore, needs to be consistently monitored for threats, misconfigurations, and other weaknesses. Once we have containers live and running, it is vital to be able to take action quickly and in real time to mitigate potential attacks. Simply, production deployments are extremely important pieces of infrastructure and highly valued to organizations and their customers. 

## Some best practices to consider

The below are a list of best practices to consider when discussing Docker runtime security

### AppArmor & Docker

From the Docker documentation: 

*AppArmor (Application Armor) is a Linux security module that protects an operating system and its applications from security threats. To use it, a system administrator associates an AppArmor security profile with each program. Docker expects to find an AppArmor policy loaded and enforced.*

It is available on Debian and Ubuntu by default. In short, it is important for system administrators to not disable Docker's default AppArmor profile or create their own customer security profile for conatiners specific to their organization. Once this profile is utilized, the container has a certain set of restrictions and capabilities such as network access or file read/write/execute permissions. For more info on Apparmor see the official Docker documentation: https://docs.docker.com/engine/security/apparmor/

### SELinux & Docker

SELinux is an appliction secrutiy system that provides an access control system that greatly augments the Discretionary Access Control model. If applicable on the Linux host OS, you can start Docker in daemon mode with SELinux enabled. The container would then have a set of restrictions as definied in the SELinux policy. For more info on SELinux: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html/container_security_guide/docker_selinux_security_policy

### Do not use privileged containers

Do not allow containers to be run with the `--privileged` flag. This flag gives all capabilities to the container, and also lifts all the limitations enforced by the device cgroup controller. In short, the container can then do almost everything the host can do. 

### Do not expose unused ports

The Dockerfile defines which ports will be opened by default on a running conatiner. Only the ports that are needed and relevant to the application should be open. If there is access to the Dockerfile this can be evident by looking for the `EXPOSE` instruction. 

### Do not run ssh within containers

SSH server should not be running within a container. Here is a blog post explaining: https://blog.docker.com/2014/06/why-you-dont-need-to-run-sshd-in-docker/

### Do not share the host's network namespace

The networking mode on a container when set to `--net=host`, skips placing the container inside a separate network stack. In other words, this tells Docker to not containerize the container's networking. This is potentially dangerous in that it allows the container to open low-numbered ports like any other root process. Additionally, a container could potentially do unexpected things such as terminate the Docker host. 

Simply, do not add the `--net=host` option when running a container. 

### Manage memory and CPU usage of containers 

By default, a container has no resource constraints and can use as much of a given resource as the host's kernal allows. Additionally. all containers on a Docker host share the resources equally and non memory limits are enfored. A major risk is when a running container begins to consume too much memory on the host machine. For Linux hosts, if the kernal detect that there is not enough memory to perform important system functions, it through an out of memory exception, and beings to kill processes to free up memory. This could potentially bring down an entire system if the wrong process is killed. 

Docker can enforce hard memory limits, which allow the container to use no more than a given amount of user or system memory. Docker can also enforce soft memory limits, which allow the conatiner to user as much memory as needed unless certain conditions are met. When running a container the `--memory` flag is what defines the maximum amount of memory the container can use. 

In the case of managing container CPU, the `--cpu` flags, give more control over the container's access to the host machines CPU cycles.

### Set on-failure container restart policy

By using the `--restart` flag when running a container, you can specify how a container should or should not be restarted on exit. If a container keeps exiting and attempting to restart, it could possibly lead to a denial of service on the host. Additionally, ignoring the exis status of a container and always attempting to restart the container can lead to a non-investigation of the root cause behind the termination. An investigation should always be conducted when a container is exited and attempted to be restarted. The `--on-failure` restart policy should be configured to limit number of retries. 

### Mount container's root filesystem as read only

Containers should be run with their root filesystems in read-only mode. This isolates writes to specifically defined directories, which can then be easily monitored. Additionally, using read-only filesystems makes containers more reslient to being compromised. Data should also not be written within containers. This is just a standard piece following the best practice of immuntable infrastructure, and reduces attack vectors since the instance cannot be written to. There should be an explicitly defined volume for writing for the container. 


## Conclusion

Docker runtime security is critical to overall container security strategy. While taking the above best practices into consideration, conducting research specific to organizational needs, and applying the appropriate host and Docker image security practices, development teams can greatly increase their overall security throughout a container's lifecyle. 