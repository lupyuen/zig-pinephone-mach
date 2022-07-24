# (Experimental) Mach Zig Game Engine on PinePhone

Can we run the Mach Game Engine (in Zig) on PinePhone?

https://machengine.org/

Let's find out!

# Build Mach

Follow these steps to download Zig, download Mach and build Mach...

https://gist.github.com/lupyuen/ff77c494b0589371b44b6c96f8491e31

# Fixing Missing Arm64 Atomics

Apply this fix for the Missing Arm64 Atomics...

https://gist.github.com/lupyuen/7cbea20f5be8efa65b971dc2a01374c4

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
