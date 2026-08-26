FROM ubuntu:24.04 AS builder

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
        wget \
        git \
        clang \
        libc++-dev \
        libc++abi-dev \
        cmake \
        ninja-build \
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
        -DCMAKE_C_COMPILER=clang \
        -DCMAKE_CXX_COMPILER=clang++ \
        -DCMAKE_CXX_FLAGS="-stdlib=libc++" \
        -DCMAKE_EXE_LINKER_FLAGS="-stdlib=libc++" \
        -DCMAKE_SHARED_LINKER_FLAGS="-stdlib=libc++" \
        -DCMAKE_BUILD_TYPE=Release \
        -DLLVM_ENABLE_PROJECTS="clang;lld" \
        -DLLVM_ENABLE_RUNTIMES="compiler-rt;libcxx;libcxxabi;libunwind" \
        -DLLVM_TARGETS_TO_BUILD="X86;RISCV" \
        -DLLVM_ENABLE_ASSERTIONS=OFF \
        -DLLVM_ENABLE_BINDINGS=OFF \
        -DLLVM_ENABLE_RTTI=OFF \
        -DLLVM_ENABLE_EH=OFF \
        -DLLVM_INCLUDE_TESTS=OFF \
        -DLLVM_INCLUDE_EXAMPLES=OFF \
        -DLLVM_INCLUDE_BENCHMARKS=OFF \
        -DLLVM_INSTALL_UTILS=ON \
        -DLLVM_BUILD_LLVM_DYLIB=ON \
        -DLLVM_LINK_LLVM_DYLIB=ON \
        -DLLVM_OPTIMIZED_TABLEGEN=ON \
        -DCLANG_DEFAULT_CXX_STDLIB=libc++ \
        -DCLANG_DEFAULT_RTLIB=compiler-rt \
        -DCLANG_DEFAULT_UNWINDLIB=libunwind \
        -DCLANG_DEFAULT_LINKER=lld \
        -DCMAKE_INSTALL_PREFIX=/opt/llvm-21 && \
    cmake --build build -j $(nproc) -- all && \
    cmake --build build -- install/strip

FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
        git \
        libc6-dev \
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

ENV CC=/opt/llvm-21/bin/clang
ENV CXX=/opt/llvm-21/bin/clang++
ENV PATH="/opt/llvm-21/bin:${PATH}"
ENV LD_LIBRARY_PATH="/opt/llvm-21/lib/x86_64-unknown-linux-gnu:${LD_LIBRARY_PATH}"
