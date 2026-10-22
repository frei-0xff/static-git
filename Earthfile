VERSION 0.8

FROM alpine:3.22

build:

    ARG GIT_VERSION=2.56.0
    ARG CURL_VERSION=8.12.1

    RUN apk add --no-cache \
        bash \
        ca-certificates \
        curl \
        autoconf \
        automake \
        gcc \
        g++ \
        make \
        musl-dev \
        linux-headers \
        perl \
        pkgconf \
        libtool \
        gettext \
        gettext-dev \
        zlib-dev \
        zlib-static \
        openssl-dev \
        openssl-libs-static \
        xz \
        tar \
        bison \
        flex \
        tcl \
        tk \
        tcl-dev \
        tk-dev

    WORKDIR /tmp

    # Build a minimal static libcurl using OpenSSL and zlib.
    # Disable optional libraries that caused static-linker conflicts.
    RUN curl -fsSL \
          "https://curl.se/download/curl-${CURL_VERSION}.tar.xz" \
          -o curl.tar.xz \
        && tar -xJf curl.tar.xz

    WORKDIR /tmp/curl-${CURL_VERSION}

    RUN ./configure \
          --prefix=/opt/curl-static \
          --disable-shared \
          --enable-static \
          --with-openssl \
          --with-zlib \
          --without-libidn2 \
          --without-libpsl \
          --without-nghttp2 \
          --without-brotli \
          --without-zstd \
        && make -j2 \
        && make install

    # Ensure Git's build uses the custom libcurl installation.
    ENV PATH="/opt/curl-static/bin:${PATH}"
    ENV CPPFLAGS="-I/opt/curl-static/include"
    ENV LDFLAGS="-L/opt/curl-static/lib -static"

    # Download and unpack Git.
    WORKDIR /tmp

    RUN curl -fsSL \
          "https://github.com/git/git/archive/refs/tags/v${GIT_VERSION}.tar.gz" \
          -o git.tar.gz \
        && tar -xzf git.tar.gz

    WORKDIR /tmp/git-${GIT_VERSION}

    # Generate Git's configure script and configure for a static executable.
    RUN make configure \
        && ./configure \
          --prefix="/dist/git-${GIT_VERSION}" \
          CFLAGS="-O2 -static" \
          LDFLAGS="-static -L/opt/curl-static/lib"

    # Build and install Git with static libcurl.
    RUN make -j2 CURL_CONFIG=/opt/curl-static/bin/curl-config CURL_STATIC=YesPlease \
        && make CURL_CONFIG=/opt/curl-static/bin/curl-config CURL_STATIC=YesPlease install

    # Export the installed Git executables.
    SAVE ARTIFACT /dist/git-${GIT_VERSION}/bin/git AS LOCAL ./dist/git
    SAVE ARTIFACT /dist/git-${GIT_VERSION}/bin/git-* AS LOCAL ./dist/
