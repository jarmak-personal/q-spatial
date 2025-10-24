# Build and Install qu-spatial (uv-only)

This project uses Astral's uv for environment management, locking, and builds. No conda required.

## Prerequisites

- Linux x86_64 with a compatible NVIDIA driver for CUDA 12
- Python 3.12 (uv can download/manage it)
- C/C++ toolchain and build tools:
	- cmake >= 3.30
	- ninja
	- cython >= 3.0

## One-time setup

```bash
# Install uv (if not already installed)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Adjust the path if your checkout directory is different
cd /home/picard/Documents/qu-spatial

# Create and activate a virtual env pinned to Python 3.12
uv venv --python 3.12
source .venv/bin/activate

# Install build toolchain
uv pip install "rapids-build-backend>=0.3,<0.4" "scikit-build-core[pyproject]>=0.10" \
	"cython>=3.0" "cmake>=3.30" ninja build

# Add runtime dependencies from NVIDIA index (CUDA 12 variants)
uv add --extra-index-url=https://pypi.nvidia.com \
	cudf-cu12 cuspatial-cu12 rmm-cu12 cupy-cuda12x geopandas numpy

# Lock dependencies (creates uv.lock)
uv lock
```

## Build wheels

```bash
# Build sdist and wheels in ./dist
uv build

# Optionally, install the wheel you just built
uv pip install dist/*.whl
```

## Build libcuspatial (C++) from source with uv (Path 2)

If you need to modify and rebuild the core C++ library, use the `python/libcuspatial` package which wraps the
native code with scikit-build-core/CMake and produces a wheel containing the shared library.

Prerequisites (on the Linux x86_64 host):
- NVIDIA driver compatible with CUDA 12
- CUDA 12 toolkit installed (nvcc, headers, libraries) and visible in PATH/LD_LIBRARY_PATH
- Build tools: cmake>=3.30, ninja, gcc/g++

Steps:

```bash
# 1) Create a clean build venv
cd /home/picard/Documents/qu-spatial/python/libcuspatial
uv venv --python 3.12
source .venv/bin/activate

# 2) Install build toolchain
uv pip install "rapids-build-backend>=0.3,<0.4" "scikit-build-core[pyproject]>=0.10" \
	"cython>=3.0" "cmake>=3.30" ninja build

# 3) Build the libcuspatial wheel
uv build

# 4) Install the produced wheel back into your root environment if desired
uv pip install dist/libcuspatial-*.whl
```

Now your environment has a locally built libcuspatial. When you build/install the Python `cuspatial` package
(`python/cuspatial`) it will link against the installed libcuspatial wheel.

```bash
# (from repository root) build cuspatial Python wheel against your local libcuspatial
cd /home/picard/Documents/qu-spatial/python/cuspatial
uv build
uv pip install dist/cuspatial-*.whl
```

Troubleshooting:
- If CMake cannot locate CUDA, export `CUDA_HOME` and ensure `nvcc` is on PATH.
- If the linker can’t find CUDA or RAPIDS libraries at runtime, adjust `LD_LIBRARY_PATH` accordingly.
- Ensure you are on Linux x86_64; locking is constrained via `[tool.uv].environments`.

## Reproducible installs (CI or fresh machines)

```bash
# Install exactly what's in the lock
uv sync

# Export for pip-only environments
uv export --format requirements --frozen > requirements.txt
```

## Notes

- The file `[tool.uv].environments` in `pyproject.toml` restricts locking to Linux x86_64
	to avoid resolving for unused splits (e.g., aarch64).
- Ensure your system has a compatible NVIDIA driver for CUDA 12 to use the `-cu12` wheels.
