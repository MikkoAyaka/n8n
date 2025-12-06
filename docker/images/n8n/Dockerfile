ARG NODE_VERSION=22.21.0
ARG N8N_VERSION=2.0.0-rc.4

# ==============================================================================
# STAGE 1: System Dependencies & Base Setup
# ==============================================================================
FROM n8nio/base:${NODE_VERSION} AS system-deps

# 安装 Python3 和其他必要的系统依赖
RUN apk add --no-cache \
    python3 \
    py3-pip \
    build-base \
    python3-dev \
    libffi-dev \
    openssl-dev \
    && ln -sf python3 /usr/bin/python \
    && pip3 install --no-cache-dir --upgrade pip

# ==============================================================================
# STAGE 2: Application Artifact Processor
# ==============================================================================
FROM alpine:3.22.2 AS app-artifact-processor

COPY ./compiled /app/

# ==============================================================================
# STAGE 3: Final Runtime Image
# ==============================================================================
FROM system-deps AS runtime

ARG N8N_VERSION
ARG N8N_RELEASE_TYPE=dev
ENV NODE_ENV=production
ENV NODE_VERSION=${NODE_VERSION}
ENV N8N_RELEASE_TYPE=${N8N_RELEASE_TYPE}
ENV NODE_ICU_DATA=/usr/local/lib/node_modules/full-icu
ENV SHELL=/bin/sh
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1

WORKDIR /home/node

# 创建 Python 相关目录并设置权限
RUN mkdir -p /home/node/.n8n \
    && mkdir -p /usr/local/lib/python3.12/site-packages \
    && chown -R node:node /home/node \
    && chown -R node:node /usr/local/lib/python3.12

COPY --from=app-artifact-processor /app /usr/local/lib/node_modules/n8n
COPY docker/images/n8n/docker-entrypoint.sh /

RUN cd /usr/local/lib/node_modules/n8n && \
    npm rebuild sqlite3 && \
    ln -s /usr/local/lib/node_modules/n8n/bin/n8n /usr/local/bin/n8n && \
    mkdir -p /home/node/.n8n && \
    chown -R node:node /home/node

# 安装 Python 依赖和常用包
RUN cd /usr/local/lib/node_modules/n8n/node_modules/pdfjs-dist && npm install @napi-rs/canvas

# 切换到 node 用户前安装全局 Python 包（如果需要）
USER root
RUN pip3 install --no-cache-dir \
    requests \
    numpy \
    pandas \
    beautifulsoup4 \
    lxml \
    cryptography \
    pillow \
    && pip3 cache purge

# 切换回 node 用户
USER node

# 验证 Python 安装
RUN python3 --version && pip3 --version

EXPOSE 5678/tcp
USER node
ENTRYPOINT ["tini", "--", "/docker-entrypoint.sh"]

LABEL org.opencontainers.image.title="n8n" \
      org.opencontainers.image.description="Workflow Automation Tool with Python Support" \
      org.opencontainers.image.source="https://github.com/n8n-io/n8n" \
      org.opencontainers.image.url="https://n8n.io" \
      org.opencontainers.image.version=${N8N_VERSION} \
      org.opencontainers.image.authors="Your Name <your.email@example.com>" \
      org.opencontainers.image.vendor="Your Organization"

# 添加 Python 支持标识
ENV N8N_PYTHON_ENABLED=true
