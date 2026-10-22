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
    RUN curl -fsSL https://github.com/git/git/archive/v${GIT_VERSION}.tar.gz -o git_v${GIT_VERSION}.tar.gz
    RUN tar zxf git_v${GIT_VERSION}.tar.gz
    WORKDIR /git-${GIT_VERSION}

    RUN make configure && sed -i 's/qversion/-version/g' configure

    RUN pkg-config --modversion libcurl
    RUN curl-config --static-libs

    RUN ./configure prefix=/dist/git-${GIT_VERSION} \
        CFLAGS="-O2 -static" \
        LDFLAGS="-static"

    RUN make -j2 CURL_STATIC=YesPlease
    RUN make CURL_STATIC=YesPlease install

    SAVE ARTIFACT /dist/git-${GIT_VERSION}/bin/* AS LOCAL ./dist/
