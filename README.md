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
$ zig build example-rotating-cube -Ddawn-from-source=true
$ ./zig-out/bin/example-rotating-cube 
__aarch64_ldadd4_acq_rel
__aarch64_ldadd8_relax
__aarch64_ldadd4_acq_rel
__aarch64_ldadd8_relax
__aarch64_ldadd8_acq_rel
free(): invalid pointer
Aborted (core dumped)
```

GDB Stack Trace shows that it's crashing in streambuf...

```text
(gdb) bt
#0  0x0000fffff7c52790 in ?? () from /lib/libc.so.6
#1  0x0000fffff7c0b6fc in raise () from /lib/libc.so.6
#2  0x0000fffff7bf78b0 in abort () from /lib/libc.so.6
#3  0x0000fffff7c4633c in ?? () from /lib/libc.so.6
#4  0x0000fffff7c5cf1c in ?? () from /lib/libc.so.6
#5  0x0000fffff7c5ee38 in ?? () from /lib/libc.so.6
#6  0x0000fffff7c61a68 in free () from /lib/libc.so.6
#7  0x0000000004de9cd4 in std::__1::__shared_count::__release_shared() ()
#8  0x0000000004de1424 in void std::__1::locale::__imp::install<std::__1::collate<wchar_t> >(std::__1::collate<wchar_t>*) ()
#9  0x0000000004dbc2a4 in std::__1::locale::__imp::__imp(unsigned long) ()
#10 0x0000000004dbdd68 in std::__1::locale::__imp::make_global() ()
#11 0x0000000004dbde7c in std::__1::locale::locale() ()
#12 0x0000000004db7d64 in std::__1::basic_streambuf<char, std::__1::char_traits<char> >::basic_streambuf() ()
#13 0x0000000004dbaabc in std::__1::DoIOSInit::DoIOSInit() ()
#14 0x0000000004dbbe0c in global constructors keyed to 000100 ()
#15 0x0000000004dfa318 in __libc_csu_init ()
#16 0x0000fffff7bf7c3c in __libc_start_main () from /lib/libc.so.6
#17 0x0000000002c92344 in _start ()
```
