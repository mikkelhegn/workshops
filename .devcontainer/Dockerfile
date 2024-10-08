# The goal of this Dockerfile is to be architecture independent. To that end,
# it avoids downloading any platform-specific binaries, and installs the required
# tools either through Debian's package manager, or through installation scripts
# that download the appropriate binaries.
FROM mcr.microsoft.com/devcontainers/base:ubuntu

# provided by docker
ARG TARGETARCH
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Install Python
RUN sudo apt update
RUN sudo apt install python3.10-venv -y

# Install Go
ARG GO_VERSION="1.23.1"
RUN wget -O go.tar.gz "https://go.dev/dl/go${GO_VERSION}.linux-${TARGETARCH}.tar.gz" && \
    tar -C /usr/local -xzf go.tar.gz && \
    rm go.tar.gz
ENV PATH="$PATH:/usr/local/go/bin"

# Install TinyGo
ARG TINYGO_VERSION="0.33.0"
RUN wget "https://github.com/tinygo-org/tinygo/releases/download/v${TINYGO_VERSION}/tinygo_${TINYGO_VERSION}_${TARGETARCH}.deb" && \
    dpkg -i "tinygo_${TINYGO_VERSION}_${TARGETARCH}.deb"

# Install Rust
ENV PATH="$PATH:/home/$USERNAME/.cargo/bin"
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | bash -s -- \
    -y --target wasm32-wasi

# Set the current user before node and spin install for plugins/templates
USER $USERNAME

# Install Spin
ARG SPIN_VERSION="v2.7.0"
RUN mkdir -p /opt/spin && \
    cd /opt/spin && \
    curl -fsSL https://developer.fermyon.com/downloads/install.sh | bash -s -- -v $SPIN_VERSION && \
    ln -s /opt/spin/spin /usr/local/bin/spin

# Install Node and NPM
ARG NODE_VERSION=20
ENV NVM_DIR=/home/${USERNAME}/.nvm
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
RUN . "$NVM_DIR/nvm.sh" && nvm install ${NODE_VERSION}
ENV PATH="/root/.nvm/versions/node/v${NODE_VERSION}/bin/:${PATH}"

# Install Spin Plugins & Templates
RUN spin plugins update && \
    spin plugins install check-for-update -y && \
    spin plugins install cloud -y && \
    spin plugins install js2wasm -y && \
    spin plugins install py2wasm -y && \
    spin templates install --upgrade --git https://github.com/fermyon/spin && \
    spin templates install --upgrade --git https://github.com/fermyon/spin-js-sdk && \
    spin templates install --upgrade --git https://github.com/fermyon/spin-python-sdk && \
    spin templates install --upgrade --git https://github.com/fermyon/spin-kv-explorer
