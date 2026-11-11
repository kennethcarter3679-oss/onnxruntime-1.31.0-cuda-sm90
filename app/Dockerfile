# ONNX Runtime 1.31.0 (commit 402979ac1bef89dcd55add41eaa7277c407599eb) checked out in /app and
# built with the CUDA EP for sm_90 in /opt/ort-build, on CUDA 12.9.1 and cuDNN 9. The build tree's
# Python package is on the path, so a rebuild with `cmake --build /opt/ort-build/Release` is picked up
# without reinstalling anything.
FROM nvidia/cuda:12.9.1-cudnn-devel-ubuntu24.04@sha256:a2e1e2360c85298ac47ec2543b406ab1e8cec42e31ee47e4d32140ebc82e1067
RUN apt-get update \
 && apt-get install -y --no-install-recommends ca-certificates git cmake ninja-build python3 python3-dev \
      python3-venv build-essential \
 && rm -rf /var/lib/apt/lists/*

RUN python3 -m venv /opt/venv \
 && /opt/venv/bin/pip install --no-cache-dir numpy==2.3.4 packaging==25.0 wheel==0.45.1 setuptools==80.9.0 \
      onnx==1.20.0 \
 && printf 'export PATH=/opt/venv/bin:/usr/local/cuda/bin:$PATH\n' > /etc/profile.d/task-path.sh
ENV PATH="/opt/venv/bin:${PATH}"

# Fetched at exactly the v1.31.0 commit, so no newer upstream commit is in the object store.
ARG TARGET_REPO=https://github.com/microsoft/onnxruntime.git
ARG TARGET_SHA=402979ac1bef89dcd55add41eaa7277c407599eb
RUN git init -q /app \
 && git -C /app fetch -q --depth 1 "${TARGET_REPO}" "${TARGET_SHA}" \
 && git -C /app checkout -q --detach FETCH_HEAD \
 && git -C /app submodule update -q --init --depth 1 --recursive \
 && git -C /app config user.name agent \
 && git -C /app config user.email agent@localhost \
 && echo "submission.patch" >> /app/.git/info/exclude

# Built outside /app so build output never enters a diff of /app. The flash, memory-efficient and
# TensorRT fused attention kernels are left out; the flash kernels alone need more than 8 GB to
# compile four at a time.
RUN cd /app && ./build.sh --config Release --build_dir /opt/ort-build \
      --use_cuda --cuda_home /usr/local/cuda --cudnn_home /usr \
      --build_shared_lib --build_wheel --disable_contrib_ops --skip_tests \
      --parallel 4 --nvcc_threads 1 --allow_running_as_root --cmake_generator Ninja \
      --cmake_extra_defines CMAKE_CUDA_ARCHITECTURES=90 onnxruntime_BUILD_UNIT_TESTS=OFF \
        onnxruntime_USE_FLASH_ATTENTION=OFF onnxruntime_USE_MEMORY_EFFICIENT_ATTENTION=OFF onnxruntime_USE_TRT_FUSED_ATTENTION=OFF \
 && echo /opt/ort-build/Release > "$(python3 -c 'import site; print(site.getsitepackages()[0])')/ort-build.pth"
# From /, since Python run in /app finds the source tree's onnxruntime/ package first.
RUN cd / && python3 -c "import onnxruntime; assert onnxruntime.__file__.startswith('/opt/ort-build/')"

WORKDIR /app
