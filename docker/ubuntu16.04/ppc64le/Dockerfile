FROM debian:buster-slim as qemu

RUN apt-get update && apt-get install --no-install-recommends -y qemu-user-static

FROM nvidia/cuda-ppc64le:8.0-devel-ubuntu16.04 as build

COPY --from=qemu /usr/bin/qemu-ppc64le-static /usr/bin/qemu-ppc64le-static

RUN apt-get update && apt-get install -y --no-install-recommends \
        g++ \
        ca-certificates \
        wget \
        cuda-cudart-dev-8-0 \
        cuda-misc-headers-8-0 \
        cuda-nvml-dev-8-0 && \
    rm -rf /var/lib/apt/lists/*

ENV GOLANG_VERSION 1.10.2
RUN wget -nv -O - https://storage.googleapis.com/golang/go${GOLANG_VERSION}.linux-ppc64le.tar.gz \
    | tar -C /usr/local -xz
ENV GOPATH /go
ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH

ENV CGO_CFLAGS "-I /usr/local/cuda-8.0/include"
ENV CGO_LDFLAGS "-L /usr/local/cuda-8.0/lib64"
ENV PATH=$PATH:/usr/local/nvidia/bin:/usr/local/cuda/bin

WORKDIR /go/src/nvidia-device-plugin
COPY . .

RUN export CGO_LDFLAGS_ALLOW='-Wl,--unresolved-symbols=ignore-in-object-files' && \
    go install -ldflags="-s -w" -v nvidia-device-plugin


FROM ppc64le/debian:stretch-slim

ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=utility

COPY --from=build /go/bin/nvidia-device-plugin /usr/bin/nvidia-device-plugin

CMD ["nvidia-device-plugin"]
