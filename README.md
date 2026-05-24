# whisper-rs-sys (Beypilot fork)

Raw FFI bindings to [whisper.cpp](https://github.com/ggerganov/whisper.cpp), generated
with [`bindgen`](https://github.com/rust-lang/rust-bindgen) over a CMake build of a vendored
copy of whisper.cpp.

This crate is intentionally low-level: it exposes the C API as-is and performs no safety
wrapping. Higher-level, idiomatic Rust APIs belong in a separate `whisper-rs` crate.

## Fork provenance

- Fork of upstream `whisper-rs-sys` **0.15.0** (originally <https://codeberg.org/tazz4843/whisper-rs>).
- Vendors **whisper.cpp 1.8.3** under [`whisper.cpp/`](whisper.cpp/).
- Maintained privately for BeyPilot at <https://github.com/devsoluxhq/whisper-rs-sys>.
- **Not published to crates.io.** Consume it as a path/git dependency.

## Build requirements

The build script (`build.rs`) compiles whisper.cpp via CMake and generates bindings via bindgen,
so the following must be available on the build host:

- A C/C++ toolchain (clang or gcc).
- **CMake** ≥ 3.5.
- **libclang** (for bindgen) — unless `WHISPER_DONT_GENERATE_BINDINGS=1` is set (see below).
- Rust **1.88.0** or newer (MSRV; required by `as_chunks` in the generated bindings target).

Platform-specific backend dependencies (Metal, CUDA, Vulkan, OpenBLAS, OpenMP) are only needed
when the corresponding feature is enabled.

## Feature flags

| Feature      | Effect |
| ------------ | ------ |
| `metal`      | Enable the Apple Metal GPU backend (macOS). Links `Metal`/`MetalKit`/`Foundation`. |
| `coreml`     | Enable the Core ML backend (macOS). Links `CoreML`/`Foundation`. |
| `cuda`       | Enable the NVIDIA CUDA backend. Requires the CUDA toolkit on `PATH`/`CUDA_PATH`. |
| `hipblas`    | Enable the AMD ROCm/HIP backend (Linux only; requires `hipcc`). |
| `vulkan`     | Enable the Vulkan backend. Requires the Vulkan SDK (`VULKAN_SDK`) on Windows/macOS. |
| `openblas`   | Use OpenBLAS for the BLAS backend. Requires `BLAS_INCLUDE_DIRS`; honors `OPENBLAS_PATH`. |
| `openmp`     | Enable OpenMP. On macOS, honors `LIBOMP_PATH`, then `brew --prefix libomp`, then the Homebrew default. |
| `intel-sycl` | Enable the Intel SYCL backend (`icx`/`icpx`); builds shared libraries. |
| `force-debug`| Build whisper.cpp with debug symbols even in release (`-DWHISPER_DEBUG`). |

On macOS the Accelerate framework (BLAS) is always linked by default — no feature required.

## Examples

Default build (CPU backend, plus Accelerate on macOS):

```bash
cargo build
```

macOS Metal backend:

```bash
cargo build --features metal
```

OpenBLAS on Linux:

```bash
BLAS_INCLUDE_DIRS=/usr/include/openblas \
OPENBLAS_PATH=/usr \
cargo build --features openblas
```

## Skipping bindgen

If `libclang` is unavailable, or to speed up CI, set:

```bash
WHISPER_DONT_GENERATE_BINDINGS=1 cargo build
```

This copies the checked-in [`src/bindings.rs`](src/bindings.rs) into `OUT_DIR` instead of
regenerating it. whisper.cpp itself is still compiled via CMake, so a C/C++ toolchain and CMake
are still required. The prebuilt bindings may lag behind the vendored headers, so prefer the
generated path when possible.

## License

`Unlicense`, matching upstream. Vendored whisper.cpp retains its MIT license
(see [`whisper.cpp/LICENSE`](whisper.cpp/LICENSE)).
