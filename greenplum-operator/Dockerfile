# syntax = docker/dockerfile:experimental
FROM golang:1.19 as build-in-docker


COPY greenplum-operator/ /greenplum-for-kubernetes/greenplum-operator/
COPY pkg/ /greenplum-for-kubernetes/pkg/
COPY tools/ /greenplum-for-kubernetes/tools/
COPY go.mod go.sum /greenplum-for-kubernetes/


WORKDIR /greenplum-for-kubernetes/greenplum-operator
RUN go install sigs.k8s.io/controller-tools/cmd/controller-gen@v0.10.0
# # RUN go get -modfile /greenplum-for-kubernetes/tools/go.mod sigs.k8s.io/controller-tools/cmd/controller-gen@v0.10.0
# RUN ls /usr/local/go/bin
# RUN controller-gen object:headerFile=./hack/boilerplate.go.txt paths=./api/...
RUN controller-gen crd:crdVersions=v1 rbac:roleName=manager-role webhook paths="./..." output:crd:artifacts:config=config/crd/bases \
    && kubectl kustomize config/crd > operator/templates/greenplum-operator-crds.yaml \

RUN --mount=type=cache,target=/root/.cache/go-build \
    --mount=type=cache,target=/go/pkg/mod \
    cd /greenplum-for-kubernetes/greenplum-operator/cmd/greenplumOperator \
    && go build
    # cd /greenplum-for-kubernetes/greenplum-operator/cmd/greenplumOperator && /usr/local/go/bin/controller-gen && go build


# build greenplum-operator image from here
FROM cloudfoundry/run:base

COPY --from=build-in-docker \
    /greenplum-for-kubernetes/greenplum-operator/cmd/greenplumOperator/greenplumOperator \
    /usr/local/bin/greenplum-operator

RUN set -x && \
    apt-get update && \
    apt-get upgrade -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ARG VERSION=dev
ARG DATE="25 Jan 2023"
ARG LICENSE="https://www.apache.org/licenses/LICENSE-2.0"
ARG GREENPLUM_VERSION=7.00.0

ENV GREENPLUM_VERSION=${GREENPLUM_VERSION}

LABEL name="Greenplum Operator" \
    vendor="PlaidCloud" \
    version="${VERSION}" \
    build_date="${DATE}" \
    license="${LICENSE}" \
    greenplum_version="${GREENPLUM_VERSION}"
