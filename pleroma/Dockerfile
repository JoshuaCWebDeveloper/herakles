FROM elixir:1.13-alpine as build

RUN apk add git gcc g++ musl-dev make cmake file-dev

COPY ./pleroma/ .

ENV MIX_ENV=prod

RUN echo "import Mix.Config" > config/prod.secret.exs &&\
	mix local.hex --force &&\
	mix local.rebar --force &&\
	HEX_HTTP_CONCURRENCY=1 HEX_HTTP_TIMEOUT=120 mix deps.get --only prod &&\
	mkdir release &&\
	mix release --path release

FROM alpine:3.16

LABEL maintainer="ops@pleroma.social" \
	org.opencontainers.image.title="pleroma" \
	org.opencontainers.image.description="Pleroma for Docker" \
	org.opencontainers.image.authors="ops@pleroma.social" \
	org.opencontainers.image.vendor="pleroma.social" \
	org.opencontainers.image.documentation="https://git.pleroma.social/pleroma/pleroma" \
	org.opencontainers.image.licenses="AGPL-3.0" \
	org.opencontainers.image.url="https://pleroma.social" 

ARG HOME=/opt/pleroma
ARG DATA=/var/lib/pleroma

RUN echo "http://nl.alpinelinux.org/alpine/latest-stable/community" >> /etc/apk/repositories &&\
	apk update &&\
	apk add exiftool ffmpeg imagemagick libmagic ncurses postgresql-client bash inotify-tools jq curl &&\
	mkdir -p ${HOME} &&\
	# adduser --system --shell /bin/false --home ${HOME} pleroma &&\
	mkdir -p ${DATA}/uploads &&\
	mkdir -p ${DATA}/static &&\
	# chown -R pleroma ${DATA} &&\
	mkdir -p /etc/pleroma
# chown -R pleroma /etc/pleroma &&\
# chown -R pleroma /etc

# USER pleroma

COPY --from=build /release ${HOME}

COPY ./config.exs /etc/pleroma/config.exs
COPY ./docker-entrypoint.sh ${HOME}
COPY ./watch_hosts.sh ${HOME}
COPY ./soapbox.zip ${HOME}

RUN mkdir -p /var/lib/pleroma/static/frontends/soapbox-fe 
RUN busybox unzip ${HOME}/soapbox.zip -o -d /var/lib/pleroma/static/frontends/soapbox-fe

RUN syslogd

EXPOSE 4000

ENTRYPOINT ["/opt/pleroma/docker-entrypoint.sh"]
