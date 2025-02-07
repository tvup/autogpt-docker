ARG GOTTY_VERSION=v1.5.0
ARG BUILD_TYPE=dev

FROM alpine:3.15 AS builder

ARG GOTTY_VERSION

WORKDIR /build

ADD https://github.com/sorenisanerd/gotty/releases/download/${GOTTY_VERSION}/gotty_${GOTTY_VERSION}_linux_arm64.tar.gz gotty-aarch64.tar.gz
ADD https://github.com/sorenisanerd/gotty/releases/download/${GOTTY_VERSION}/gotty_${GOTTY_VERSION}_linux_amd64.tar.gz gotty-x86_64.tar.gz

RUN tar -xzvf "gotty-$(uname -m).tar.gz"
RUN apk update
RUN apk add --no-cache git
RUN git clone -b stable https://github.com/tvup/Auto-GPT.git

##############################

FROM python:3.10-slim AS autogpt-base

COPY --chmod=+x --from=builder /build/gotty /bin/gotty

WORKDIR /app

COPY --from=builder /build/Auto-GPT/ /app

RUN pip install --upgrade pip

# Install browsers
RUN apt-get update && apt-get install -y \
    chromium-driver firefox-esr ca-certificates \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install utilities
RUN apt-get update && apt-get install -y \
    curl jq wget git nano unzip \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Set environment variables
ENV PIP_NO_CACHE_DIR=yes \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1

# Install the required python packages globally
ENV PATH="$PATH:/root/.local/bin"
ENV OPENAI_API_KEY=${OPENAI_API_KEY} \
    ELEVENLABS_API_KEY=${ELEVENLABS_API_KEY} \
    ELEVENLABS_VOICE_ID=${ELEVENLABS_VOICE_ID} \
    SMART_LLM_MODEL=${SMART_LLM_MODEL} \
    FAST_LLM_MODEL=${FAST_LLM_MODEL} \
    GOOGLE_API_KEY=${GOOGLE_API_KEY} \
    GOOGLE_CUSTOM_SEARCH_ENGINE_ID=${GOOGLE_CUSTOM_SEARCH_ENGINE_ID} \
    USE_AZURE=${USE_AZURE} \
    IMAGE_PROVIDER=${IMAGE_PROVIDER} \
    HUGGINGFACE_API_TOKEN=${HUGGINGFACE_API_TOKEN} \
    MEMORY_BACKEND=${MEMORY_BACKEND} \
    REDIS_HOST=${REDIS_HOST} \
    REDIS_PORT=${REDIS_PORT} \
    REDIS_PASSWORD=${REDIS_PASSWORD} \
    WIPE_REDIS_ON_START=${WIPE_REDIS_ON_START} \
    EXECUTE_LOCAL_COMMANDS=${EXECUTE_LOCAL_COMMANDS} \
    COMMAND_LINE_PARAMS=${COMMAND_LINE_PARAMS} \
    GITHUB_API_KEY=${GITHUB_API_KEY} \
    GITHUB_USERNAME=${GITHUB_USERNAME}

COPY requirements.txt .

# Set the entrypoint
ENTRYPOINT gotty --port 50001 --permit-write --title-format "AutoGPT Terminal" /bin/bash -c "python -m autogpt ${COMMAND_LINE_PARAMS}"

# dev build -> include everything
FROM autogpt-base as autogpt-dev
RUN pip install --no-cache-dir -r requirements.txt
WORKDIR /app
ONBUILD COPY requirements.txt ./
ONBUILD COPY plugins_config.yaml ./
ONBUILD COPY prompt_settings.yaml ./
ONBUILD COPY Auto-GPT-Plugins.zip ./plugins/
ONBUILD RUN touch /.dockerenv

# release build -> include bare minimum
FROM autogpt-base as autogpt-release
RUN sed -i '/Items below this point will not be included in the Docker Image/,$d' requirements.txt && \
	pip install --no-cache-dir -r requirements.txt
WORKDIR /app
ONBUILD COPY autogpt/ ./autogpt
ONBUILD COPY scripts/ ./scripts
ONBUILD COPY plugins/ ./plugins
ONBUILD COPY prompt_settings.yaml ./prompt_settings.yaml
ONBUILD COPY plugins_config.yaml ./plugins_config.yaml
ONBUILD RUN mkdir ./data
ONBUILD RUN touch /.dockerenv

FROM autogpt-${BUILD_TYPE} AS auto-gpt
