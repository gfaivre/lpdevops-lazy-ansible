########
# Base #
########

FROM debian:bullseye-slim

ARG DEBIAN_FRONTEND="noninteractive"

ARG MANALA_USER_ID="1000"
ARG MANALA_GROUP_ID="1000"

ARG GOSU_VERSION="1.16"
ARG GOMPLATE_VERSION="3.11.3"

# The 'container' environment variable tells systemd that it's running inside a
# Docker container environment.
# It's also internally used for checking we're running inside a container.
ENV container="docker"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN \
    # Backports
    echo "deb http://deb.debian.org/debian $(. /etc/os-release; echo "${VERSION_CODENAME}")-backports main" > /etc/apt/sources.list.d/backports.list \
    && apt-get --quiet update \
    && apt-get --quiet --yes --purge --autoremove upgrade \
    && apt-get --quiet --yes --no-install-recommends --verbose-versions install \
        s6 \
        sudo \
        curl \
        ca-certificates \
        gnupg \
        libarchive-tools bzip2 \
        bash-completion \
        rsync \
        git \
        make \
        less \
        vim \
        socat \
    # Apt keyrings (debian < bookworm)
    && mkdir --verbose --parents /etc/apt/keyrings \
    # User
    && addgroup --gid ${MANALA_GROUP_ID} lazy \
    && adduser --home /home/lazy --shell /bin/bash --uid ${MANALA_USER_ID} --gecos lazy --ingroup lazy --disabled-password lazy \
    && mkdir --verbose --parents /run/user/${MANALA_USER_ID} && chown lazy:lazy /run/user/${MANALA_USER_ID} \
    && echo "lazy ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/lazy \
    # Gosu
    && curl -sSL "https://github.com/tianon/gosu/releases/download/${GOSU_VERSION}/gosu-$(case $(dpkg --print-architecture) in "amd64") echo "amd64" ;; "arm64") echo "arm64" ;; esac)" \
        --output /usr/local/bin/gosu \
    && chmod +x /usr/local/bin/gosu \
    # Gomplate
    && curl -sSL "https://github.com/hairyhenderson/gomplate/releases/download/v${GOMPLATE_VERSION}/gomplate_linux-$(case $(dpkg --print-architecture) in "amd64") echo "amd64" ;; "arm64") echo "arm64" ;; esac)" \
        --output /usr/local/bin/gomplate \
    && chmod +x /usr/local/bin/gomplate \
    # Bash completion
    && mkdir --verbose --parents /etc/bash_completion.d \
    # Oh My Bash
    && git clone https://github.com/ohmybash/oh-my-bash.git /usr/local/share/oh-my-bash \
    # Clean
    && rm -rf /var/lib/apt/lists/*

##########
# System #
##########

ARG PIPX_HOME="/usr/local"
ARG PIPX_BIN_DIR="/usr/local/bin"

RUN \
    apt-get --quiet update \
    && apt-get --quiet --yes --no-install-recommends --verbose-versions install \
        openssh-client \
        sshpass \
        python3 python3-pip \
        python3-argcomplete \
        python3-apt \
        shellcheck \
    # Bash completion
    && activate-global-python-argcomplete3 --dest /etc/bash_completion.d \
    # Clean
    && rm -rf /var/lib/apt/lists/*

# Ansible
RUN \
    pip3 --no-cache-dir --disable-pip-version-check install \
        ansible-core==2.14.2

# Run
COPY docker/entrypoint.sh /
ENTRYPOINT ["/entrypoint.sh"]
CMD ["bash"]
