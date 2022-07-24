# (Experimental) Mach Zig Game Engine on PinePhone

Can we run the Mach Game Engine (in Zig) on PinePhone?

https://machengine.org/

Let's find out!

# Build Mach

Follow these steps to download Zig, download Mach and build Mach...

https://gist.github.com/lupyuen/ff77c494b0589371b44b6c96f8491e31

# Fixing Missing Arm64 Atomics

Apply this fix for the Missing Arm64 Atomics...

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

# Mach Crashes

After fixing, mach builds OK but crashes when we run it...

```text
$ export GPU_BACKEND=opengl
$ zig-out/bin/example-rotating-cube
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=0
__aarch64_ldadd8_relax: x=1, p=0x4eb3f18, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=1
__aarch64_ldadd8_relax: x=1, p=0x4eb3f28, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=2
__aarch64_ldadd8_relax: x=1, p=0x4eb3f38, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=3
__aarch64_ldadd8_relax: x=1, p=0x4eb3f58, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=4
__aarch64_ldadd8_relax: x=1, p=0x4eb3f68, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=5
__aarch64_ldadd8_relax: x=1, p=0x4eb3f78, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=6
__aarch64_ldadd8_relax: x=1, p=0x4eb3f98, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=7
__aarch64_ldadd8_relax: x=1, p=0x4eb3fa8, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=8
__aarch64_ldadd8_relax: x=1, p=0x4eb3fb8, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=9
__aarch64_ldadd8_relax: x=1, p=0x4eb3fc8, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=10
__aarch64_ldadd8_relax: x=1, p=0x4eb3fd8, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=11
__aarch64_ldadd8_relax: x=1, p=0x4eb4008, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=12
__aarch64_ldadd8_relax: x=1, p=0x4eb4038, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=13
__aarch64_ldadd8_relax: x=1, p=0x4eb4048, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=14
__aarch64_ldadd8_relax: x=1, p=0x4eb4058, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=15
__aarch64_ldadd8_relax: x=1, p=0x4eb4068, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=16
__aarch64_ldadd8_relax: x=1, p=0x4eb4078, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=17
__aarch64_ldadd8_relax: x=1, p=0x4eb4088, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=18
__aarch64_ldadd8_relax: x=1, p=0x4eb4098, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=19
__aarch64_ldadd8_relax: x=1, p=0x4eb40a8, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=20
__aarch64_ldadd8_relax: x=1, p=0x4eb40b8, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=21
__aarch64_ldadd8_relax: x=1, p=0x4eb40c8, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=22
__aarch64_ldadd8_relax: x=1, p=0x4eb40d8, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=23
__aarch64_ldadd8_relax: x=1, p=0x4eb40e8, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=24
__aarch64_ldadd8_relax: x=1, p=0x4eb40f8, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=25
__aarch64_ldadd8_relax: x=1, p=0x4eb4118, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=26
__aarch64_ldadd8_relax: x=1, p=0x4eb4138, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=27
__aarch64_ldadd8_relax: x=1, p=0x4eb4158, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=28
__aarch64_ldadd8_relax: x=1, p=0x4eb4178, ret=0
__aarch64_ldadd4_acq_rel: x=1, p=0x4eb3550, ret=29
__aarch64_ldadd8_relax: x=1, p=0x4eb4188, ret=0
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=0
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=1
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=2
__aarch64_ldadd8_acq_rel: x=-1, p=0x4eb4198, ret=3
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=2
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=3
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=4
__aarch64_ldadd8_acq_rel: x=-1, p=0x4eb4198, ret=5
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=4
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=5
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=6
__aarch64_ldadd8_acq_rel: x=-1, p=0x4eb4198, ret=7
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=6
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=7
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=8
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=9
__aarch64_ldadd8_acq_rel: x=-1, p=0x4eb4198, ret=10
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=9
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=10
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=11
__aarch64_ldadd8_acq_rel: x=-1, p=0x4eb4198, ret=12
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=11
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=12
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=13
__aarch64_ldadd8_acq_rel: x=-1, p=0x4eb4198, ret=14
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=13
__aarch64_ldadd8_relax: x=1, p=0x4eb4198, ret=14
glfw: error.VersionUnavailable: EGL: Failed to create context: Arguments are inconsistent
error(gpa): memory address 0xffffbf178000 leaked: 
???:?:?: 0x2c9d143 in ??? (???)
???:?:?: 0x2c9cdff in ??? (???)
???:?:?: 0x2c9dc8b in ??? (???)
???:?:?: 0xffffbed77b7f in ??? (???)


error: VersionUnavailable
???:?:?: 0x2cadbd7 in ??? (???)
???:?:?: 0x2ca2cd3 in ??? (???)
???:?:?: 0x2ca2e33 in ??? (???)
???:?:?: 0x2ca2ec3 in ??? (???)
???:?:?: 0x2ca1a93 in ??? (???)
???:?:?: 0x2c9d1df in ??? (???)
???:?:?: 0x2c9ce27 in ??? (???)
```
