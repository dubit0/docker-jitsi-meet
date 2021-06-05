###############################
FROM golang:1.13-alpine AS build

RUN apk add --no-cache make git

RUN mkdir -p /go/src/github.com/subchen/frep
RUN git clone https://github.com/subchen/frep.git /go/src/github.com/subchen/frep
WORKDIR /go/src/github.com/subchen/frep

RUN GO111MODULE=on CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build -a -ldflags "-s -w" -installsuffix cgo -o _releases/frep-1.3.8

###############################

FROM debian:buster-slim

ARG JITSI_RELEASE=stable
ARG FREP_VERSION=1.3.11

ENV S6_BEHAVIOUR_IF_STAGE2_FAILS=2

COPY rootfs /
COPY --from=build /go/src/github.com/subchen/frep/_releases/frep-* /usr//bin/frep


RUN \
	apt-dpkg-wrap apt-get update && \
	apt-dpkg-wrap apt-get install -y apt-transport-https apt-utils ca-certificates gnupg wget && \
	wget -qO - https://github.com/just-containers/s6-overlay/releases/download/v2.2.0.3/s6-overlay-aarch64.tar.gz | tar xfz - -C / && \
	wget -qO - https://download.jitsi.org/jitsi-key.gpg.key -O /etc/apt/trusted.gpg.d/jitsi.asc && \
	echo "deb https://download.jitsi.org $JITSI_RELEASE/" > /etc/apt/sources.list.d/jitsi.list && \
	echo "deb http://ftp.debian.org/debian buster-backports main" > /etc/apt/sources.list.d/backports.list && \
	apt-dpkg-wrap apt-get update && \
	apt-dpkg-wrap apt-get dist-upgrade -y && \
	apt-cleanup && \
	chmod +x /usr/bin/frep

RUN \
	[ "$JITSI_RELEASE" = "unstable" ] && \
	apt-dpkg-wrap apt-get update && \
	apt-dpkg-wrap apt-get install -y jq procps curl vim iputils-ping net-tools && \
	apt-cleanup || \
	true

ENTRYPOINT [ "/init" ]
