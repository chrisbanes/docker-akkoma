FROM hexpm/elixir:1.13.4-erlang-24.3.4.5-alpine-3.15.6

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

ARG DATA=/var/lib/akkoma
RUN mkdir -p ${DATA}/uploads \
    && mkdir -p ${DATA}/static \
    && chown -R $UNAME ${DATA}

USER $UNAME
WORKDIR /akkoma

RUN git clone -b stable https://akkoma.dev/AkkomaGang/akkoma.git /akkoma \
    && git checkout ${AKKOMA_VER} 

RUN echo "import Mix.Config" > config/prod.secret.exs \
    && mix local.hex --force \
    && mix local.rebar --force \
    && mix deps.get --only prod \
    && mkdir release \
    && mix release --path /akkoma

COPY ./config.exs /akkoma/config/prod.exs

EXPOSE 4000

CMD ["/akkoma/docker-entrypoint.sh"]
