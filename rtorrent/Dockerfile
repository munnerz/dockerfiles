FROM alpine:3.6 AS build

RUN apk add --no-cache \
	  git \
	  make \
	  automake \
	  autoconf \
	  libtool \
	  openssl \
	  ca-certificates \
	  g++ \
	  ncurses-dev \
	  curl-dev \
	  zlib-dev \
	  openssl-dev \
	  linux-headers

ENV RTORRENT_REVISION 226e670decf92e7adaa845a6982aca4f164ea740
ENV LIBTORRENT_REVISION c167c5a9e0bcf0df23ae5efd91396aae0e37eb87
ENV XMLRPC_VERSION 1.39.12

ENV OUTPUT_DIR /out
ENV PATH $PATH:$OUTPUT_DIR/bin
ENV PKG_CONFIG_PATH $OUTPUT_DIR/lib/pkgconfig

WORKDIR /tmp

RUN mkdir -p $OUTPUT_DIR

RUN wget "https://downloads.sourceforge.net/project/xmlrpc-c/Xmlrpc-c%20Super%20Stable/$XMLRPC_VERSION/xmlrpc-c-$XMLRPC_VERSION.tgz" \
	&& tar -C $OUTPUT_DIR/ -xvf xmlrpc-c-$XMLRPC_VERSION.tgz \
	&& cd $OUTPUT_DIR/xmlrpc-c-$XMLRPC_VERSION \
	&& ./configure --prefix=$OUTPUT_DIR \
	&& make \
	&& make install

RUN git clone https://github.com/rakshasa/libtorrent \
	&& cd libtorrent \
	&& git reset --hard $LIBTORRENT_REVISION \
	&& ./autogen.sh \
	&& ./configure --prefix=$OUTPUT_DIR --enable-static --disable-shared \
	&& make -j2 \
	&& make install

ENV libtorrent_CFLAGS "-I$OUTPUT_DIR/include"
ENV libtorrent_LIBS "-L$OUTPUT_DIR/lib"
ENV LDFLAGS "-L$OUTPUT_DIR/lib"
ENV CPPFLAGS "-I$OUTPUT_DIR/include"

RUN git clone https://github.com/rakshasa/rtorrent \
	&& cd rtorrent \
	&& git reset --hard $RTORRENT_REVISION \
	&& ./autogen.sh \
	&& ./configure --prefix=$OUTPUT_DIR --enable-static --disable-shared --with-xmlrpc-c \
	&& make -j2

RUN git clone https://github.com/Novik/ruTorrent.git /tmp/rutorrent

FROM alpine:3.6

# s6 overlay
RUN apk add --no-cache curl \
	&& curl -L -s https://github.com/just-containers/s6-overlay/releases/download/v1.21.1.0/s6-overlay-amd64.tar.gz \
		| tar xvzf - -C /

# Install rTorrent dependencies
RUN apk add --no-cache \
		ncurses \
		libcurl \
		zlib \
		libstdc++ \
		libgcc \
		openssl-dev

COPY --from=build /out /out
COPY --from=build /tmp/rtorrent/src/rtorrent /usr/local/bin/rtorrent

ENV LD_LIBRARY_PATH /out/lib

# Install ruTorrent dependencies
RUN apk add --no-cache \
		nginx \
		ca-certificates \
		fcgi \
		ffmpeg \
		geoip \
		gzip \
		nginx \
		php7 \
		php7-cgi \
		php7-fpm \
		php7-json  \
		php7-mbstring \
		php7-pear \
		tar \
		unrar \
		unzip \
		wget \
		zip
RUN apk add --no-cache \
	--repository http://nl.alpinelinux.org/alpine/edge/testing \
	mediainfo

COPY --from=build /tmp/rutorrent/ /var/www/html
COPY rutorrent.conf /etc/nginx/conf.d/rutorrent.conf

RUN chown -R nginx:nginx /var/www/html /var/log/php7 \
	&& rm -f /etc/nginx/conf.d/default.conf \
	&& mkdir -p /run/nginx

VOLUME /session

COPY root/ /

ENTRYPOINT ["/init"]
