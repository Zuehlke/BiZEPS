#	Using BiZEPS with Amazon Cloud

##	Setup
###	Selecting Image
- Used Image from Amazon Market Place
  - **Amazon Linux 2 AMI (HVM), SSD Volume Type (64-bit x86 or 64-bit Arm)**
- Modify Security Settings
	- Open Port 22 for SSH (Putty) for current Address only (hint: Only open when needed)
	- Open Port 8080 for webserver access (build server)
- Connect with Putty and private key
  - User: ec2-user

###	Install and Setup Docker
Execute the following commands via SSH on the cloud instance to setup docker
- `sudo yum update -y`
- `sudo yum install -y docker`
  - Optional: `sudo amazon-linux-extras install docker`
- `sudo service docker start`
- `sudo systemctl start docker`
  - To start docker service after a restart
- `sudo usermod -a -G docker ec2-user`
  - Add ec2-user to use the service without sudo
- Logout and log in to pick new docker group permissions or reboot
- Optional: Install docker-compose:
  - `sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`
  - `sudo chmod +x /usr/local/bin/docker-compose`

### Configure Docker Daemon (TLS)
Configure the docker daemon to enable the docker REST API on the network interface and secure the API with TLS authentication.

- Use the [certGenerator utility](/utils/certGenerator/readme.md) to create the required certificates
  - `docker run --rm -u root -v /srv/certs/docker:/var/build bizeps/certgenerator`
  - Hint: To connect from another docker client make sure to add the host machines name or IP to the certificates, see `SERVER_ALTNAMES`
- Configure docker daemon to use the certificates
	- Open `/etc/sysconfig/docker`
  - Add or modify the options line
    - `OPTIONS="-H unix:///var/run/docker.sock -H tcp://0.0.0.0:2376 --tlsverify --tlscacert=/srv/certs/docker/server/ca.pem --tlscert=/srv/certs/docker/server/cert.pem --tlskey=/srv/certs/docker/server/key.pem --default-ulimit nofile=1024:4096"`
  - `sudo service docker restart`
  - Verify that the configuration is in use
    - `ps aux | grep docker`
