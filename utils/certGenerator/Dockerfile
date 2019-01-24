FROM alpine:3.8

MAINTAINER Dominik Ebert <doe@zuehlke.com>

ARG app_dir=/var/boxer
ENV target_dir=/var/build

RUN apk update && apk add --no-cache \
  make=4.2.1-r2 \
  openssl=1.0.2q-r0 \
  && apk del --purge

RUN mkdir -p ${app_dir} && \
    mkdir -p ${target_dir}

COPY resources/Makefile ${app_dir}

WORKDIR ${app_dir}

ENTRYPOINT ["make", "OUT_DIR=${target_dir}"]
CMD ["rebuild"]