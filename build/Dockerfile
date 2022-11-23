FROM elixir:1.13.4-alpine

ARG AKKOMA_VER=stable
ARG UNAME=akkoma
ARG UID=911
ARG GID=911
ENV MIX_ENV=prod

LABEL org.opencontainers.image.title="akkoma" \
    org.opencontainers.image.description="Akkoma for Docker" \
    org.opencontainers.image.vendor="akkoma.dev" \
    org.opencontainers.image.documentation="https://docs.akkoma.dev/stable/" \
    org.opencontainers.image.licenses="AGPL-3.0" \
    org.opencontainers.image.url="https://akkoma.dev" \
    org.opencontainers.image.revision=$AKKOMA_VER \
    org.opencontainers.image.created=$BUILD_DATE

ARG HOME=/akkoma

RUN addgroup -g $GID $UNAME \
    && adduser -h $HOME -s /bin/false -D -G $UNAME -u $UID $UNAME

RUN apk add git gcc g++ musl-dev make cmake file-dev exiftool ffmpeg imagemagick libmagic ncurses postgresql-client

RUN mkdir -p /uploads && chown -R $UNAME /uploads \
    && mkdir -p /static && chown -R $UNAME /static \
    && mkdir -p /config && chown -R $UNAME /config

USER $UNAME
WORKDIR /akkoma

RUN git clone -b stable https://akkoma.dev/AkkomaGang/akkoma.git /akkoma \
    && git checkout ${AKKOMA_VER} 

RUN echo "import Mix.Config" > config/prod.secret.exs \
    && mix local.hex --force \
    && mix local.rebar --force \
    && mix deps.get --only prod \
    && mix compile
# Need to investigate using mix release to avoid needing to recompile
# on startup. It breaks Akkoma currently from a quick test.

COPY ./config.exs /akkoma/config/prod.exs

EXPOSE 4000

CMD ["/akkoma/docker-entrypoint.sh"]
