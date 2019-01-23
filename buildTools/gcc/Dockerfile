FROM bizeps/mcmatools:19.01-r1
MAINTAINER Peter Schmid <peter.schmid@zuehlke.com>

# An image with g++ and basic build tools

# User settings
ARG USER=Tron
ARG GROUP=Tron
ARG UID=1000
ARG GID=1000

USER ${ROOT}

RUN apk update && apk add --no-cache \
  g++=6.4.0-r9 \
  && apk del --purge

USER ${USER}

ENTRYPOINT ["sh", "-c"]

# Without an argument print the tool version
CMD ["g++ --version | head -1"]
