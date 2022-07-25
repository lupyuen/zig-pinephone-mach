# (Experimental) Mach Zig Game Engine on PinePhone

Can we run the Mach Game Engine (in Zig) on PinePhone?

https://machengine.org/

https://github.com/hexops/mach

Let's find out! (With Manjaro Phosh)

```text
██████████████████  ████████   manjaro@manjaro-arm 
██████████████████  ████████   ------------------- 
██████████████████  ████████   OS: Manjaro ARM Linux aarch64 
██████████████████  ████████   Host: Pine64 PinePhone (1.2) 
████████            ████████   Kernel: 5.18.3-1-MANJARO-ARM 
████████  ████████  ████████   Uptime: 1 hour, 48 mins 
████████  ████████  ████████   Packages: 806 (pacman) 
████████  ████████  ████████   Shell: bash 5.1.16 
████████  ████████  ████████   Resolution: 720x1440 
████████  ████████  ████████   Terminal: node 
████████  ████████  ████████   CPU: (4) @ 1.152GHz 
████████  ████████  ████████   Memory: 507MiB / 1991MiB 
████████  ████████  ████████
████████  ████████  ████████       
```

# Build Mach

Follow these steps to download Zig, download Mach and build Mach on PinePhone...

```bash
$ curl -O -L https://ziglang.org/builds/zig-linux-aarch64-0.10.0-dev.3080+eb1b2f5c5.tar.xz
$ tar xf zig-linux-aarch64-0.10.0-dev.3080+eb1b2f5c5.tar.xz
$ export PATH="$HOME/zig-linux-aarch64-0.10.0-dev.3080+eb1b2f5c5:$PATH"
$ git clone https://github.com/hexops/mach
$ cd mach
$ zig build example-rotating-cube -Ddawn-from-source=true
```

