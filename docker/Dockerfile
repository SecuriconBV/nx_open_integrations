
FROM ubuntu:18.04
MAINTAINER "Dmitry Kargin" <dkargin@networkoptix.com>
# Copyright 2018-present Network Optix, Inc. Licensed under MPL 2.0: www.mozilla.org/MPL/2.0/
# This Dockerfile was based on the knowledge from the following sources:
# - some words about systemd problems inside docker 
# https://developers.redhat.com/blog/2014/05/05/running-systemd-within-docker-container/ 
# - some parts from https://github.com/defn/docker-systemd

# This string is needed for systemd to know we are working inside container environment.
ENV container docker

# This overrides VMS EULA check. 
ENV DEBIAN_FRONTEND noninteractive

# Disabling it since build.sh does all the necessary checks.
# RUN ["/bin/bash", "-c", ": ${mediaserver_deb:?Missing source deb file. 
# You should set --build-arg mediaserver_deb=path_to_mediaserver.deb!}"]

COPY manage.sh /setup/
RUN chmod a+x /setup/manage.sh

# We need:
# - bin-utils for patching cloud host
# - lsb-release is used as a part of install scripts inside the deb package.
RUN apt-get update && apt-get install -y dbus systemd lsb-release binutils \ 
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN find /etc/systemd -name '*.timer' | xargs rm -v

RUN systemctl set-default multi-user.target

STOPSIGNAL SIGRTMIN+3

# Using systemd as an entry point. It will launch mediaserver according to systemd scripts.
ENTRYPOINT ["/sbin/init", "--log-target=journal"]

# This is the deb file used to install mediaserver.
ARG mediaserver_deb

COPY "${mediaserver_deb}" /setup/

# Installing the mediaserver. It will download and install requirements as well.
RUN apt-get update -qq \
    && apt install -y -qq --no-install-recommends "/setup/${mediaserver_deb}" \
    && mkdir -p /opt/networkoptix/mediaserver/var

# Override for cloud host.
# We keep this argumet last to prevent docker rebuilding all previous steps.
ARG cloud_host
RUN /setup/manage.sh --cloud "$cloud_host" --norestart