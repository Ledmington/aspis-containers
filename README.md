# ASPIS CI containers

A collection of Docker containers with the dependencies for the [ASPIS](https://github.com/HEAPLab/ASPIS) project.

For each PR targeting `master`, all enabled images are built. For each push on `master`, all enabled images are built and pushed to `docker.io/ledmington/aspis-deps`.

`llvm-builder/21` compiles LLVM from source and is never published itself — it's a shared build input for the `llvm/21` and `llvm-rust/1.93` images, so LLVM is compiled once and reused rather than rebuilt per image. To build one of the published images locally, build the builder image first:

```bash
docker build -t llvm21-builder -f llvm-builder/21/Dockerfile llvm-builder/21
docker build -t aspis-deps:llvm21 -f llvm/21/Dockerfile llvm/21
docker build -t aspis-deps:llvm-rust1.93 -f llvm-rust/1.93/Dockerfile llvm-rust/1.93
```
