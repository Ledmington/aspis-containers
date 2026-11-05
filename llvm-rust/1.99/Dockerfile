ARG BUILDER_IMAGE=llvm21-builder

FROM ${BUILDER_IMAGE} AS builder

FROM builder AS rust-builder

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        wget \
        pkg-config \
        libssl-dev && \
    cd /tmp && \
    wget -q https://static.rust-lang.org/dist/rustc-1.99.0-src.tar.gz && \
    echo "2035e4077b834a42ff8afd07f277ae3f06340098b86b1d2843aa234b4cfcae67 /tmp/rustc-1.99.0-src.tar.gz" | sha256sum -c - && \
    tar xzf /tmp/rustc-1.99.0-src.tar.gz && \
    mv /tmp/rustc-1.99.0-src /tmp/rust

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
        libedit2 \
        libncurses6 \
        liblzma5 \
        libxml2 \
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
