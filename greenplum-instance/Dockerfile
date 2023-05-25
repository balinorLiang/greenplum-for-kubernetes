# syntax = docker/dockerfile:experimental
ARG TAG_PREFIX=""
ARG GREENPLUM_VERSION=unknown
FROM golang:1.19 as build-in-docker

COPY greenplum-instance/ /greenplum-for-kubernetes/greenplum-instance/
COPY greenplum-operator/ /greenplum-for-kubernetes/greenplum-operator/
COPY pkg/ /greenplum-for-kubernetes/pkg/
COPY go.mod go.sum /greenplum-for-kubernetes/

WORKDIR /greenplum-for-kubernetes/greenplum-instance

RUN --mount=type=cache,target=/root/.cache/go-build \
    --mount=type=cache,target=/go/pkg/mod \
    mkdir buildcmd && \
    go build -o ./buildcmd/ \
    ./cmd/sshKeyScan \
    ./cmd/startGreenplumContainer  \
    ./cmd/initializeCluster \
    ./cmd/startPXF \
    ./cmd/runGpexpand \
    ./cmd/waitForKnownHosts

# build greenplum-instance image from here
# FROM gcr.io/plaidcloud-build/greenplum-pxf:latest
FROM rockylinux:8.7


ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
ENV GPHOME=/usr/local/greenplum-db
# ENV PXF_HOME=/usr/local/pxf
# ENV PXF_BASE=${HOME}/pxf-base
ENV GOPATH=/usr/local/go
ENV PATH=${PATH}:/usr/local/go/bin:/usr/local/greenplum-db/bin/

# RUN alternatives --set python3 /usr/bin/python3.9
RUN yum -y install passwd
RUN yum -y localinstall https://github.com/greenplum-db/gpdb/releases/download/7.0.0-beta.3/open-source-greenplum-db-7.0.0-beta.3-el8-x86_64.rpm
RUN curl https://raw.githubusercontent.com/greenplum-db/gpdb/master/concourse/scripts/setup_gpadmin_user.bash > ./setup_gpadmin_user.bash
RUN sed -i '/gpdb_src/d' ./setup_gpadmin_user.bash
RUN chmod +x ./setup_gpadmin_user.bash \
&& ./setup_gpadmin_user.bash

# remove gppkg because it's misleading to users in the kubernetes environment
RUN rm -f /usr/local/greenplum-db/bin/gppkg

# RUN mv /etc/sudoers /etc/sudoers.bak
# RUN apt-get update
# RUN apt-get -y install sudo iproute2 rsync
# RUN mv /etc/sudoers.bak /etc/sudoers

ARG TOOLS_DIR=/tools

COPY --from=build-in-docker \
    /greenplum-for-kubernetes/greenplum-instance/buildcmd/sshKeyScan \
    /greenplum-for-kubernetes/greenplum-instance/buildcmd/initializeCluster \
    /greenplum-for-kubernetes/greenplum-instance/buildcmd/startGreenplumContainer \
    /greenplum-for-kubernetes/greenplum-instance/buildcmd/startPXF \
    /greenplum-for-kubernetes/greenplum-instance/buildcmd/runGpexpand \
    /greenplum-for-kubernetes/greenplum-instance/buildcmd/waitForKnownHosts \
    ${TOOLS_DIR}/

COPY \
    greenplum-instance/scripts/gpexpand_job.sh \
    ${TOOLS_DIR}/

COPY greenplum-instance/scripts/gpadmin-limits.conf /etc/security/limits.d/

RUN \
    rm -f /etc/ssh/ssh_host_*_key* && \
    # comment out all other host key files
    sed -ri 's@^HostKey /etc/ssh/ssh_host_dsa_key$@#&@' /etc/ssh/sshd_config && \
    sed -ri 's@^HostKey /etc/ssh/ssh_host_ecdsa_key$@#&@' /etc/ssh/sshd_config && \
    sed -ri 's@^HostKey /etc/ssh/ssh_host_ed25519_key$@#&@' /etc/ssh/sshd_config && \
    # uncomment the rsa host key file
    sed -ri '\@^#HostKey /etc/ssh/ssh_host_rsa_key$@s@^#@@g' /etc/ssh/sshd_config && \
    # See https://gist.github.com/gasi/5691565
    sed -ri 's/UsePAM yes/UsePAM no/g' /etc/ssh/sshd_config && \
    # Disable password authentication so builds never hang given bad keys
    sed -ri 's/#PasswordAuthentication yes/PasswordAuthentication no/g' /etc/ssh/sshd_config && \
    mkdir -p /var/run/sshd && \
    chmod 0755 /var/run/sshd && \
    echo "gpadmin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

ENV PXF_CONF=/etc/pxf
RUN mkdir -p ${PXF_CONF} && chown -R gpadmin:gpadmin ${PXF_CONF}

