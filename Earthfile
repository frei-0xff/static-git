VERSION 0.6

FROM alpine

RUN apk add --no-cache \
    curl curl-dev curl-static \
    openssl-dev ca-certificates \
    pkgconf \
    autoconf gcc flex bison make bash cmake libtool musl-dev g++ \
    linux-headers \
    zlib-dev zlib-static \
    tcl tk tcl-dev gettext \
    cargo rust

ARG GIT_VERSION

build:
    RUN curl -fsSL https://github.com/git/git/archive/v${GIT_VERSION}.tar.gz -o git.tar.gz
    RUN tar zxf git.tar.gz
    WORKDIR /git-${GIT_VERSION}

    RUN make configure && sed -i 's/qversion/-version/g' configure

    # Confirm curl supports TLS and static linking.
    RUN curl-config --version
    RUN curl-config --features
    RUN curl-config --static-libs

    # Configure Git with static linking.
    RUN ./configure prefix=/dist/git-${GIT_VERSION} CFLAGS="-O2 -static" LDFLAGS="-static"

    # Explicitly provide static curl and TLS libraries.
    RUN make -j2 CURL_LDFLAGS="$(curl-config --static-libs)" NO_CURL= CURL_STATIC=YesPlease

    RUN make CURL_LDFLAGS="$(curl-config --static-libs)" NO_CURL= CURL_STATIC=YesPlease install

    SAVE ARTIFACT /dist/git-${GIT_VERSION}/bin/* AS LOCAL ./dist/
