Moving to [GitLab](https://gitlab.com/bi_zeps/jenkins)

[![Gitlab Mainline](https://gitlab.com/bi_zeps/jenkins/badges/master/pipeline.svg)](https://gitlab.com/bi_zeps/jenkins)
[![Stable Version](https://img.shields.io/docker/v/bizeps/jenkins/stable?color=informational&label=stable)](https://hub.docker.com/repository/docker/bizeps/jenkins)
[![Docker Pulls](https://badgen.net/docker/pulls/bizeps/jenkins?icon=docker&label=pulls)](https://hub.docker.com/repository/docker/bizeps/jenkins)
[![Docker Image Size](https://badgen.net/docker/size/bizeps/jenkins/stable?icon=docker&label=size)](https://hub.docker.com/repository/docker/bizeps/jenkins)

![logo](doc/Logo/BiZEPS_Logo_small.png)

**Zühlke Embedded Build System**

#   Documentation
- [What is BiZEPS](doc/introduction/01_BiZEPS_Introduction.md)
- [BiZEPS Reference Project](doc/referenceProject/01_BizepsReferenceProject.md)
- [BiZEPS on AWS](doc/integrations/BiZEPS_AWS_EC2.md)

#   Overview
BiZEPS realizes the continuous integration build with Jenkins and Docker.
The Jenkins master runs in a docker container and each build job runs in its own container.
Each tool chain is managed and maintained in its own docker image.

BiZEPS has the vision to be used on the server as also on the local development machine.
The corresponding tool chain container could be used from the IDE to build (and run?) the source code.

##  BiZEPS Advantages
### Separation of Tool Chains
Every tool chain has its own container (virtual context) and runs independent from other containers.
The installation of a new tool chain does not affect the current tool chains or the Jenkins server.
BiZEPS is able to support all kind of tool chain in any version at the same time.

### Minimize Idle Resources
In BiZEPS the Jenkins slaves are only active when there is a job running for their tool chain.
They are not active (even not in idle mode), when they are not used.
An idle BiZEPS Jenkins slave only uses disk space.
And even the used disk space is optimized due to Dockers union file system.

### Tool Chain Versions
For every tool chain a Docker image is created from a Docker file.
The Dockerfiles are managed by a version control system (git).
With a properly written Dockerfile any version of a specific image can be recovered at any time.

Also Docker it self provides a mechanism to tag and specify image versions.
As long as the images are on the Docker host or if they are pushed to the Docker Hub Registry,
the tags can be used to recover a previous image version.

### Application Updates
To update an application in BiZEPS, the image of the corresponding container must be updated.
The current container and the container from the updated image can concurrently run on the Docker Host.
This means that an application update can be tested before the old container is replaced by the updated container.
If the updated application container does not behave as expected, a roll back to the previous version can be done.

### Simple Distribution of the Development Environment
If parts of BiZEPS could also be used on the development machine the developer
would use the same environment as the build server.
This means local builds would no more be affected by local installations (e.g. new version of a specific library).

The tool chain update could be provided in the same way as for the build server.
