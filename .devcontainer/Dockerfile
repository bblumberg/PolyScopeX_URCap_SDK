FROM node:20.14.0-bookworm
ENV PS_VERSION=0.16.31 \
    URSIM_VERSION=0.14.47 \
    SDK_VERSION=6.1.29
SHELL ["/bin/bash", "-c"]

RUN set -ex; \
    apt-get update && apt-get install -y \
             build-essential \
             ca-certificates \
             curl \
             gnupg \
             sudo \
             tzdata \
             unzip \
             python3 \
             python3-venv \
             python3-pip \
             binutils \
             git \
             rsync; \
    rm -rf /var/lib/apt/lists/*

#   Install docker
RUN set -ex; \
    curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg; \
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null; \
    apt-get update -y; \
    apt-get install -y \
        docker-ce \
        docker-ce-cli \
        containerd.io  \
        docker-buildx-plugin \
        docker-compose-plugin \
        gh; \
    rm -rf /var/lib/apt/lists/* 

#   Copy zipped files to /
COPY *.zip /

# Update NPM
RUN npm -g install npm@10.9.1
#   Set up non-root user
ARG USERNAME=dev
ARG USER_UID=1001
ARG USER_GID=$USER_UID
RUN set -ex; \
    groupadd --gid $USER_GID $USERNAME; \
    useradd --uid $USER_UID --gid $USER_GID -m $USERNAME; \
    usermod -aG docker $USERNAME; \
    echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME; \
    chmod 0440 /etc/sudoers.d/$USERNAME

#   unzip files, chown to dev
RUN set -ex; \
    unzip '*.zip'; \
    rm *.zip; \
    chown -R $USERNAME:$USERNAME /sdk-polyscopex-${SDK_VERSION}/ /ursim-polyscopex-${URSIM_VERSION}/  /usr/local/lib/node_modules/ /usr/local/bin

USER $USERNAME

RUN cd /ursim-polyscopex-${URSIM_VERSION}; ./install.sh
RUN cd /sdk-polyscopex-${SDK_VERSION}; ./install.sh
RUN cd /sdk-polyscopex-${SDK_VERSION}; python3 -m venv urcap-generator/venv; source urcap-generator/venv/bin/activate; pip3 install -r urcap-generator/requirements.txt