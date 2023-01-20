FROM ubuntu:18.04

COPY greenplum-oss/ /greenplum-for-kubernetes/greenplum-oss/

WORKDIR /greenplum-for-kubernetes/greenplum-oss

RUN chmod +x ./install.bash \
 && set debconf to Noninteractive \
 && echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections \
 && ./install.bash

#RUN add-apt-repository ppa:ubuntu-toolchain-r/test -y \
# && apt-get update \
# && apt-get install -y wget bison ccache cmake curl flex git-core gcc g++ inetutils-ping krb5-kdc krb5-admin-server libapr1-dev libbz2-dev libcurl4-gnutls-dev libevent-dev libkrb5-dev libpam-dev libperl-dev libreadline-dev libssl-dev libxerces-c-dev libxml2-dev libyaml-dev libzstd-dev locales net-tools ninja-build openssh-client openssh-server openssl pkg-config python3-dev python3-pip python3-psutil python3-pygresql python3-yaml zlib1g-dev

RUN apt-get install -y wget \
 && wget -q -O /tmp/gpdb.deb https://github.com/greenplum-db/gpdb/releases/download/6.23.0/greenplum-db-6.23.0-ubuntu18.04-amd64.deb \
 && dpkg -i /tmp/gpdb.deb
