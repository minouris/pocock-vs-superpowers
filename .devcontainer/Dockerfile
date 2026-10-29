ARG BASE_IMAGE=mcr.microsoft.com/devcontainers/base:ubuntu-24.04
FROM ${BASE_IMAGE}

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    ca-certificates \
    curl \
    git \
    gnupg \
    jq \
    gawk \
    lsb-release \
    make \
    unzip \
    python3 \
    python3-pip \
    python3-venv \
    && rm -rf /var/lib/apt/lists/*

RUN update-ca-certificates

# Install Node.js 22.x (LTS) from NodeSource (apt's npm ships Node 18)
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - && \
    apt-get install -y nodejs && \
    rm -rf /var/lib/apt/lists/*

ENV NODE_EXTRA_CA_CERTS=/etc/ssl/certs/ca-certificates.crt

RUN npm install -g pyright

USER vscode

RUN mkdir -p /home/vscode/.npm-global && \
    npm config set prefix /home/vscode/.npm-global

ENV PATH="/home/vscode/.npm-global/bin:${PATH}"

ENV PIP_BREAK_SYSTEM_PACKAGES=1

RUN curl -fsSL https://opencode.ai/install | bash
