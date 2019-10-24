FROM alpine:3.10.3

# User settings
ARG USER=Tron
ARG GROUP=Tron
ARG UID=1000
ARG GID=1000

ARG AWS_VERSION=1.16.265

RUN apk update && apk add --no-cache \
  groff \
  python3 \
  && apk del --purge

RUN pip3 install awscli==${AWS_VERSION}

# Create user
RUN addgroup -g ${GID} ${GROUP} \
    && adduser -u ${UID} -G ${GROUP} -D ${USER}

USER ${USER}

ENTRYPOINT ["aws"]
