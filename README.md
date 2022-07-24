# (Experimental) Mach Zig Game Engine on PinePhone

Can we run the Mach Game Engine (in Zig) on PinePhone?

https://machengine.org/

https://github.com/hexops/mach

Let's find out! (With Manjaro Phosh)

# Build Mach

Follow these steps to download Zig, download Mach and build Mach...

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

# Fixing Missing Arm64 Atomics

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

After fixing, Mach builds OK...

```bash
$ zig build example-rotating-cube -Ddawn-from-source=true
```

# GLFW Error

Mach fails with a GLFW Error when we run it...

```bash
$ zig build example-rotating-cube -Ddawn-from-source=true
$ export GPU_BACKEND=opengl
$ zig-out/bin/example-rotating-cube
glfw: error.VersionUnavailable: GLX: Failed to create context: GLXBadFBConfig
error: VersionUnavailable
```

[(See complete log)](https://gist.github.com/lupyuen/700efb3b25463bc042ce9e23169efb18)

Which might be caused by the OpenGL Version on PinePhone...

# OpenGL Version

PinePhone supports OpenGL version 2.1, which is rather old...

```text
$ glxinfo | grep "OpenGL version"
OpenGL version string: 2.1 Mesa 22.1.3
```

Overriding the OpenGL version doesn't help, we get the same error...

```bash
$ export MESA_GL_VERSION_OVERRIDE=4.3
$ glxinfo | grep "OpenGL version"
OpenGL version string: 4.3 (Compatibility Profile) Mesa 22.1.3

$ export GPU_BACKEND=opengl
$ zig-out/bin/example-rotating-cube
glfw: error.VersionUnavailable: GLX: Failed to create context: GLXBadFBConfig
error: VersionUnavailable
```

# Pinebook Pro

TODO: Will Mach run on Pinebook Pro?
