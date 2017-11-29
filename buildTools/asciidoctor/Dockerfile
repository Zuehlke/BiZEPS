FROM debian:jessie
MAINTAINER Dominik Ebert <doe@zuehlke.com>

# Asciidoctor Slave Image

ARG ASCIID_MAIN_VERSION="1.5.4"
ARG ASCIID_DIAGRAMM_VERSION="1.4.0"
ARG ASCIID_PDF_VERSION="1.5.0.alpha.11"

RUN apt-get update && apt-get install -y \
  graphviz \
  ruby2.1 \
  rubygems \
  && rm -rf /var/lib/apt/lists/*

RUN gem install --no-ri --no-rdoc asciidoctor --version ${ASCIID_MAIN_VERSION} \
  && gem install --no-ri --no-rdoc asciidoctor-diagram --version ${ASCIID_DIAGRAMM_VERSION} \
  && gem install --no-ri --no-rdoc asciidoctor-pdf --version ${ASCIID_PDF_VERSION}

CMD ["/bin/bash"]
