FROM bitnami/minideb-extras-base:stretch-r257
LABEL maintainer "Bitnami <containers@bitnami.com>"

ENV BITNAMI_PKG_CHMOD="-R g+rwX" \
    HOME="/" \
    OS_ARCH="amd64" \
    OS_FLAVOUR="debian-9" \
    OS_NAME="linux"

# Install required system packages and dependencies
RUN install_packages libbsd0 libc6 libedit2 libgcc1 libicu57 liblzma5 libncurses5 libnss-wrapper libssl1.1 libstdc++6 libtinfo5 libxml2 libxslt1.1 locales zlib1g
RUN . ./libcomponent.sh && component_unpack "postgresql" "10.8.0-2" --checksum 926e5842cbe9bee8e600bbc0b553e7dbcd5da44d3bdfff279a13132b817d0764
RUN echo 'en_GB.UTF-8 UTF-8' >> /etc/locale.gen && locale-gen
RUN echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen && locale-gen

COPY rootfs /
RUN /postunpack.sh
ENV BITNAMI_APP_NAME="postgresql" \
    BITNAMI_IMAGE_VERSION="10.8.0-debian-9-r23" \
    LANG="en_US.UTF-8" \
    LANGUAGE="en_US:en" \
    NAMI_PREFIX="/.nami" \
    NSS_WRAPPER_LIB="/usr/lib/libnss_wrapper.so" \
    PATH="/opt/bitnami/postgresql/bin:$PATH"

VOLUME [ "/bitnami/postgresql", "/docker-entrypoint-initdb.d" ]


ENV POSTGIS_VERSION 2.5.2
ENV POSTGIS_SHA256 b6cb286c5016029d984f8c440947bf9178da72e1f6f840ed639270e1c451db5e

## Install Postgis
RUN install_packages wget gcc make build-essential libxml2-dev libproj-dev libgdal-dev autoconf automake libtool libprotobuf-c-dev protobuf-c-compiler protobuf-compiler \
    && cd /tmp \
    && wget -P ./ https://download.osgeo.org/geos/geos-3.7.1.tar.bz2 \
    && export C_INCLUDE_PATH=/opt/bitnami/postgresql/include/:/opt/bitnami/common/include/ \
    && export LIBRARY_PATH=/opt/bitnami/postgresql/lib/:/opt/bitnami/common/lib/ \
    && export LD_LIBRARY_PATH=/opt/bitnami/postgresql/lib/:/opt/bitnami/common/lib/ \
    && tar xfj geos-3.7.1.tar.bz2 \
    && cd geos-3.7.1 \
    && ./configure --prefix=/opt/bitnami/postgresql \
    && make -j $(cat /proc/cpuinfo | grep processor | wc -l) \
    && make install \
    && cd /tmp \
    && rm -rf geos-3.7.1 \
    && wget -O postgis.tar.gz "https://download.osgeo.org/postgis/source/postgis-$POSTGIS_VERSION.tar.gz" \
    && echo "$POSTGIS_SHA256 *postgis.tar.gz" | sha256sum -c - \
    && mkdir -p /usr/src/postgis \
    && tar \
        --extract \
        --file postgis.tar.gz \
        --directory /usr/src/postgis \
        --strip-components 1 \
    && rm postgis.tar.gz \
    && cd /usr/src/postgis \
    && ./autogen.sh \
    && ./configure --with-pgconfig=/opt/bitnami/postgresql/bin/pg_config --prefix=/opt/bitnami/postgresql \
    && make -j $(cat /proc/cpuinfo | grep processor | wc -l) \
    && make install \
    && rm -rf /usr/src/postgis

EXPOSE 5432

USER 1001
ENTRYPOINT [ "/entrypoint.sh" ]
CMD [ "/run.sh" ]
