FROM alpine:3.5

LABEL maintainer='amaya <mail@sapphire.in.net>'

ENV SCRIPT_DIR='/usr/local/bin'
ENV TURNOUT='/usr/local/turnout/mirakurun'
ENV CONF_DIR='/usr/local/etc/mirakurun'

COPY service $SCRIPT_DIR

RUN set -eux \
	&& apk add --no-cache \
		bash \
		nodejs \
		coreutils \
		libusb \
		pcsc-lite \
		pcsc-lite-libs \
		ca-certificates \
	\
	&& apk add --no-cache --virtual .build-deps \
		curl \
		git \
		make \
		gcc \
		g++ \
		gzip \
		autoconf \
		automake \
		libc-dev \
		musl-dev \
		eudev-dev \
		libevent-dev \
		pcsc-lite-dev \
		libusb-dev \
	\
	&& npm install pm2 -g \
	&& npm install arib-b25-stream-test -g --unsafe \
	\
	&& : > $SCRIPT_DIR/update-rc.d \
	&& chmod +x $SCRIPT_DIR/update-rc.d \
	&& npm install mirakurun -g --unsafe --production \
	&& mkdir -p $TURNOUT \
	&& mv $CONF_DIR/* $TURNOUT \
	\
	# recpt1
	&& git clone https://github.com/stz2012/recpt1 /tmp/recpt1 \
	&& cd /tmp/recpt1/recpt1 \
	&& ./autogen.sh \
	&& ./configure \
	&& sed -i '/#include <sys\/msg.h>/d' recpt1core.h \
	&& sed -i -E 's!(#include <sys/msg.h>)!#undef _GNU_SOURCE\n#undef _BSD_SOURCE\n\1!' recpt1.c \
	&& sed -i -E 's!(#include <sys/msg.h>)!#undef _GNU_SOURCE\n#undef _BSD_SOURCE\n\1!' recpt1ctl.c \
	&& sed -i -E 's!(#include <sys/msg.h>)!#undef _GNU_SOURCE\n#undef _BSD_SOURCE\n\1!' checksignal.c \
	&& sed -i -E 's!(#include <ctype.h>)!\1\n#include <event.h>!' tssplitter_lite.c \
	&& sed -i 's#-I../driver#-I../driver -I/usr/local/include#' Makefile \
	&& make \
	&& make install \
	\
	# ccid
	&& cd /tmp \
	&& curl https://alioth.debian.org/frs/download.php/file/4171/ccid-1.4.24.tar.bz2 -o ccid-latest.tar.bz2 \
	&& tar xvf ccid-*.tar.bz2 \
	&& cd ccid-* \
	&& ./configure \
	&& make \
	&& make install \
	\
	&& apk del --purge .build-deps \
	&& npm cache clean \
	&& rm -rf /tmp/* ~/.npm

EXPOSE 40772

CMD ["service"]
