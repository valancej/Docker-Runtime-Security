# Docker Runtime Security

## Introduction

Previously, in our Docker Security Best Practices series, we took a deeper look into Docker Image security, and what best practices to follow. This post will continue the series, focusing on Docker container runtime, the challenges that come with securing them, and what countermeasures can be taken to achieve a better container runtime security stance. Left out from this discussion will be any considerations that touch on host or static image security.

## Background on containers

We previously defined a Docker image as is a collection of data that includes all files, software packages and metadata needed to create a running instance of a Docker container. What are Docker containers? Containers are a method of operating system virtualization that allow you to run an application and its dependencies in resource-isolated processes. These isolated processes can be running on a single host without visiability into each others' processes, files, and network. Oftentimes, each container instance provices a single service or piece of an application (commonly known as a microservice). 

Containers themselves are immutable, meaning any changes made to a running container instance will be made on the image, and then reployed. This allows to more streamlined development, and higher degree of confidence when deploying. 

**Why runtime container security is important?** One of the last pieces of a container's lifecycle is deployment to production, for many organizations, this stage is the most critical. Oftentimes, a production deployment is the longest period of a container lifecyle, and therefore, needs to be consistently monitored for threats, misconfigurations, and other weaknesses. Once we have containers live and running, it is vital to be able to take action quickly and in real time to mitigate potential attacks. Simply, production deployments are extremely important pieces of infrastructure and highly valued to organizations and their customers. 

## AppArmour

From the Docker documentation: 

*AppArmor (Application Armor) is a Linux security module that protects an operating system and its applications from security threats. To use it, a system administrator associates an AppArmor security profile with each program. Docker expects to find an AppArmor policy loaded and enforced.*

