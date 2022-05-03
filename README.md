# Naga

[![Matrix](https://img.shields.io/badge/Matrix-%23naga%3Amatrix.org-blueviolet.svg)](https://matrix.to/#/#naga:matrix.org)
[![Crates.io](https://img.shields.io/crates/v/naga.svg?label=naga)](https://crates.io/crates/naga)
[![Docs.rs](https://docs.rs/naga/badge.svg)](https://docs.rs/naga)
[![Build Status](https://github.com/gfx-rs/naga/workflows/pipeline/badge.svg)](https://github.com/gfx-rs/naga/actions)
![MSRV](https://img.shields.io/badge/rustc-1.56+-blue.svg)
[![codecov.io](https://codecov.io/gh/gfx-rs/naga/branch/master/graph/badge.svg?token=9VOKYO8BM2)](https://codecov.io/gh/gfx-rs/naga)

The shader translation library for the needs of [wgpu](https://github.com/gfx-rs/wgpu).

## Supported end-points

Front-end       |       Status       | Feature | Notes |
--------------- | ------------------ | ------- | ----- |
SPIR-V (binary) | :white_check_mark: | spv-in  |       |
WGSL            | :white_check_mark: | wgsl-in | Fully validated |
GLSL            | :ok:               | glsl-in | GLSL 440+ |

Back-end        |       Status       | Feature  | Notes |
--------------- | ------------------ | -------- | ----- |
SPIR-V          | :white_check_mark: | spv-out  |       |
WGSL            | :ok:               | wgsl-out |       |
Metal           | :white_check_mark: | msl-out  |       |
HLSL            | :white_check_mark: | hlsl-out | Shader Model 5.0+ (DirectX 11+) |
GLSL            | :ok:               | glsl-out | GLSL 330+ and GLSL ES 300+ |
AIR             |                    |          |       |
DXIL/DXIR       |                    |          |       |
DXBC            |                    |          |       |
DOT (GraphViz)  | :ok:               | dot-out  | Not a shading language |

:white_check_mark: = Primary support — :ok: = Secondary support — :construction: = Unsupported, but support in progress

## Conversion tool

Naga can be used as a CLI, which allows to test the conversion of different code paths.

First, install `naga-cli` from crates.io or directly from GitHub.

```bash
# release version
cargo install naga-cli

# development version
cargo install naga-cli --git https://github.com/gfx-rs/naga.git
```

Then, you can run `naga` command.

```bash
naga my_shader.wgsl # validate only
naga my_shader.spv my_shader.txt # dump the IR module into a file
naga my_shader.spv my_shader.metal --flow-dir flow-dir # convert the SPV to Metal, also dump the SPIR-V flow graph to `flow-dir`
naga my_shader.wgsl my_shader.vert --profile es310 # convert the WGSL to GLSL vertex stage under ES 3.20 profile
```

## Development workflow

The main instrument aiding the development is the good old `cargo test --all-features --workspace`,
which will run the unit tests, and also update all the snapshots. You'll see these
changes in git before committing the code.

If working on a particular front-end or back-end, it may be convenient to
enable the relevant features in `Cargo.toml`, e.g.
```toml
default = ["spv-out"] #TEMP!
```
This allows IDE basic checks to report errors there, unless your IDE is sufficiently configurable already.

Finally, when changes to the snapshots are made, we should verify that the produced shaders
are indeed valid for the target platforms they are compiled for. We automate this with `Makefile`:
```bash
make validate-spv # for Vulkan shaders, requires SPIRV-Tools installed
make validate-msl # for Metal shaders, requires XCode command-line tools installed
make validate-glsl # for OpenGL shaders, requires GLSLang installed
make validate-dot # for dot files, requires GraphViz installed
make validate-wgsl # for WGSL shaders
make validate-hlsl # for HLSL shaders. Note: this Make target makes use of the "sh" shell. This is not the default shell in Windows.
```