# RUN apt install curl ca-certificates gnupg
# RUN curl https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor | tee /etc/apt/trusted.gpg.d/apt.postgresql.org.gpg >/dev/null
# RUN echo "deb http://apt.postgresql.org/pub/repos/apt bionic-pgdg main" > /etc/apt/sources.list.d/pgdg.list
# RUN apt update
# RUN apt-get install -y postgresql-12 postgresql-server-dev-12 postgresql-contrib-12

# RUN apt-get install -y software-properties-common
# RUN add-apt-repository -y ppa:ubuntugis/ppa
# RUN apt update
# RUN apt-get install -y libgeos-3.7.1 libgeos++-dev  libgeos-c1v5  libgeos-dev libgeos-doc \
#     # gdal-bin gdal-data libgdal-dev libgdal-doc libgdal20  python3-gdal  \
#     proj-bin \
#     libxml2 libxml2-dev \
#     libjson-c-dev \
#     protobuf-c-compiler protobuf-compiler libprotobuf-c1 libprotobuf-c-dev \
#     autoconf \
#     expat \
#     libcunit1 libcunit1-dev
#     # postgresql-12-dev
#     # libjson-c libjson-c-dev

# RUN apt install curl ca-certificates gnupg
# RUN curl https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor | tee /etc/apt/trusted.gpg.d/apt.postgresql.org.gpg >/dev/null
# RUN echo "deb http://apt.postgresql.org/pub/repos/apt bionic-pgdg main" > /etc/apt/sources.list.d/pgdg.list
# # RUN wget --quiet -O - http://apt.postgresql.org/pub/repos/apt/ACCC4CF8 | apt-key add -
# RUN apt update
# RUN apt install -y postgresql-12-postgis-2.5
# # RUN apt install -y postgresql-12-postgis-scripts
# RUN apt install -y postgis

# RUN wget https://download.osgeo.org/postgis/source/postgis-3.3.2.tar.gz
# RUN tar -xzf postgis-3.3.2.tar.gz
# WORKDIR /workspace/postgis-3.3.2
# RUN ./configure
# RUN make
# RUN make install

# RUN wget https://download.osgeo.org/gdal/1.11.1/gdal-1.11.1.tar.gz
# RUN tar zxf gdal-1.11.1.tar.gz
# WORKDIR /workspace/gdal-1.11.1
# RUN ./configure --prefix=$GPHOME
# RUN make
# RUN make install

# WORKDIR /workspace

# RUN wget https://github.com/greenplum-db/geospatial/archive/refs/heads/main.zip
# RUN unzip main.zip
# WORKDIR /workspace/geospatial-main/postgis/build/postgis-2.5.4
# RUN ./autogen.sh
# RUN ./configure --with-pgconfig=$GPHOME/bin/pg_config --with-raster --without-topology --prefix=$GPHOME
# RUN make USE_PGXS=1 clean all install

# WORKDIR /workspace

# RUN cp -r /usr/share/postgresql/12/extension/postgis* /usr/local/greenplum-db/share/postgresql/extension/
# RUN cp -r /usr/share/postgresql/12/contrib/postgis* /usr/local/greenplum-db/share/postgresql/contrib/
# RUN cp -r /usr/lib/postgresql/12/lib/*postgis* /usr/local/greenplum-db/lib/postgresql
# ENV libdir=/usr/local/greenplum-db/share/postgresql/contrib/

# Install tini
RUN yum -y localinstall https://github.com/krallin/tini/releases/download/v0.19.0/tini_0.19.0.rpm
# RUN wget https://github.com/krallin/tini/releases/download/v0.19.0/tini_0.19.0-amd64.deb && \
#     dpkg -i tini_0.19.0-amd64.deb && \
#     rm tini_0.19.0-amd64.deb
# ENTRYPOINT ["/usr/bin/tini", "-vvv", "--"]
ENTRYPOINT ["/usr/bin/tini", "--"]
# ENTRYPOINT ["/bin/bash", "-c", "echo $1; while true; do sleep 30; done; "]

USER gpadmin

# USER env is referenced by `gpconfig`, `gpcheckcat`, `gpssh`, `gpscp`, `gpinitsegment`,
# and `gpload` utilities through `os.environ.get('USER')`
ENV USER=gpadmin
ENV PGPORT=5432
WORKDIR /home/gpadmin

# TODO update the deploy.sh etc. to only depends on /tools instead of /home/gpadmin/tools
RUN ln -s ${TOOLS_DIR} /home/gpadmin/tools

# Initialize pxf directories; needs GPHOME set
# RUN /bin/bash -c 'source /usr/local/greenplum-db/greenplum_path.sh && /usr/local/pxf/bin/pxf init'

# USER root

EXPOSE 22

ARG VERSION=dev
ARG DATE=unknown
ARG LICENSE="https://www.apache.org/licenses/LICENSE-2.0"

LABEL name="Greenplum for Kubernetes" \
    vendor="PlaidCloud" \
    version="${VERSION}" \
    build_date="${DATE}" \
    license="${LICENSE}"
