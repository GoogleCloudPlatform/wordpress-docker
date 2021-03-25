FROM marketplace.gcr.io/google/debian9:latest as exporter-builder

ENV GOPATH /usr/local
ENV GOROOT /usr/local/go
ENV GO_VERSION 1.13
ENV PATH=${GOPATH}/bin:${GOROOT}/bin:${PATH}
ENV EXPORTER_VERSION 0.8.0
ENV NOTICES_SHA256 0be64dd674a0e4124a4b0d97269e1810027999ad38e12741e4dd25d3c0bfc74e

# Installs packages
RUN set -eux \
    && apt-get update \
    && apt-get install -y \
        curl \
        govendor \
        tar \
        git \
        make \
        gcc

# Installs golang version ${GO_VERSION}
RUN set -eux \
    && curl -L -o /tmp/go${GO_VERSION}.linux-amd64.tar.gz "https://dl.google.com/go/go${GO_VERSION}.linux-amd64.tar.gz" \
    && tar -xzf /tmp/go${GO_VERSION}.linux-amd64.tar.gz -C ${GOPATH}

RUN set -eux \
    # Downloads source code
    && curl -L -o /tmp/apache_exporter.tar.gz "https://github.com/Lusitaniae/apache_exporter/archive/v${EXPORTER_VERSION}.tar.gz" \
    && mkdir -p /usr/local/src/apache_exporter \
    && tar -xzf /tmp/apache_exporter.tar.gz --strip-components=1 -C /usr/local/src/apache_exporter

RUN set -eux \
    && mkdir -p "${GOPATH}/src/github.com/Lusitaniae/apache_exporter" \
    && cd "${GOPATH}/src/github.com/Lusitaniae/apache_exporter" \
    && tar -xzf /tmp/apache_exporter.tar.gz --strip-components=1 -C . \
    # Builds binary
    && make \
    && mv apache_exporter /apache_exporter \
    # Extracts licences
    && govendor license +vendor > /NOTICES \
    # Verifies checksum. Changing the checksum means changing the licenses.
    && echo "${NOTICES_SHA256} /NOTICES" | sha256sum -c

FROM marketplace.gcr.io/google/debian9:latest

COPY --from=exporter-builder /apache_exporter /bin/apache_exporter
COPY --from=exporter-builder /NOTICES /usr/share/apache_exporter/NOTICES
COPY --from=exporter-builder /usr/local/src/apache_exporter /usr/local/src/apache_exporter

EXPOSE 9117
ENTRYPOINT ["/bin/apache_exporter"]
