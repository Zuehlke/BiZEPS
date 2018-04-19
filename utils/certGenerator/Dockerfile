FROM alpine:3.7

MAINTAINER Dominik Ebert <doe@zuehlke.com>

ARG app_dir=/var/boxer
ENV target_dir=/var/build

RUN apk add --no-cache \
  make \
  openssl

RUN mkdir -p ${app_dir} && \
    mkdir -p ${target_dir}

COPY resources/Makefile ${app_dir}

WORKDIR ${app_dir}

ENTRYPOINT ["make", "OUT_DIR=${target_dir}"]
CMD ["rebuild"]