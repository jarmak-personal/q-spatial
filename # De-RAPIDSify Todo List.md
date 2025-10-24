# De-RAPIDSify Todo List

This list outlines steps to decouple the cuSpatial fork from RAPIDS infrastructure while retaining core libraries (e.g., cuDF, RMM). Focus on removing CI, build wrappers, and switching from conda to uv (a fast Python package manager) for pip-based installs.

## Infrastructure Removal
- [ ] Extract useful knowledge from `ci/` directory (e.g., build/test commands) and integrate into local scripts like `build.sh` or new files (e.g., `local_test.sh`). Delete `ci/` after extraction, as we can't use RAPIDS runners.
- [ ] Remove `.github/workflows/` once we can run tests locally and make part of pre-commit.
- [ ] Audit `rapids_config.cmake` for infra-only content.
- [ ] Rewrite `build.sh` to use and `uv` where it makes sense

## Build and Dependency Updates
- [ ] Update `python/libcuspatial/CMakeLists.txt` and `python/cuspatial/CMakeLists.txt` to retain RAPIDS lib finds, stripping infra flags.
- [ ] Drop conda support: Delete the `conda/` directory entirely.
- [ ] Update `dependencies.yaml` to remove RAPIDS-specific entries (e.g., shared CI tools), keeping CUDA/cuDF.
- [ ] Update `pyproject.toml` to remove RAPIDS extras/version pins tied to infra; ensure uv-based installation (e.g., use `uv pip install` for dependencies).

## Documentation and Branding
- [ ] Update `README.md` to remove RAPIDS CI/docs links and update build instructions for uv/CMake, including local test guidance.
- [ ] Update `docs/source/conf.py` to remove RAPIDS intersphinx/shared doc links.
- [ ] Update `CHANGELOG.md` to remove RAPIDS PR/issue links related to infra.

## Testing and Validation
- [ ] Retain RAPIDS test frameworks (e.g., Google Test with cuDF), but run via standard `ctest` locally.
- [ ] Update examples/notebooks to avoid RAPIDS data sources requiring infra.
- [ ] Ensure local test runs succeed: Create or update scripts (e.g., `local_test.sh`) to run all tests on Linux, mimicking extracted CI logic without runners.
- [ ] Test builds on Linux with updated `build.sh` and uv installs, followed by local test execution.

## Additional Notes
- Core RAPIDS libraries (cuDF, RMM) remain for functionality.
- Ensure Apache 2.0 license compliance.
- After completing, validate with a full build and run tests.