# trame-tools base dev environment image
# Provides: Node.js, Git 2.48+, gosu, MCP server
FROM node:22-bookworm-slim

ENV DEBIAN_FRONTEND=noninteractive

# System dependencies + gosu for entrypoint user switching
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    gnupg \
    ca-certificates \
    python3 \
    build-essential \
    gosu \
    libcurl4-gnutls-dev \
    libexpat1-dev \
    gettext \
    libz-dev \
    libssl-dev \
    && rm -rf /var/lib/apt/lists/*

# Git >= 2.48 from source — needed for --relative-paths worktree support
RUN cd /tmp \
    && curl -sL https://github.com/git/git/archive/refs/tags/v2.48.1.tar.gz | tar xz \
    && cd git-2.48.1 \
    && make -j$(nproc) prefix=/usr all \
    && make prefix=/usr install \
    && cd / && rm -rf /tmp/git-2.48.1

# Claude Code CLI — native installer, symlinked into /usr/local/bin for all users
RUN curl -fsSL https://claude.ai/install.sh | bash \
    && cp /root/.local/bin/claude /usr/local/bin/claude \
    && chmod +x /usr/local/bin/claude

# MCP server — provides shell_exec tool for Claude Code on the host
RUN npm install -g @modelcontextprotocol/sdk zod
COPY mcp-server.mjs /usr/local/lib/mcp-server.mjs
RUN printf '#!/bin/sh\nexec node /usr/local/lib/mcp-server.mjs "$@"\n' > /usr/local/bin/mcp-server \
    && chmod +x /usr/local/bin/mcp-server

# Entrypoint: maps container UID/GID to host user, then drops privileges
COPY entrypoint.sh /usr/local/bin/entrypoint.sh
RUN chmod +x /usr/local/bin/entrypoint.sh

WORKDIR /workspace
VOLUME /workspace

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
CMD ["bash"]
