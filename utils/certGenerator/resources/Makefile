CA_NAME=DockerHost
SERVER_NAME=DockerHost
SERVER_ALTNAMES=DNS:${SERVER_NAME},IP:127.0.0.1
SERVER_BIND=0.0.0.0
OUT_DIR=.

SYSTEMD_FILE=/etc/systemd/system/docker.service.d/docker-tls.conf
CERT_DIR=/etc/docker/certs

CA_CERT=${OUT_DIR}/ca.pem
CA_KEY=${OUT_DIR}/ca-key.pem
CA_SRL=$(basename ${CA_CERT}).srl
CA_DAYS=3650

SERVER_DIR=${OUT_DIR}/server
SERVER_CERT=${SERVER_DIR}/cert.pem
SERVER_KEY=${SERVER_DIR}/key.pem
SERVER_CSR=${SERVER_DIR}/request.csr
SERVER_DAYS=3650

CLIENT_NAME=client
CLIENT_DIR=${OUT_DIR}/client
CLIENT_CERT=${CLIENT_DIR}/cert.pem
CLIENT_KEY=${CLIENT_DIR}/key.pem
CLIENT_CSR=${CLIENT_DIR}/request.csr
CLIENT_DAYS=3650

.PHONY : all clean rebuild ca server client

all: server client

clean:
	rm -rf ${OUT_DIR}/*

rebuild: clean all
  
${CA_CERT}:
	openssl genrsa -out ${CA_KEY} 4096
	openssl req -new -x509 -sha256 \
	  -subj "/CN=${CA_NAME}" \
	  -days ${CA_DAYS} \
	  -key ${CA_KEY} \
	  -out ${CA_CERT}

server: ${CA_CERT}
	mkdir -p ${SERVER_DIR}
	openssl genrsa -out ${SERVER_KEY} 4096
	openssl req -new -sha256 \
	  -subj "/CN=${SERVER_NAME}" \
	  -key ${SERVER_KEY} \
	  -out ${SERVER_CSR}
	echo subjectAltName = ${SERVER_ALTNAMES} > ${SERVER_CSR}.ext
	openssl x509 -req -sha256 \
	  -in ${SERVER_CSR} \
	  -days ${SERVER_DAYS} \
	  -CA ${CA_CERT} \
	  -CAkey ${CA_KEY} \
	  -CAcreateserial \
	  -extfile ${SERVER_CSR}.ext \
	  -out ${SERVER_CERT}
	cp ${CA_CERT} ${SERVER_DIR}

client: ${CA_CERT}
	mkdir -p ${CLIENT_DIR}
	openssl genrsa -out ${CLIENT_KEY} 4096
	openssl req -new -sha256 \
	  -subj "/CN=${CLIENT_NAME}" \
	  -key ${CLIENT_KEY} \
	  -out ${CLIENT_CSR}
	echo extendedKeyUsage = clientAuth > ${CLIENT_CSR}.ext
	openssl x509 -req -sha256 \
	  -in ${CLIENT_CSR} \
	  -days ${CLIENT_DAYS} \
	  -CA ${CA_CERT} \
	  -CAkey ${CA_KEY} \
	  -CAcreateserial \
	  -extfile ${CLIENT_CSR}.ext \
	  -out ${CLIENT_CERT}
	rm ${CLIENT_CSR} ${CLIENT_CSR}.ext
	cp ${CA_CERT} ${CLIENT_DIR}

install:
	mkdir -p ${CERT_DIR}
	install -m0400 ${CA_KEY} ${SERVER_KEY} ${CLIENT_KEY} ${CERT_DIR}
	install -m0444 ${CA_CERT} ${SERVER_CERT} ${CLIENT_CERT} ${CERT_DIR}
	install -m0644 ${CA_SRL} ${CERT_DIR}
	mkdir -p $(dir ${SYSTEMD_FILE})
	echo [Service] > ${SYSTEMD_FILE}
	echo ExecStart= >> ${SYSTEMD_FILE}
	echo ExecStart=/usr/bin/dockerd \
	  -H=${SERVER_BIND}:2376 \
	  --tlsverify \
	  --tlscacert=$(abspath ${CERT_DIR}/${CA_CERT}) \
	  --tlscert=$(abspath ${CERT_DIR}/${SERVER_CERT}) \
	  --tlskey=$(abspath ${CERT_DIR}/${SERVER_KEY}) \
	   >> ${SYSTEMD_FILE}
	systemctl daemon-reload
