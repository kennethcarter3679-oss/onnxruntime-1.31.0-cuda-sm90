# ONNX Runtime 1.31.0 CUDA builds for sm_90

Two images of the same ONNX Runtime build, published to GHCR by `.github/workflows/build.yml` and
meant to be referenced by digest:

- `app-<commit>`: the source checked out in `/app` and built in `/opt/ort-build`, as root.
- `opt-target-<commit>`: the source checked out in `/opt/target` and built in `/opt/ort-build`,
  both owned by the user `runner` (uid 1001).

Both start from `nvidia/cuda:12.9.1-cudnn-devel-ubuntu24.04`, pinned by digest, and build ONNX Runtime
at commit `402979ac1bef89dcd55add41eaa7277c407599eb` (v1.31.0) with the CUDA EP for `sm_90`, contrib
ops and unit tests off, and the flash, memory-efficient and TensorRT fused attention kernels off. The
build tree keeps its object files, so an edit is rebuilt incrementally with
`cmake --build /opt/ort-build/Release`. Each Python package is pinned in the Dockerfiles.

The workflow also pulls the `/opt/target` image cold, changes one comment in
`include/onnxruntime/core/framework/op_kernel.h` and times the rebuild that triggers, with the disk
and memory it used. The digests and timings are in each run's summary.
