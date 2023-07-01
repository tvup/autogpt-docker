ARG GOTTY_VERSION=v1.5.0

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

FROM python:3.11-slim

COPY --chmod=+x --from=builder /build/gotty /bin/gotty

WORKDIR /app

COPY --from=builder /build/Auto-GPT/ /app



RUN apt update
RUN apt install -y git vim nano curl wget sudo
RUN apt install -y pkg-config g++ gcc
RUN apt install -y chromium firefox-esr
RUN apt install -y screen
RUN apt install -y libcairo-dev libffi-dev libjpeg-dev libxml2-dev libxslt-dev \
    gstreamer-qapt gstreamer1.0-plugins-base gstreamer1.0-plugins-good \
    libgirara-dev libgirepository1.0-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev \
    libgstreamer-plugins-bad1.0-dev gstreamer1.0-plugins-base gstreamer1.0-plugins-good \
    gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-tools \
    gstreamer1.0-x gstreamer1.0-alsa gstreamer1.0-gl gstreamer1.0-gtk3 gstreamer1.0-qt5 \
    gstreamer1.0-pulseaudio pulseaudio

ARG user=docker
ARG group=docker
ARG UID=1000
ARG USERNAME=myuser
ARG app_home=/app

RUN groupmod -g 92 audio
RUN groupmod -g 91 video

RUN adduser --disabled-password --force-badname --gecos "" --uid ${UID} ${USERNAME}

RUN adduser ${USERNAME} sudo
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' | tee -a /etc/sudoers
USER ${USERNAME}
CMD /bin/bash
RUN sudo apt-get update
RUN sudo apt-get install wget gnupg
RUN wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
RUN sudo sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list'
RUN sudo apt-get update
RUN sudo apt-get install google-chrome-stable
RUN sudo pip install --upgrade pip
RUN sudo pip install pycairo PyGObject
RUN sudo python3 -m pip install --no-cache-dir -r requirements.txt
USER root
RUN chown -R ${USERNAME}:${GID} /app
RUN chown -R ${USERNAME}:${GID} /home/${USERNAME}
RUN chmod -R a+rx /usr/bin

USER ${USERNAME}

ENV PINECONE_API_KEY=${PINECONE_API_KEY} \
    PINECONE_ENV=${PINECONE_ENV} \
    OPENAI_API_KEY=${OPENAI_API_KEY} \
    ELEVENLABS_API_KEY=${ELEVENLABS_API_KEY} \
    ELEVENLABS_VOICE_1_ID=${ELEVENLABS_VOICE_1_ID} \
    ELEVENLABS_VOICE_2_ID=${ELEVENLABS_VOICE_2_ID} \
    SMART_LLM_MODEL=${SMART_LLM_MODEL} \
    FAST_LLM_MODEL=${FAST_LLM_MODEL} \
    GOOGLE_API_KEY=${GOOGLE_API_KEY} \
    CUSTOM_SEARCH_ENGINE_ID=${CUSTOM_SEARCH_ENGINE_ID} \
    USE_AZURE=${USE_AZURE} \
    OPENAI_AZURE_API_BASE=${OPENAI_AZURE_API_BASE} \
    OPENAI_AZURE_API_VERSION=${OPENAI_AZURE_API_VERSION} \
    OPENAI_AZURE_DEPLOYMENT_ID=${OPENAI_AZURE_DEPLOYMENT_ID} \
    IMAGE_PROVIDER=${IMAGE_PROVIDER} \
    HUGGINGFACE_API_TOKEN=${HUGGINGFACE_API_TOKEN} \
    USE_MAC_OS_TTS=${USE_MAC_OS_TTS} \
    MEMORY_BACKEND=${MEMORY_BACKEND} \
    REDIS_HOST=${REDIS_HOST} \
    REDIS_PORT=${REDIS_PORT} \
    REDIS_PASSWORD=${REDIS_PASSWORD} \
    WIPE_REDIS_ON_START=${WIPE_REDIS_ON_START} \
    EXECUTE_LOCAL_COMMANDS=${EXECUTE_LOCAL_COMMANDS} \
    COMMAND_LINE_PARAMS=${COMMAND_LINE_PARAMS} \
    GITHUB_API_KEY=${GITHUB_API_KEY} \
    GITHUB_USERNAME=${GITHUB_USERNAME} \
    TW_CONSUMER_KEY=${TW_CONSUMER_KEY} \
    TW_CONSUMER_SECRET=${TW_CONSUMER_SECRET} \
    TW_ACCESS_TOKEN=${TW_ACCESS_TOKEN} \
    TW_ACCESS_TOKEN_SECRET=${TW_ACCESS_TOKEN_SECRET} \
    TW_BEARER_TOKEN=${TW_BEARER_TOKEN}

EXPOSE 8080

WORKDIR /app
