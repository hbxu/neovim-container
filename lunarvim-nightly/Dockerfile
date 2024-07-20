# Set default alpine tag.
ARG ALPINE=edge
ARG GID=10000
ARG UID=10001
ARG USER=lunaruser
ARG GROUP=$USER

#####

# Base image.
FROM alpine:$ALPINE AS builder
LABEL stage=builder

# Install minimal required.
RUN apk update --no-cache; \
    apk upgrade --no-cache; \
    apk add --no-cache \
      alpine-sdk \
      bash \
      cargo \
      curl \
      git \
      neovim \
      python3 \
      ripgrep \
      yarn

# Always run non-privileged, creating a new user.
# Default set UID to >10000, to decrease attack vector.
# Renew ARG
ARG GID
ARG UID
ARG USER
ARG GROUP

ENV HOME=/home/$USER

RUN set -eux; \
    addgroup --gid $GID $GROUP; \
    adduser --uid $UID $USER --ingroup $USER --home $HOME --disabled-password

WORKDIR $HOME
USER $GROUP:$USER

# Install and configure applications.
RUN set -eux; \
    bash <(curl -s https://raw.githubusercontent.com/lunarvim/lunarvim/master/utils/installer/install.sh) --no-install-dependencies

RUN set -eux; \
    echo 'Set files permission'; \
    chown -R $GROUP:$USER $HOME

###########################

FROM alpine:$ALPINE

# Install minimal required.
RUN apk update --no-cache; \
    apk upgrade --no-cache; \
    apk add --no-cache \
      bash \
      git \
      neovim

# Always run non-privileged, creating a new user.
# Default set UID to >10000, to decrease attack vector.
# Renew ARG
ARG GID
ARG UID
ARG USER
ARG GROUP

ENV HOME=/home/$USER

RUN set -eux; \
    mkdir -p $HOME; \
    addgroup --gid $GID $GROUP; \
    adduser --uid $UID $USER --ingroup $USER --home $HOME --disabled-password; \
    echo '\
        . /etc/profile ; \
        export PATH="$PATH:.local/bin" ; \
        ' > $HOME/.profile

COPY --from=builder $HOME/.local/ $HOME/.local/

RUN set -eux; \
    echo 'Set files permission'; \
    mkdir -p $HOME; \
    chown -R $GROUP:$USER $HOME

WORKDIR $HOME
USER $GROUP:$USER

#####
# FOR MY OWN CONVENIENCE:
# build> podman build . -t lvim:nightly-240720-alpine
