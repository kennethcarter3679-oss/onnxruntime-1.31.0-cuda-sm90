# ONNX Runtime 1.31.0 (commit 402979ac1bef89dcd55add41eaa7277c407599eb) checked out in /opt/target
# and built with the CUDA EP for sm_90 in /opt/ort-build, both owned by the unprivileged user runner
# (uid 1001), on CUDA 12.9.1 and cuDNN 9, with the same build flags as the /app image.
FROM nvidia/cuda:12.9.1-cudnn-devel-ubuntu24.04@sha256:a2e1e2360c85298ac47ec2543b406ab1e8cec42e31ee47e4d32140ebc82e1067
RUN apt-get update \
 && apt-get install -y --no-install-recommends ca-certificates git cmake ninja-build python3 python3-dev \
      python3-venv build-essential \
 && rm -rf /var/lib/apt/lists/*

RUN python3 -m venv /opt/venv \
 && /opt/venv/bin/pip install --no-cache-dir numpy==2.3.4 packaging==25.0 wheel==0.45.1 setuptools==80.9.0 \
      onnx==1.20.0 pytest==9.1.1 pytest-timeout==2.4.0
ENV PATH="/opt/venv/bin:${PATH}"

RUN mkdir -p /app /logs/verifier
RUN useradd -m -u 1001 runner \
 && mkdir -p /opt/target /opt/ort-build \
 && chown runner:runner /opt/target /opt/ort-build

ARG TARGET_REPO=https://github.com/microsoft/onnxruntime.git
ARG TARGET_SHA=402979ac1bef89dcd55add41eaa7277c407599eb
USER runner
RUN git init -q /opt/target \
 && git -C /opt/target fetch -q --depth 1 "${TARGET_REPO}" "${TARGET_SHA}" \
 && git -C /opt/target checkout -q --detach FETCH_HEAD \
 && git -C /opt/target submodule update -q --init --depth 1 --recursive \
 && git -C /opt/target config user.name verifier \
 && git -C /opt/target config user.email verifier@localhost
RUN cd /opt/target && ./build.sh --config Release --build_dir /opt/ort-build \
      --use_cuda --cuda_home /usr/local/cuda --cudnn_home /usr \
      --build_shared_lib --build_wheel --disable_contrib_ops --skip_tests \
      --parallel 4 --nvcc_threads 1 --cmake_generator Ninja \
      --cmake_extra_defines CMAKE_CUDA_ARCHITECTURES=90 onnxruntime_BUILD_UNIT_TESTS=OFF \
        onnxruntime_USE_FLASH_ATTENTION=OFF onnxruntime_USE_MEMORY_EFFICIENT_ATTENTION=OFF onnxruntime_USE_TRT_FUSED_ATTENTION=OFF
USER root
