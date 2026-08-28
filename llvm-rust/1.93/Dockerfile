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

# Builds a rustc from source, linked against the /opt/llvm-21 build above
# (not a separate LLVM instance bundled in an official rustc release), so
# ASPIS's Rust front-end and its LLVM-IR-consuming pass pipeline share one
# real LLVM build end to end.
#
# 1.93.0 must stay in sync with ASPIS/rust-toolchain.toml (chosen there to
# match LLVM 21.1.8).
RUN git clone --branch 1.93.0 --recurse-submodules --depth 1 --shallow-submodules \
        https://github.com/rust-lang/rust.git /tmp/rust

# Keys confirmed against bootstrap.example.toml on the rust-lang/rust
# 1.93.0 tag. llvm-config points bootstrap at our own LLVM build instead of
# compiling the (unused, since llvm-config is set) src/llvm-project
# submodule; llvm-has-rust-patches=false because /opt/llvm-21 is built from
# vanilla llvm.org, not rust-lang/llvm-project.
RUN cat <<'EOF' > /tmp/rust/bootstrap.toml
[llvm]
llvm-config = "/opt/llvm-21/bin/llvm-config"
llvm-has-rust-patches = false

[build]
target = ["x86_64-unknown-linux-gnu"]

[install]
prefix = "/opt/rust-llvm21"
EOF

# Stage 1 is the standard "working, self-hosting-capable compiler" target
# (rustc-dev-guide's own recommendation short of a full stage-2
# self-optimizing rebuild): builds rustc + library/std against
# /opt/llvm-21, and carries along the stage0 bootstrap cargo, which
# doesn't link LLVM and so doesn't need rebuilding against our copy.
RUN cd /tmp/rust && python3 x.py install --stage 1

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
