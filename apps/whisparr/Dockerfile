FROM ghcr.io/k8s-at-home/ubuntu-jammy:rolling@sha256:fdbafba13f7ed147d6f95ab5269ddaa7f17395e226f1bbc25381de4c6671442f

ARG VERSION
USER root

ENV \
  WHISPARR__INSTANCE_NAME="Whisparr" \
  WHISPARR__BRANCH="develop" \
  WHISPARR__PORT="6969"

# hadolint ignore=DL3008,DL3015,SC2086
RUN \
  apt-get -qq update \
  && apt-get -qq install -y gnupg \
	&& \
  apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF \
  && echo "deb https://download.mono-project.com/repo/ubuntu stable-focal main" | tee /etc/apt/sources.list.d/mono-official.list \
  && apt-get -qq update \
  && \
  apt-get -qq install -y \
    ca-certificates \
    mono-complete \
    ca-certificates-mono \
    libmediainfo0v5 \
    xmlstarlet 

RUN curl -fsSL -o /tmp/whisparr.tar.gz \
    "http://whisparr.servarr.com/v1/update/nightly/updatefile?os=linux&runtime=netcore&arch=x64"
RUN tar xzf /tmp/whisparr.tar.gz -C /app --strip-components 1

RUN rm -rf \
    /app/whisparr.Update* \
  && \
  printf "UpdateMethod=docker\nBranch=%s\nPackageVersion=%s\nPackageAuthor=[Team k8s-at-home](https://github.com/k8s-at-home)" "${WHISPARR__BRANCH}" "${VERSION}" > /app/package_info \
  && \
  gpgconf --kill all \
  && apt-get remove -y gnupg \
  && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
  && apt-get autoremove -y \
  && apt-get clean \
  && \
  rm -rf \
    /tmp/* \
    /var/lib/apt/lists/* \
    /var/cache/apt/* \
    /var/tmp/ \
  && chown -R kah:kah /app \
  && chmod -R u=rwX,go=rX /app \
  && printf "umask %d" "${UMASK}" >> /etc/bash.bashrc

USER kah

EXPOSE ${WHISPARR__PORT}/tcp

COPY ./apps/whisparr/config.xml.tmpl /app/config.xml.tmpl
COPY ./apps/whisparr/entrypoint.sh /entrypoint.sh
CMD ["/entrypoint.sh"]

LABEL \
  org.opencontainers.image.title="${WHISPARR__INSTANCE_NAME}" \
  org.opencontainers.image.source="https://github.com/Whisparr/Whisparr" \
  org.opencontainers.image.version="${VERSION}"
