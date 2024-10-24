# define the liboqs tag to be used
ARG LIBOQS_TAG=0.11.0

# define the openssl tag to be used
ARG OPENSSL_TAG=openssl-3.3.2

# define the oqsprovider tag to be used
ARG OQSPROVIDER_TAG=0.7.0

# Default location where all binaries wind up:
ARG OSSLGIODIR=/opt/ossl-gio

# 2-step build: First create binaries
FROM ubuntu:22.04 as intermediate
ARG LIBOQS_TAG
ARG OPENSSL_TAG
ARG OQSPROVIDER_TAG
ARG OSSLGIODIR
ENV DEBIAN_FRONTEND noninteractive

# everything to build liboqs, oqs-openssl, glib-networking:

RUN apt update && apt upgrade -y && apt install -y build-essential clang meson gnome-pkg-tools libglib2.0-dev libproxy-dev gsettings-desktop-schemas-dev ca-certificates epiphany-browser libtool make gcc ninja-build libssl-dev cmake libtool wget

WORKDIR /opt

RUN git clone --branch ${LIBOQS_TAG} https://github.com/open-quantum-safe/liboqs.git && \
    git clone --depth 1 --branch ${OPENSSL_TAG} https://github.com/openssl/openssl ossl-src && \
    git clone https://gitlab.gnome.org/GNOME/glib-networking.git && \
    git clone --depth 1 --branch ${OQSPROVIDER_TAG} https://github.com/open-quantum-safe/oqs-provider.git

# make sure the liboqs library is distributable:
RUN cd liboqs && mkdir build && cd build && cmake -GNinja -DOQS_DIST_BUILD=ON -DCMAKE_INSTALL_PREFIX=/opt/liboqs .. && ninja && ninja install && cd ../..

# build oqs-provider:
RUN cd oqs-provider && cmake -GNinja -DOQS_DIST_BUILD=ON -Dliboqs_DIR=/opt/liboqs -S . -B _build && cd _build && ninja && ninja install && cd ../..

RUN cd ossl-src && ./config no-shared -lm && make -j 2 && make install_sw && cd ..
RUN if [ -d /usr/lib/x86_64-linux-gnu ]; then ln -s /usr/lib/x86_64-linux-gnu /usr/lib/aarch64-linux-gnu; fi && \
    if [ -d /usr/lib/aarch64-linux-gnu ]; then ln -s /usr/lib/aarch64-linux-gnu /usr/lib/x86_64-linux-gnu; fi
RUN if [ -d /usr/local/lib64 ]; then ln -s /usr/local/lib64 /usr/local/lib; fi && \
    if [ -d /usr/local/lib ]; then ln -s /usr/local/lib /usr/local/lib64; fi

# build glib-networking such as to use (OQS-)OpenSSL and not GnuTLS:
RUN cd glib-networking && git checkout 2.72.2 && mkdir build && cd build && PKG_CONFIG_PATH=${OSSLGIODIR}/lib/pkgconfig CPATH=${OSSLGIODIR}/include LIBRARY_PATH=${OSSLGIODIR}/lib  meson --prefix=${OSSLGIODIR} -Dopenssl=enabled -Dgnutls=disabled .. && CPATH=${OSSLGIODIR}/include ninja && ninja install

# obtain and register test.openquantumsafe.org and letsencrypt certs
COPY certlink.sh /usr/local/ssl/certs/certlink.sh
RUN cd /usr/local/ssl/certs && wget https://letsencrypt.org/certs/isrgrootx1.pem && wget https://test.openquantumsafe.org/CA.crt -O oqsrootca.pem && ./certlink.sh isrgrootx1.pem && ./certlink.sh oqsrootca.pem

# 2nd build step: Only retain what's necessary:
FROM ubuntu:22.04
ARG OSSLGIODIR
ENV DEBIAN_FRONTEND noninteractive

RUN apt update && apt upgrade -y && apt install -y dbus-x11 epiphany-browser

COPY --from=intermediate ${OSSLGIODIR} ${OSSLGIODIR}
COPY --from=intermediate /usr/local /usr/local
# Move oqsprovider in place; for some unknown reason, GIO needs two locations for finding providers
RUN if [ -d /usr/lib/x86_64-linux-gnu ]; then rm -rf /usr/local/lib64/ossl-modules/ && ln -s /usr/lib/x86_64-linux-gnu/ossl-modules /usr/local/lib64/ossl-modules; fi && \
    if [ -d /usr/lib/aarch64-linux-gnu ]; then rm -rf /usr/local/lib/ossl-modules/ && ln -s /usr/lib/aarch64-linux-gnu/ossl-modules /usr/local/lib/ossl-modules; fi
COPY --from=intermediate /usr/lib/x86_64-linux-gnu/ossl-modules /usr/local/lib64/ossl-modules

# Run everything under a limited user account:
RUN groupadd -g 1001 oqs && useradd -u 1001 -d /home/oqs -g oqs oqs
COPY openssl-client.cnf /home/oqs/openssl-client.cnf
COPY startepiphany.sh /home/oqs/startepiphany.sh
RUN chown -R oqs.oqs /home/oqs

# certlink doesn't seem to work perfectly in this version of GIO, so use this "hardcore" method to install oqsCA for testing:
RUN cp /usr/local/ssl/certs/oqsrootca.pem /usr/share/ca-certificates/oqsrootca.crt  && echo "oqsrootca.crt" >> /etc/ca-certificates.conf && update-ca-certificates

USER oqs

ENV OPENSSL_CONF=/home/oqs/openssl-client.cnf
ENV LD_LIBRARY_PATH=${OSSLGIODIR}/lib
ENV GIO_MODULE_DIR=${OSSLGIODIR}/lib/x86_64-linux-gnu/gio/modules

ENTRYPOINT [ "/home/oqs/startepiphany.sh" ]
STOPSIGNAL SIGTERM
