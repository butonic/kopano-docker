ARG docker_repo=zokradonh
FROM ${docker_repo}/kopano_base

ARG VCS_REF
ARG ADDITIONAL_KOPANO_PACKAGES=""
ARG DOWNLOAD_COMMUNITY_PACKAGES=1
ARG KOPANO_REPOSITORY_FLAGS="trusted=yes"
ARG RELEASE_KEY_DOWNLOAD=0
ARG DEBIAN_FRONTEND=noninteractive
ARG KOPANO_CORE_REPOSITORY_URL="file:/kopano/repo/core"
ARG KOPANO_CORE_VERSION=newest

ENV \
    ADDITIONAL_KOPANO_PACKAGES=$ADDITIONAL_KOPANO_PACKAGES \
    DOWNLOAD_COMMUNITY_PACKAGES=$DOWNLOAD_COMMUNITY_PACKAGES \
    DOWNLOAD_DISTRIBUTION="Debian_9.0" \
    DOWNLOAD_CHANNEL="community" \
    DOWNLOAD_BRANCH="" \
    KOPANO_CORE_REPOSITORY_URL=$KOPANO_CORE_REPOSITORY_URL \
    KOPANO_CORE_VERSION=$KOPANO_CORE_VERSION \
    KOPANO_REPOSITORY_FLAGS=$KOPANO_REPOSITORY_FLAGS \
    RELEASE_KEY_DOWNLOAD=$RELEASE_KEY_DOWNLOAD \
    GRAPI_BACKEND="kopano" \
    KCCONF_GRAPI_LDAP_URI="" \
    KCCONF_GRAPI_LDAP_BASEDN="" \
    KCCONF_GRAPI_LDAP_BINDDN="" \
    KCCONF_GRAPI_LDAP_BINDPW_FILE=""

LABEL maintainer=az@zok.xyz \
    org.label-schema.name="Kopano core container" \
    org.label-schema.description="Container for running applications out of Kopano Groupware Core" \
    org.label-schema.url="https://kopano.io" \
    org.label-schema.vcs-ref=$VCS_REF \
    org.label-schema.vcs-url="https://github.com/zokradonh/kopano-docker" \
    org.label-schema.version=$KOPANO_CORE_VERSION \
    org.label-schema.schema-version="1.0"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# install Kopano Core and refresh ca-certificates
RUN \
    # community download and package as apt source repository
    . /kopano/helper/create-kopano-repo.sh && \
    if [ ${DOWNLOAD_COMMUNITY_PACKAGES} -eq 1 ]; then \
        dl_and_package_community "core" "$DOWNLOAD_DISTRIBUTION" "$DOWNLOAD_CHANNEL" "$DOWNLOAD_BRANCH"; \
    fi; \
    # apt key has already been installed in base
    echo "deb [${KOPANO_REPOSITORY_FLAGS}] ${KOPANO_CORE_REPOSITORY_URL} ./" > /etc/apt/sources.list.d/kopano.list; \
    # install
    apt-get update && \
    set -x && \
    apt-get install --no-install-recommends -y \
        kopano-server-packages \
        kopano-grapi kopano-kapid \
        ${ADDITIONAL_KOPANO_PACKAGES} \
        && \
    coreversion=$(dpkg-query --showformat='${Version}' --show kopano-server) && \
    if dpkg --compare-versions "$coreversion" "gt" "8.7.84"; then \
        apt-get install --no-install-recommends -y \
            python3-grapi.backend.ldap; \
    fi; \
    set +x && \
    rm -rf /var/cache/apt /var/lib/apt/lists && \
    touch /etc/kopano/admin.cfg && \
    cp /usr/share/doc/kopano/example-config/*.cfg /etc/kopano/ && \
    cp /usr/share/doc/kopano/example-config/*.cfg.gz /etc/kopano/ && \
    gzip -d -f /etc/kopano/*.gz

ENV LANG=en_US.UTF-8

ENV SERVICE_TO_START=server

COPY defaultconfigs/ start-service.sh healthcheck.sh /kopano/
COPY bin/ /usr/local/bin/
COPY goss/ /kopano/goss

WORKDIR /kopano/path

ENTRYPOINT ["/usr/bin/dumb-init", "--"]

CMD [ "/kopano/start-service.sh" ]

HEALTHCHECK --interval=1m --timeout=10s \
    CMD /kopano/healthcheck.sh
