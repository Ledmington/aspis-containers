ARG BUILDER_IMAGE=llvm21-builder

FROM ${BUILDER_IMAGE} AS builder

FROM builder AS rust-builder

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        wget && \
    cd /tmp && \
    wget -q https://static.rust-lang.org/dist/rustc-1.93.0-src.tar.gz && \
    echo "69112bd83c321943ffc390b7db2f59ef3febb95dac700f67c2156fbf6b50a705 /tmp/rustc-1.93.0-src.tar.gz" | sha256sum -c - && \
    tar xzf /tmp/rustc-1.93.0-src.tar.gz && \
    mv /tmp/rustc-1.93.0-src /tmp/rust

COPY bootstrap.toml /tmp/rust/bootstrap.toml

RUN cd /tmp/rust && ./x install --stage 1

FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
        git \
        build-essential \
        libatomic1 \
        ca-certificates \
        python3 \
        python3-pip \
        python3-venv \
        cmake \
        ninja-build && \
    apt-get autoremove -y && \
    apt-get clean -y && \
    rm -rf /var/lib/apt/lists/*

COPY --from=builder /opt/llvm-21 /opt/llvm-21
COPY --from=rust-builder /opt/rust-llvm21 /opt/rust-llvm21

ENV PATH="/opt/llvm-21/bin:/opt/rust-llvm21/bin:${PATH}"
