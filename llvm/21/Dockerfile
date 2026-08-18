FROM ubuntu:24.04 AS builder

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
    # Setup LLVM
    cd /tmp && \
    wget -q https://github.com/llvm/llvm-project/archive/refs/tags/llvmorg-21.1.8.tar.gz && \
    echo "7ba3f2a8d8fda88be18a31d011e8195d3b7f87f9fa92b20c94cba2d7f65b0e3f /tmp/llvmorg-21.1.8.tar.gz" >> /tmp/llvm_hash.txt && \
    sha256sum --check /tmp/llvm_hash.txt && \
    tar xzf /tmp/llvmorg-21.1.8.tar.gz && \
    cd /tmp/llvm-project-llvmorg-21.1.8 && \
    cmake -S llvm -B build \
        -G Ninja \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=/opt/llvm-21 \
        -DLLVM_ENABLE_PROJECTS='clang;openmp' \
        -DLLVM_ENABLE_RUNTIMES='libcxx;libcxxabi;libunwind' \
        -DLLVM_TARGETS_TO_BUILD='X86' \
        -DLLVM_INCLUDE_EXAMPLES=OFF \
        -DLLVM_INCLUDE_TESTS=OFF \
        -DLLVM_INCLUDE_BENCHMARKS=OFF \
        -DLLVM_USE_LINKER=mold && \
    cmake --build build -j $(nproc) -- all && \
    cmake --build build -- install && \
    # Cleanup
    cd ${HOME} && \
    rm -rf /tmp/* && \
    apt-get remove -y wget git mold && \
    apt-get autoremove -y && \
    apt-get clean -y

FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        python3 \
        cmake \
        ninja-build \
        ccache \
        mold && \
    apt-get autoremove -y && \
    apt-get clean -y && \
    rm -rf /var/lib/apt/lists/*

COPY --from=builder /opt/llvm-21 /opt/llvm-21
