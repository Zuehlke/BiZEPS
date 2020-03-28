FROM alpine:3.11.3
MAINTAINER Dominik Ebert <doe@zuehlke.com>

# Modified official Jenkins Dockerfile
# See: https://github.com/jenkinsci/docker

# The MIT License
#
# Copyright (c) 2014-, Michael Neale, Nicolas de Loof, Carlos Sanchez, and a number of other of contributors
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.

ARG CPU_ARCH=x86_64
ARG DOCKER_CLIENT_VERSION=19.03.5

RUN apk update && apk add --no-cache \
  bash \
  coreutils \
  curl \
  git \
  openssh-client \
  openjdk8-jre \
  ttf-dejavu \
  tini \
  unzip \
  && apk del --purge

# Definition of jenkins user
ARG user=jenkins
ARG group=jenkins
ARG uid=1000
ARG gid=1000
ARG http_port=8080
ARG agent_port=50000

# jenkins version being bundled in this docker image and checksum
ARG JENKINS_VERSION
ENV JENKINS_VERSION ${JENKINS_VERSION:-2.222.1}
ARG JENKINS_SHA=5a6cbb836ceb79728c2d9f72645d0680f789cdb09a44485076aba6143bea953e

ENV JENKINS_HOME /var/jenkins_home
ENV JENKINS_SLAVE_AGENT_PORT ${agent_port}

# Jenkins is run with user `jenkins`, uid = 1000
# If you bind mount a volume from the host or a data container,
# ensure you use the same uid
RUN mkdir -p $JENKINS_HOME \
    && chown ${uid}:${gid} $JENKINS_HOME \
    && addgroup -g ${gid} ${group} \
    && adduser -h "$JENKINS_HOME" -u ${uid} -G ${group} -s /bin/bash -D ${user}

# Install docker client to connect to a docker host from the container
# Downloaded from 'https://download.docker.com/linux/static/stable/'
RUN mkdir -p /var/tmp
COPY resources/${CPU_ARCH}/docker-client-${DOCKER_CLIENT_VERSION}.tgz /var/tmp
RUN tar xzvf /var/tmp/docker-client-${DOCKER_CLIENT_VERSION}.tgz -C /var/tmp \
    && cp /var/tmp/docker/docker /usr/local/bin/docker \
    && rm -rf /var/tmp

# `/usr/share/jenkins/ref/` contains all reference configuration we want
# to set on a fresh new installation. Use it to bundle additional plugins
# or config file with your custom jenkins Docker image.
RUN mkdir -p /usr/share/jenkins/ref/init.groovy.d

COPY resources/init.groovy /usr/share/jenkins/ref/init.groovy.d/tcp-slave-agent-port.groovy

# Can be used to customize where jenkins.war get downloaded from
ARG JENKINS_URL=https://repo.jenkins-ci.org/public/org/jenkins-ci/main/jenkins-war/${JENKINS_VERSION}/jenkins-war-${JENKINS_VERSION}.war

# could use ADD but this one does not check Last-Modified header neither does it allow to control checksum
# see https://github.com/docker/docker/issues/8331
RUN curl -fsSL ${JENKINS_URL} -o /usr/share/jenkins/jenkins.war \
  && sha256sum "/usr/share/jenkins/jenkins.war" \
  && echo "${JENKINS_SHA}  /usr/share/jenkins/jenkins.war" | sha256sum -c -

ENV JENKINS_UC https://updates.jenkins.io
ENV JENKINS_UC_EXPERIMENTAL=https://updates.jenkins.io/experimental
RUN chown -R ${user} "$JENKINS_HOME" /usr/share/jenkins/ref

# for main web interface:
EXPOSE ${http_port}

# will be used by attached slave agents:
EXPOSE ${agent_port}

ENV COPY_REFERENCE_FILE_LOG $JENKINS_HOME/copy_reference_file.log

# Jenkins home directory is a volume, so configuration and build history
# can be persisted and survive image upgrades
VOLUME $JENKINS_HOME

USER ${user}

COPY resources/jenkins-support /usr/local/bin/jenkins-support
COPY resources/jenkins.sh /usr/local/bin/jenkins.sh

# Setup and install predefined plugins
COPY resources/plugins.sh /usr/local/bin/plugins.sh
COPY resources/install-plugins.sh /usr/local/bin/install-plugins.sh
COPY resources/plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins.txt

ENTRYPOINT ["/sbin/tini", "--", "/usr/local/bin/jenkins.sh"]
