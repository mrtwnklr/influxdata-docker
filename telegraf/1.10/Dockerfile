ARG IMAGE_NAME_PREFIX
FROM ${IMAGE_NAME_PREFIX}buildpack-deps:stretch-curl

RUN DEBIAN_FRONTEND=noninteractive apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends iputils-ping snmp procps lm-sensors && \
    rm -rf /var/lib/apt/lists/*

RUN set -ex && \
    for key in \
        05CE15085FC09D18E99EFB22684A14CF2582E0C5 ; \
    do \
        gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key" || \
        gpg --keyserver pgp.mit.edu --recv-keys "$key" || \
        gpg --keyserver keyserver.pgp.com --recv-keys "$key" ; \
    done

ARG TELEGRAF_VERSION
ENV TELEGRAF_VERSION ${TELEGRAF_VERSION:-1.10.4}
RUN ARCH= && dpkgArch="$(dpkg --print-architecture)" && \
    case "${dpkgArch##*-}" in \
      amd64) ARCH='amd64';; \
      arm64) ARCH='arm64';; \
      armhf) ARCH='armhf';; \
      armel) ARCH='armel';; \
      *)     echo "Unsupported architecture: ${dpkgArch}"; exit 1;; \
    esac && \
    TELEGRAF_RELEASE_TYPE=releases && \
    if [ "${TELEGRAF_VERSION}" = 'nightly' ]; then \
      TELEGRAF_RELEASE_TYPE=nightlies ; \
    else \
      TELEGRAF_VERSION=${TELEGRAF_VERSION}-1 ; \
    fi && \
    wget --no-verbose https://dl.influxdata.com/telegraf/${TELEGRAF_RELEASE_TYPE}/telegraf_${TELEGRAF_VERSION}_${ARCH}.deb.asc && \
    wget --no-verbose https://dl.influxdata.com/telegraf/${TELEGRAF_RELEASE_TYPE}/telegraf_${TELEGRAF_VERSION}_${ARCH}.deb && \
    if [ "${TELEGRAF_VERSION}" != 'nightly' ]; then \
      gpg --batch --verify telegraf_${TELEGRAF_VERSION}_${ARCH}.deb.asc telegraf_${TELEGRAF_VERSION}_${ARCH}.deb ; \
    fi && \
    dpkg -i telegraf_${TELEGRAF_VERSION}_${ARCH}.deb && \
    rm -f telegraf_${TELEGRAF_VERSION}_${ARCH}.deb*

EXPOSE 8125/udp 8092/udp 8094

COPY entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
CMD ["telegraf"]
