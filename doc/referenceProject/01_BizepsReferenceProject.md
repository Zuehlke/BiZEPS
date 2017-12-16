[next: Docker Configuration](02_DockerConfiguration.md)

#   BiZEPS Reference Project

The reference project describes the BiZEPS architecture, setup and configuration by an example.

##  Overview

```
                                   +------------------------+
                                   |                        |
                                   |  f) Jenkins master     |
                                   |     Container          |
                                   |                        |
                                   | +--------------------+ |
                                   | |  Credentials Store | |
                                   | |  * Dockerdaemon    | |
                                   | |  * Dockerhub       | |
                 +-------------------+  * ....            | |                         +--------------------+
                 |                 | |                    | |                         |                    |
                 |                 | +--------------------+ |                       +-+------------------+ |
                 |                 |                        |                       |                    | |
                 |                 |                        |                     +-+------------------+ | |
                 |                 | +--------------------+ | j) Docker API       |                    | +-+
                 |                 | | Docker client      +-----------------------> i) Toolchain       | |
                 |                 | +-------------+------+ |                     |    Container       +-+
                 |                 |               |        |                     |                    |
                 |                 +----------------------^-+                     +---^----------------+
                 |                                 |      |                           |
                 |               g) Docker Daemon  |      |          h) Network Access|
                 |                  REST API Access|      +---------------------------+
Container        |                                 |                                  |
+------------------------------------------+------------------------------+---------------------------------------+
Host             |                         |       |                      |           |
                 |                         |       |                      |           |
                 |                         |       |                      |           |
                 |                         |   +---v-----------------+    |   +-------v--------+  +---------------+
   +-------------v---+-----------------+   |   | b) Docker REST API  |    |   |                |  |               |
   | client/ca.pem   | server/ca.pem   |   |   +---------------------+  bind  | e) docker0     |  |d) eth0        |
   | client/cert.pem | server/cert.pem <-------+ a) Docker daemon    +-------->    (172.17.0.1)|  |   (172.17.0.1)|
   | client/key.pem  | server/key.pem  |   |   +---------------------+        |                |  |               |
   +-----------------+-----------------+   |                              |   +----------------+  +---------------+
   |     c) Docker TLS Certificates    |   |                              |
   +-----------------------------------+   |                              |
                                           |                              |
   Host File System                        |   Host Daemon                |   Host Network Interfaces
                                           |                              |
```

##  Details
The described architecture is a possible solution, how a BiZEPS project could look like.
This setup is a good starting point which addresses flexibility, scalability and basic security needs.
Other project requirements or customer needs may lead into a different project setup.

### Docker
####  a) Docker daemon
Docker images and container are managed through the docker daemon.
Every docker command (e.g. `docker run ...`) is handled by the docker daemon.
The docker daemon is bound to the docker0 bridge.

####  b) Docker REST API
The docker daemon provides a REST API to access the daemon and execute docker commands.
The REST API is usually used by applications for interacting with the daemon.
The reference project publishes the docker REST API only on local host.
Remote clients cannot access the docker daemon.

####  c) Certificates Docker TLS
The BiZEPS reference project uses the built in TLS authentication to protect the docker REST API.
A client has to authenticate himself before he can interact with the REST API.
The certificates can be stored on the host, the docker daemon needs access to the server certificates.
With the client certificates the Jenkins master can authenticate himself to the docker daemon.

The BiZEPS projects provides a [certGenerator utility](/utils/certGenerator/summary.md) to create self signed server and client certificates.

### Network
####  d) eth0
The physical network interface of the host.

####  e) docker0 bridge
The virtual docker0 interface is used to communicate
- with each other
- with the host
- with other resources in the network (or internet)

### Jenkins Master
####  f) Container Jenkins master
The container with the Jenkins service.
The Jenkins master manages the Jenkins configuration, plugins and builds.
By default Jenkins master creates and starts a container for every build job.
The build is executed in a container, the build results are returned
to the Jenkins master and afterwards the container is destroyed.

A docker client allows Jenkins to interact with the Docker daemon (Docker host).
There is no Docker daemon installed within the Jenkins container.
The credentials to access the Docker host are stored in a credential store on Jenkins.

####  g) Docker daemon REST API access
To create, start and destroy containers the Jenkins master
has to interact with the docker daemon (via REST API).
The Jenkins master has to authenticate with the TLS certificate and key to access the docker daemon API.
Jenkins master uses the routing table entry to access the docker daemon.

####  h) Network access
To interact with the Jenkins server the 'public' network interface is published in the network.
Even Jenkins slave container may access the public network.

### Buildjob container
####  i) Container Toolchain
For every build job a dedicated toolchain container is started.
The toolchain container contains the tools and basic configuration to fulfill a specific build job.
By default a container shall contain one toolchain (in one specific version).

####  j) Docker API
Jenkins master controls the build containers with the Docker client through
the Docker REST API provided by the docker host.

[next: Docker Configuration](02_DockerConfiguration.md)
