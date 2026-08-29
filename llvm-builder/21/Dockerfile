FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
        wget \
        git \
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
        -DLLVM_INSTALL_UTILS=ON \
        -DCMAKE_INSTALL_PREFIX=/opt/llvm-21 && \
    cmake --build build -j $(nproc) -- all && \
    cmake --build build -- install/strip && \
    rm -rf /tmp/*
