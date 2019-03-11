FROM alpine:3.9

MAINTAINER Dominik Ebert <doe@zuehlke.com>

ARG app_dir=/opt/utils
ENV target_dir=/var/build

# User settings
ARG USER=Tron
ARG GROUP=Tron
ARG UID=1000
ARG GID=1000

RUN apk update && apk add --no-cache \
  make=4.2.1-r2 \
  openssl=1.1.1a-r1 \
  && apk del --purge

# Create user
RUN addgroup -g ${GID} ${GROUP} \
    && adduser -u ${UID} -G ${GROUP} -D ${USER}
  
RUN mkdir -p ${app_dir} && \
    mkdir -p ${target_dir}

COPY resources/Makefile ${app_dir}

RUN chown -R ${USER}:${GROUP} ${app_dir}
RUN chown -R ${USER}:${GROUP} ${target_dir}

USER ${USER}

VOLUME ${target_dir}
WORKDIR ${app_dir}

ENTRYPOINT ["make", "OUT_DIR=${target_dir}"]
CMD ["rebuild"]