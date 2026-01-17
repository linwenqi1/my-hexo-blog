FROM mcr.microsoft.com/devcontainers/javascript-node:24

# 安装 Pandoc
RUN apt-get update && apt-get install -y \
    pandoc \
    && rm -rf /var/lib/apt/lists/*