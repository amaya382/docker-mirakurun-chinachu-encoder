FROM alpine:3.5

LABEL maintainer='amaya <mail@sapphire.in.net>'

ARG REPOSITORY='https://github.com/Chinachu/Chinachu.git'
ARG BRANCH='gamma'

ENV CHINACHU_HOME='/usr/local/chinachu'
ENV SCRIPT_DIR='/usr/local/bin'
ENV RECORDED_DIR="$CHINACHU_HOME/recorded"
ENV DUMP_SITE="$RECORDED_DIR/dump"

WORKDIR $CHINACHU_HOME

COPY chinachu-mp4.patch $SCRIPT_DIR
COPY service $SCRIPT_DIR

RUN set -eux && \
    apk add --no-cache \
        bash \
        curl \
        nodejs \
        coreutils \
        procps \
        ca-certificates \
        jq && \
    \
    apk add --no-cache --virtual .build-deps \
        git \
        make \
        gcc \
        g++ \
        autoconf \
        automake \
        wget \
        tar \
        xz \
        libc-dev \
        musl-dev \
        eudev-dev \
        libevent-dev && \
    git clone -b $BRANCH --single-branch $REPOSITORY $CHINACHU_HOME && \
    cd $CHINACHU_HOME && \
    echo 1 | ./chinachu installer && \
    patch -p1 < $SCRIPT_DIR/chinachu-mp4.patch && \
    rm $SCRIPT_DIR/chinachu-mp4.patch && \
    mkdir turnout && \
    cat config.sample.json | \
        jq ".mirakurunPath = \"http://mirakurun:40772/\" | .recordedDir = \"$RECORDED_DIR/\" | .storageLowSpaceAction = \"none\" | .storageLowSpaceCommand = \"$SCRIPT_DIR/storage_low_space_command\" | .recordedCommand = \"$SCRIPT_DIR/recorded_command\"" \
        > turnout/config.json && \
    echo [] > turnout/rules.json && \
    rm *.sample.json && \
    ./chinachu service operator initscript > /etc/init.d/chinachu-operator && \
    ./chinachu service wui initscript > /etc/init.d/chinachu-wui && \
    chmod +x /etc/init.d/* && \
    \
    apk del --purge .build-deps && \
    rm -rf .git ~/.npm

EXPOSE 20772

CMD ["service"]
