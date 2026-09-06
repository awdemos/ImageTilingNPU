# ImageTilingNPU

Agent context for the ImageTilingNPU repository (AMD XDNA NPU super-resolution inference).

## Project overview

- Language: C++20
- Build system: CMake ≥ 3.18
- Platform: Windows with AMD XDNA NPU hardware, Visual Studio 2022, and the AMD NPU driver installed.
- Layout: `src/` (application source), `scripts/` (build/test/model-prep helpers), `voe_package/` and `third_party/` (external dependencies), `rel/` (release binaries copied by CMake POST_BUILD).
- Runtime: ONNX Runtime with VitisAI EP or the Vitis AI graph engine (xmodel).

## Prerequisites

1. Windows 10/11 with AMD XDNA NPU and driver.
2. Visual Studio 2022.
3. CMake ≥ 3.18.
4. Python 3.
5. Download `imagetiling_dep.zip` from AMD and extract `voe_package/` and `third_party/` into the project root.

## Setup commands

After extracting dependencies, place an ONNX model at `model/test.onnx` and an xclbin under `xclbin/`.

```powershell
# Configure and build Release
scripts\build.bat
```

`build.bat` configures CMake, builds the four executables, and copies runtime DLLs into `rel/`.

## Run / test commands

Run the helper scripts in order from the project root:

```powershell
scripts\inject_vaip_stride.bat   # inject SR stride shapes into VAIP config
scripts\gen_xmodel.bat           # ONNX → xmodel
scripts\gen_offset_grid.bat     # generate offset grid for the input image
scripts\func_xmodel.bat          # functional test (zero-copy xmodel)
scripts\func_onnx.bat           # functional test (ONNX CPU EP by default)
scripts\perf_xmodel.bat         # xmodel performance test (NPU turbo, 60 s)
```

After `build.bat`, `rel/` contains:

- `test_SR_onnx.exe`, `perf_SR_onnx.exe`, `test_SR_xmodel.exe`, `perf_SR_xmodel.exe`
- `vitis-ai-runtime.dll`, `opencv_world4110.dll`
- ONNX Runtime VitisAI EP DLLs from `voe_package/bin/`

If `xrt_coreutil.dll` is not in `rel/`, add the XRT `bin` directory to `PATH` before running xmodel tests.

## Key conventions

- C++20 with `/Zc:__cplusplus /Zi /ZH:SHA_256 /guard:cf /sdl /MP`.
- `add_compile_definitions(GLOG_NO_ABBREVIATED_SEVERITIES)`.
- External binaries and headers live in `voe_package/` (ONNX Runtime + VitisAI EP) and `third_party/` (Vitis AI runtime, OpenCV, glog, XRT).
- Do not commit `voe_package/`, `third_party/`, `rel/`, `model/`, `xclbin/`, or `vaip_config/` artifacts.

## Important gotchas

- The project is Windows-only and targets the AMD XDNA NPU; it cannot build or run on Linux/macOS without significant changes.
- `voe_package/` and `third_party/` are not in the repo; they must be obtained from `imagetiling_dep.zip`.
- Model files (`model/test.onnx`) and xclbin (`xclbin/`) are also external inputs.
- Script order matters: stride injection, xmodel generation, and offset-grid generation must happen before functional/perf tests.
- `perf_xmodel.bat` enables NPU turbo mode and runs for 60 seconds.

## Useful shortcuts

```powershell
# Build only
scripts\build.bat

# Run ONNX functional test after build and prep
scripts\func_onnx.bat
```
