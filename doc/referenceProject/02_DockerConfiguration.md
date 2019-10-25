[prev: BiZEPS Reference Project Overview](01_BizepsReferenceProject.md) | [next: Jenkins Configuration](03_JenkinsConfiguration.md)

# Setup docker0 bridge
Set up docker0 as network interface bridge:

```bash
ip link add docker0 type bridge
ip link set docker0 up
ip addr add 172.17.0.1/16 dev docker0
```

# Docker Daemon Startup
The BiZEPS reference project starts the daemon with the following parameters:

`dockerd -D --tlsverify --tlscacert=/var/docker/server/ca.pem --tlscert=/var/docker/server/cert.pem --tlskey=/var/docker/server/key.pem -H tcp://172.17.0.1:2376`

| Parameter   | Description |
| ---         | ---         |
| -D          | Enable debug mode |
| --tlsverify | Use TLS and verify the remote  |
| --tlscacert | Trust certs signed only by this CA (default: ~/.docker/ca.pem) |
| --tlscert   | Path to the TLS certificate (default: ~/.docker/cert.pem) |
| --tlskey    | Path to the TLS server key  (default: ~/.docker/key.pem)  |
| -H tcp://172.17.0.1:2376 | Binds the docker REST API to the docker0 interface |

## Details

###  Protect the REST API
The reference project protects the REST API with two approaches:

**TLS authentication (certificates):**
Clients have to authenticate themself with a TLS certificate before they can
interact with the docker daemon through the docker REST API.

**Do not publish in the network:**
The reference project publishes the docker daemon REST API only on the localhost network interface
and not on a physical network interface.
Therefore it is not possible to access the docker REST API from the network.
On the one hand this is a limitation but also improves the security.

**TLS Certificates**
The [certGenerator utility](/utils/certGenerator/readme.md) of the BiZEPS project can be used
to generate self signed client and server TLS certificates.

[prev: BiZEPS Reference Project Overview](01_BizepsReferenceProject.md) | [next: Jenkins Configuration](03_JenkinsConfiguration.md)
