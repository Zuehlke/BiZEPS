# certGenerator
`docker-compose run --rm certGenerator [rebuild | all | clean | server | client]`

Generates self signed root CA and derived server and client certificate and key.

##  Usage
  - `docker-compose run --rm certGenerator`:
    * Removes current CA and certificates and generates new CA and certificates
    * Same as `docker-compose run --rm certGenerator rebuild`
  - `docker-compose run --rm certGenerator clean`
    * Removes the certificates in the `output`
  - `docker-compose run --rm certGenerator server`
    * Regenerates the server certificates
    * Does not generate CA if it already exist
  - `docker-compose run --rm certGenerator client`
    * Generates the client certificates
    * Does not generate CA if it already exist
  - `docker-compose up`
    * Short cut for generating the certificates
    * Does not remove the stopped container
    * `docker-compose rm` will be required

`docker-compose` helps to simplify the build, create and run process.
See `docker-compose --help` for further information.


##  Input
### Server Name and Server Alternative Name
By default, the server certificate is created with the following properties:
- CA_NAME=DockerHost
- SERVER_NAME=DockerHost
- SERVER_ALTNAMES=DNS:DockerHost,IP:127.0.0.1

The default properties can be overridden with the following command:
`docker-compose run --rm certGenerator "CA_NAME=MyHost" "SERVER_NAME=MyHost" "SERVER_ALTNAMES=DNS:MyHost,IP:127.0.0.1,IP:10.10.10.20"`

##  Output
  - Create/use `output` directory on host in the execution folder
  - Create or overwrite certificates in the output directory
    - output contains root CA
    - Server and client folders contain CA, certificate and the according key
