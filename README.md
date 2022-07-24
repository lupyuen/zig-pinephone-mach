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
