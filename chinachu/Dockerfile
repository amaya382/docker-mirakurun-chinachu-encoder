FROM alpine:3.5

LABEL maintainer='amaya <mail@sapphire.in.net>'

ARG CHINACHU_REPOSITORY='https://github.com/Chinachu/Chinachu.git'
ARG CHINACHU_COMMIT='f1a825439f29341da805cbe7a1f01fd81777fad1'
ARG FFMPEG_REPOSITORY='https://github.com/FFmpeg/FFmpeg.git'
ARG FFMPEG_COMMIT='27ebdcf079ed54d294db010e9f50700f75ed5e3d'

ENV CHINACHU_UID=1000
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
      jq \
      \
      libva \
      libva-intel-driver \
      zlib-dev \
      yasm-dev \
      lame-dev \
      libva-dev \
      libogg-dev \
      libvpx-dev \
      libvorbis-dev \
      x264-dev \
      x265-dev \
      freetype-dev \
      libass-dev \
      libwebp-dev \
      rtmpdump-dev \
      libtheora-dev \
      opus-dev && \
    apk add --no-cache --virtual .build-deps \
      build-base \
      git \
      autoconf \
      automake \
      xz \
      musl-dev \
      eudev-dev \
      libevent-dev \
      nasm \
      bzip2 && \
    adduser -D -u $CHINACHU_UID -h $CHINACHU_HOME chinachu && \
    git clone $CHINACHU_REPOSITORY $CHINACHU_HOME && \
    git checkout $CHINACHU_COMMIT && \
    patch -p1 < $SCRIPT_DIR/chinachu-mp4.patch && \
    echo 1 | ./chinachu installer && \
    mkdir turnout conf && \
    cat config.sample.json | \
      jq ".uid = \"chinachu\" \
        | .mirakurunPath = \"http://mirakurun:40772/\" \
        | .recordedDir = \"$RECORDED_DIR/\" \
        | .storageLowSpaceAction = \"none\" \
        | .storageLowSpaceCommand = \"$SCRIPT_DIR/storage_low_space_command\" \
        | .recordedCommand = \"$SCRIPT_DIR/recorded_command\"" \
      > turnout/config.json && \
    echo [] > turnout/rules.json && \
    ln -s conf/config.json config.json && \
    ln -s conf/rules.json rules.json && \
    rm $SCRIPT_DIR/chinachu-mp4.patch *.sample.json && \
    chown chinachu. -R $CHINACHU_HOME && \
    ./chinachu service operator initscript | sed -e "s/^USER=.*$/USER=chinachu/" > /etc/init.d/chinachu-operator && \
    ./chinachu service wui initscript | sed -e "s/^USER=.*$/USER=chinachu/" > /etc/init.d/chinachu-wui && \
    chmod +x /etc/init.d/* && \
    \
    rm -rf usr/bin/* usr/src/* && \
    git clone $FFMPEG_REPOSITORY ffmpeg && \
    cd ffmpeg && \
    git checkout $FFMPEG_COMMIT && \
    ./configure \
      --enable-version3 \
      --enable-gpl \
      --enable-nonfree \
      --enable-small \
      --enable-libmp3lame \
      --enable-libx264 \
      --enable-libx265 \
      --enable-libvpx \
      --enable-libtheora \
      --enable-libvorbis \
      --enable-libopus \
      --enable-libass \
      --enable-libwebp \
      --enable-librtmp \
      --enable-postproc \
      --enable-avresample \
      --enable-libfreetype \
      --disable-debug \
      --disable-doc && \
    make && \
    make install && \
    cd .. && \
    \
    apk del --purge .build-deps && \
    rm -rf ffmpeg .git ~/.npm

EXPOSE 20772

CMD ["service"]
