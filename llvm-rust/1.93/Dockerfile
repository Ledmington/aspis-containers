FROM ubuntu:24.04 AS builder

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
        wget \
        gcc \
        g++ \
        cmake \
        ninja-build \
        mold \
        python3 \
        ca-certificates && \
    # Build LLVM
    cd /tmp && \
    wget -q https://github.com/llvm/llvm-project/archive/refs/tags/llvmorg-21.1.8.tar.gz && \
    echo "7ba3f2a8d8fda88be18a31d011e8195d3b7f87f9fa92b20c94cba2d7f65b0e3f /tmp/llvmorg-21.1.8.tar.gz" | sha256sum -c - && \
    tar xzf /tmp/llvmorg-21.1.8.tar.gz && \
    cd /tmp/llvm-project-llvmorg-21.1.8 && \
    cmake -S llvm -B build \
        -G Ninja \
        -DCMAKE_BUILD_TYPE=Release \
        -DLLVM_ENABLE_PROJECTS="clang" \
        -DLLVM_ENABLE_ASSERTIONS=OFF \
        -DLLDB_INCLUDE_TESTS=OFF \
        -DLIBCLC_TARGETS_TO_BUILD="RISCV;X86" \
        -DLLVM_USE_LINKER=mold \
        -DCMAKE_INSTALL_PREFIX=/opt/llvm-21 && \
    cmake --build build -j $(nproc) -- all && \
    cmake --build build -- install/strip

RUN apt update && \
    apt install -y --no-install-recommends curl && \
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
COPY --from=builder /opt/rust-llvm21 /opt/rust-llvm21

ENV PATH="/opt/llvm-21/bin:/opt/rust-llvm21/bin:${PATH}"
