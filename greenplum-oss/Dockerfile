FROM ubuntu:18.04

RUN add-apt-repository ppa:ubuntu-toolchain-r/test -y \
 && apt-get update \
 && ../README.Ubuntu.bash \
 && apt-get update \
 && apt-get install wget -y \
 && wget -q -O /tmp/gpdb.deb https://github.com/greenplum-db/gpdb/releases/download/6.23.0/greenplum-db-6.23.0-ubuntu18.04-amd64.deb \
 && dpkg -i /tmp/gpdb.deb
