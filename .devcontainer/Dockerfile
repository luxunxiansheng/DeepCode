FROM python:3.13-slim

# Non-interactive apt
ENV DEBIAN_FRONTEND=noninteractive \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=off

# Use faster mirrors for pip and npm by default (can be overridden by the user)
ENV PIP_INDEX_URL=https://pypi.tuna.tsinghua.edu.cn/simple
ENV NPM_CONFIG_REGISTRY=https://registry.npmmirror.com

# Allow switching to a China apt mirror at build time to speed up apt downloads in China
# Usage: docker build --build-arg USE_CHINA_MIRROR=true --build-arg APT_MIRROR_URL=http://mirrors.aliyun.com/ubuntu/ .
ARG USE_CHINA_MIRROR=true
ARG APT_MIRROR_URL=http://mirrors.aliyun.com/ubuntu/

# Install basic build tools, curl and Node.js (Node 20)
RUN if [ "$USE_CHINA_MIRROR" = "true" ]; then \
      sed -i "s|http://deb.debian.org/debian|${APT_MIRROR_URL}|g" /etc/apt/sources.list || true; \
      sed -i "s|http://security.debian.org/debian-security|${APT_MIRROR_URL}|g" /etc/apt/sources.list || true; \
      sed -i "s|http://archive.ubuntu.com/ubuntu/|${APT_MIRROR_URL}|g" /etc/apt/sources.list || true; \
    fi && apt-get update \
  && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    gnupg2 \
    build-essential \
    git \
    sudo \
    wget \
  && curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
  && apt-get install -y --no-install-recommends nodejs \
  && rm -rf /var/lib/apt/lists/*

# Create a non-root 'vscode' user (UID/GID 1000 by default) to match devcontainer expectations
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=1000
RUN groupadd --gid ${USER_GID} ${USERNAME} \
  && useradd --uid ${USER_UID} --gid ${USER_GID} -m ${USERNAME} \
  && usermod -aG sudo ${USERNAME} \
  && echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/${USERNAME} \
  && chmod 0440 /etc/sudoers.d/${USERNAME}

WORKDIR /workspace

# Copy workspace into image and give ownership to vscode user (helps with some container setups).
# Note: copying the whole workspace can make builds less cache-friendly; adjust if needed.
COPY . /workspace
RUN chown -R ${USERNAME}:${USERNAME} /workspace || true

USER ${USERNAME}
ENV PATH=/home/${USERNAME}/.local/bin:$PATH
CMD ["/bin/bash"]

# Final workdir for developer
WORKDIR /workspace

# Ensure npm/pip use the configured mirrors by default for the non-root user
RUN --mount=type=cache,id=pip-cache,target=/home/${USERNAME}/.cache/pip \
  mkdir -p /home/${USERNAME}/.config && \
  mkdir -p /home/${USERNAME}/.npm && \
  chown -R ${USERNAME}:${USERNAME} /home/${USERNAME} || true

# Use non-root user from now on
USER ${USERNAME}