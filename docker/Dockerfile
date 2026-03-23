FROM golang:1.23 AS builder
LABEL maintainer="multiversx"

ARG GITHUB_ORG=multiversx

RUN apt-get update && apt-get install -y

WORKDIR /config
COPY . .

WORKDIR /go
RUN BINARY_VER=$(cat /config/binaryVersion | tr -d '[:space:]') && \
    echo "Cloning ${GITHUB_ORG}/mx-chain-go @ ${BINARY_VER}" && \
    git clone --depth=1 --branch="${BINARY_VER}" https://github.com/${GITHUB_ORG}/mx-chain-go.git
WORKDIR /go/mx-chain-go

WORKDIR /go/mx-chain-go/cmd/node
RUN go build -v -ldflags="-X main.appVersion=$(git --git-dir /config/.git describe --tags --long --dirty 2>/dev/null || echo 'xdp-dev')"

RUN mkdir -p /lib_amd64 /lib_arm64
RUN cp /go/pkg/mod/github.com/multiversx/$(cat /go/mx-chain-go/go.mod | grep mx-chain-vm-v | sort -n | tail -n -1 | awk -F '/' '{print$3}' | sed 's/ /@/g')/wasmer/libwasmer_linux_amd64.so /lib_amd64/ 2>/dev/null || true
RUN cp /go/pkg/mod/github.com/multiversx/$(cat /go/mx-chain-go/go.mod | grep mx-chain-vm-go | sort -n | tail -n -1 | awk -F '/' '{print$3}' | sed 's/ /@/g')/wasmer2/libvmexeccapi.so /lib_amd64/ 2>/dev/null || true
RUN cp /go/pkg/mod/github.com/multiversx/$(cat /go/mx-chain-go/go.mod | grep mx-chain-vm-v | sort -n | tail -n -1 | awk -F '/' '{print$3}' | sed 's/ /@/g')/wasmer/libwasmer_linux_arm64_shim.so /lib_arm64/ 2>/dev/null || true
RUN cp /go/pkg/mod/github.com/multiversx/$(cat /go/mx-chain-go/go.mod | grep mx-chain-vm-go | sort -n | tail -n -1 | awk -F '/' '{print$3}' | sed 's/ /@/g')/wasmer2/libvmexeccapi_arm.so /lib_arm64/ 2>/dev/null || true

WORKDIR /config
RUN cp -r *.toml *.json gasSchedules genesisContracts /go/mx-chain-go/cmd/node/config/ 2>/dev/null || true

WORKDIR /go/mx-chain-go/cmd/node

FROM ubuntu:22.04
ARG TARGETARCH
RUN apt-get update -y && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
COPY --from=builder "/go/mx-chain-go/cmd/node" "/go/mx-chain-go/cmd/node/"
RUN mkdir -p /lib
COPY --from=builder /lib_amd64/ /lib/
WORKDIR /go/mx-chain-go/cmd/node/
EXPOSE 8080
ENTRYPOINT ["/go/mx-chain-go/cmd/node/node", "--rest-api-interface=:8080"]