[(See the complete log)](https://gist.github.com/lupyuen/ff77c494b0589371b44b6c96f8491e31)

Be patient, it takes roughly 1.5 hours for the first build. Subsequent builds will complete in seconds.

# Missing Arm64 Atomics

Mach fails to build on PinePhone due to Missing Arm64 Atomics...

```bash
$ zig build example-rotating-cube -Ddawn-from-source=true
LLD Link... 
ld.lld: error: undefined symbol: __aarch64_ldadd8_relax
ld.lld: error: undefined symbol: __aarch64_ldadd8_acq
ld.lld: error: undefined symbol: __aarch64_ldadd8_rel
ld.lld: error: undefined symbol: __aarch64_ldadd8_acq_rel
ld.lld: error: undefined symbol: __aarch64_ldadd4_relax
ld.lld: error: undefined symbol: __aarch64_ldadd4_acq
ld.lld: error: undefined symbol: __aarch64_ldadd4_rel
ld.lld: error: undefined symbol: __aarch64_ldadd4_acq_rel
ld.lld: error: undefined symbol: __aarch64_swp8_acq_rel
ld.lld: error: undefined symbol: __aarch64_cas8_acq_rel
```

[(See the complete log)](https://gist.github.com/lupyuen/ff77c494b0589371b44b6c96f8491e31)

This is caused by Zig's incomplete support for LLVM and Atomics...

https://github.com/ziglang/zig/issues/10086

So we copy the Arm64 Atomics from this Rust Implementation...

https://github.com/suptalentdev/mustang/blob/master/c-scape/aarch64_outline_atomics/aarch64_outline_atomics/src/lib.rs

https://docs.rs/atomic/latest/atomic/struct.Atomic.html

# Fix Missing Arm64 Atomics

To build Mach correctly on PinePhone, apply this fix for the Missing Arm64 Atomics...

```zig
//// TODO: To fix the Missing Atomics for Mach Zig Game Engine on PinePhone Manjaro Phosh,
//// Insert these lines at the bottom of mach/examples/rotating-cube/main.zig
////
//// Here are the Linker Errors for the Missing Atomics: 
//// https://gist.github.com/lupyuen/ff77c494b0589371b44b6c96f8491e31
////
//// Refer to this issue: https://github.com/ziglang/zig/issues/10086
//// Rust Implementation: https://github.com/suptalentdev/mustang/blob/master/c-scape/aarch64_outline_atomics/aarch64_outline_atomics/src/lib.rs
//// Rust Atomic Struct: https://docs.rs/atomic/latest/atomic/struct.Atomic.html

pub export fn __aarch64_ldadd4_relax()   c_int { _ = puts("TODO: __aarch64_ldadd4_relax");   return 0; }
pub export fn __aarch64_ldadd4_acq()     c_int { _ = puts("TODO: __aarch64_ldadd4_acq");     return 0; }
pub export fn __aarch64_ldadd4_rel()     c_int { _ = puts("TODO: __aarch64_ldadd4_rel");     return 0; }

pub export fn __aarch64_ldadd4_acq_rel(x: u32, p: [*c]u32) u32 { 
    // (*p).fetch_add(x, AcqRel)
    // Add to the current value, returning the previous value.
    // AcqRel: For loads it uses Acquire ordering. For stores it uses the Release ordering.
    const ret = p.*;
    _ = printf("__aarch64_ldadd4_acq_rel: x=%d, p=%p, ret=%d\n", x, p, ret);
    p.* +%= x;
    return ret; 
}

pub export fn __aarch64_ldadd8_relax(x: u32, p: [*c]u32) u32 { 
    // (*p).fetch_add(x, Relaxed)
    // Add to the current value, returning the previous value.
    // Relaxed: No ordering constraints, only atomic operation
    const ret = p.*;
    _ = printf("__aarch64_ldadd8_relax: x=%d, p=%p, ret=%d\n", x, p, ret);
    p.* +%= x;
    return ret; 
}

pub export fn __aarch64_ldadd8_acq()     c_int { _ = puts("TODO: __aarch64_ldadd8_acq");     return 0; }

pub export fn __aarch64_ldadd8_rel(x: u64, p: [*c]u64) u64 {
    // (*p).fetch_add(x, AcqRel)
    // Add to the current value, returning the previous value.
    const ret = p.*;
    _ = printf("__aarch64_ldadd8_rel: x=%d, p=%p, ret=%d\n", x, p, ret);
    p.* +%= x;
    return ret; 
}

pub export fn __aarch64_ldadd8_acq_rel(x: u64, p: [*c]u64) u64 {
    // (*p).fetch_add(x, AcqRel)
    // Add to the current value, returning the previous value.
    // AcqRel: For loads it uses Acquire ordering. For stores it uses the Release ordering.
    const ret = p.*;
    _ = printf("__aarch64_ldadd8_acq_rel: x=%d, p=%p, ret=%d\n", x, p, ret);
    p.* +%= x;
    return ret; 
}

pub export fn __aarch64_swp8_acq_rel()   c_int { _ = puts("TODO: __aarch64_swp8_acq_rel");   return 0; }
pub export fn __aarch64_cas8_acq_rel()   c_int { _ = puts("TODO: __aarch64_cas8_acq_rel");   return 0; }

extern fn printf(format: [*:0]const u8, ...) c_int;
extern fn puts(str: [*:0]const u8) c_int;
```

After fixing, Mach builds OK on PinePhone...

```bash
$ zig build example-rotating-cube -Ddawn-from-source=true
```

# GLFW Error

When we run Mach on PinePhone, it fails with a GLFW Error...

```bash
$ zig build example-rotating-cube -Ddawn-from-source=true

$ export GPU_BACKEND=opengl
$ zig-out/bin/example-rotating-cube
glfw: error.VersionUnavailable: GLX: Failed to create context: GLXBadFBConfig

$ export GPU_BACKEND=opengles
$ zig-out/bin/example-rotating-cube
glfw: error.VersionUnavailable: EGL: Failed to create context: An EGLConfig argument does not name a valid EGL frame buffer configuration
```

[(See complete log)](https://gist.github.com/lupyuen/700efb3b25463bc042ce9e23169efb18)

Which might be caused by the OpenGL Version on PinePhone...

# OpenGL Version

PinePhone supports OpenGL version 2.1, which might be too old for Mach...

```text
$ glxinfo | grep "OpenGL version"
OpenGL version string: 2.1 Mesa 22.1.3
```

When we set `MESA_GL_VERSION_OVERRIDE`, Mach fails with a GLSL Error ...

```bash
$ zig build example-rotating-cube -Ddawn-from-source=true
$ export GPU_BACKEND=opengl
$ export MESA_GL_VERSION_OVERRIDE=4.5

$ glxinfo | grep 'OpenGL version'
OpenGL version string: 4.5 (Compatibility Profile) Mesa 22.1.3

$ zig-out/bin/example-rotating-cube
mach: found OpenGL backend on Unknown adapter: Mali400, OpenGL version 4.5 (Core Profile) Mesa 22.1.3
gpu: validation error: #version 450
Program compilation failed:
0:1(10): error: GLSL 4.50 is not supported. Supported versions are: 1.10, 1.20, and 1.00 ES
 - While calling [Device].CreateRenderPipeline([RenderPipelineDescriptor]).
```

[(See the complete log)](https://gist.github.com/lupyuen/9d3aca0d10cc6dfee0a7d3e50f14e191)

This says that Mach failed to create the GPU Render Pipeline...

```zig
const vs_module = core.device.createShaderModule(&.{
    .label = "my vertex shader",
    .code = .{ .wgsl = @embedFile("vert.wgsl") },
});

const pipeline_descriptor = gpu.RenderPipeline.Descriptor{
    .fragment = &fragment,
    .layout = pipeline_layout,
    .depth_stencil = null,
    .vertex = .{
        .module = vs_module,
        .entry_point = "main",
        .buffers = &.{vertex_buffer_layout},
```

[(Source)](https://github.com/hexops/mach/blob/main/examples/rotating-cube/main.zig#L87-L107)

Because PinePhone failed to load the [OpenGL Shading Language](https://www.khronos.org/opengl/wiki/OpenGL_Shading_Language) (GLSL) file [vert.wgsl](https://github.com/hexops/mach/blob/main/examples/rotating-cube/vert.wgsl).

This GLSL Error appears when we set `MESA_GL_VERSION_OVERRIDE` to 4.4 or above.

The GLFW Error appears when we set `MESA_GL_VERSION_OVERRIDE` to 4.3 or below.

So it seems Mach only works with OpenGL / GLSL version 4.4 and above. Which isn't supported on PinePhone.

# Pinebook Pro

Will Mach run on Pinebook Pro?

```text
██████████████████  ████████   luppy@pinebook 
██████████████████  ████████   -------------- 
██████████████████  ████████   OS: Manjaro ARM Linux aarch64 
██████████████████  ████████   Host: Pine64 Pinebook Pro 
████████            ████████   Kernel: 5.18.5-1-MANJARO-ARM 
████████  ████████  ████████   Uptime: 53 mins 
████████  ████████  ████████   Packages: 1006 (pacman) 
████████  ████████  ████████   Shell: bash 5.1.16 
████████  ████████  ████████   Resolution: 1920x1080 
████████  ████████  ████████   Terminal: node 
████████  ████████  ████████   CPU: (6) @ 1.416GHz 
████████  ████████  ████████   Memory: 914MiB / 3868MiB 
████████  ████████  ████████
████████  ████████  ████████    
```

We run the same steps to build Mach on Pinebook Pro (Manjaro Xfce)...

https://github.com/lupyuen/zig-pinephone-mach#build-mach

And apply the fix for Missing Arm64 Atomics...

https://github.com/lupyuen/zig-pinephone-mach#missing-arm64-atomics

[(See the build log)](https://gist.github.com/lupyuen/1aaca1615848d86844a4d2873a847439)

(Mach builds on Pinebook Pro in roughly half an hour)

Mach fails with the same GLFW Error on Pinebook Pro even though it supports OpenGL 3.1...

```bash
$ zig build example-rotating-cube -Ddawn-from-source=true
$ export GPU_BACKEND=opengl

$ glxinfo | grep 'OpenGL version'
OpenGL version string: 3.1 Mesa 22.1.3

$ zig-out/bin/example-rotating-cube
glfw: error.VersionUnavailable: GLX: Failed to create context: GLXBadFBConfig
```

[(See the complete log)](https://gist.github.com/lupyuen/46d5e398fad09ec498d8c9c93d82e03a)

When we set `MESA_GL_VERSION_OVERRIDE`, Mach fails with the same GLSL Error...

```bash
$ zig build example-rotating-cube -Ddawn-from-source=true
$ export GPU_BACKEND=opengl
$ export MESA_GL_VERSION_OVERRIDE=4.5

$ glxinfo |  grep 'OpenGL version'
OpenGL version string: 4.5 (Compatibility Profile) Mesa 22.1.3

$ zig-out/bin/example-rotating-cube
mach: found OpenGL backend on Unknown adapter: Mali-T860 (Panfrost), OpenGL version 4.5 (Core Profile) Mesa 22.1.3
gpu: validation error: #version 450
Program compilation failed:
0:1(10): error: GLSL 4.50 is not supported. Supported versions are: 1.10, 1.20, 1.30, 1.40, 1.00 ES, and 3.00 ES
 - While calling [Device].CreateRenderPipeline([RenderPipelineDescriptor]).
```

[(See the complete log)](https://gist.github.com/lupyuen/849491f0aef2b8ad67e8ab1ce250f067)

So it seems Mach won't run on Pinebook Pro because it doesn't support OpenGL / GLSL version 4.4 or later.
