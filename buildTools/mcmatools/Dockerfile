FROM alpine:3.8
MAINTAINER Peter Schmid <peter.schmid@zuehlke.com>

# Basic build tools: make, cmake and auto tools

# User settings
ARG USER=Tron
ARG GROUP=Tron
ARG UID=1000
ARG GID=1000

RUN apk update && apk add --no-cache \
  automake=1.16.1-r0 \
  cmake=3.11.1-r2 \
  make=4.2.1-r2 \
  && apk del --purge

# Create user
RUN addgroup -g ${GID} ${GROUP} \
    && adduser -u ${UID} -G ${GROUP} -D ${USER}

USER ${USER}

ENTRYPOINT ["sh", "-c"]

# Print the tool versions if no command is given
CMD ["automake --version | head -1; cmake --version | head -1; make --version | head -1;"]
