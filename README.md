![CI](https://github.com/wkz/ply/workflows/CI/badge.svg)

ply
===

Documentation and language reference is available at
[wkz.github.io/ply][3].

A light-weight dynamic tracer for Linux that leverages the kernel's
BPF VM in concert with kprobes and tracepoints to attach probes to
arbitrary points in the kernel. Most tracers that generate BPF
bytecode are based on the LLVM based BCC toolchain. ply on the other
hand has no required external dependencies except for `libc`. In
addition to `x86_64`, ply also runs on `aarch64`, `arm`, `riscv64` and
`powerpc`. Adding support for more ISAs is easy.

`ply` follows the [Little Language][1] approach of yore, compiling ply
scripts into Linux [BPF][2] programs that are attached to kprobes and
tracepoints in the kernel. The scripts have a C-like syntax, heavily
inspired by `dtrace(1)` and, by extension, `awk(1)`.

The primary goals of `ply` are:

   * Expose most of the BPF tracing feature-set in such a way that new
     scripts can be whipped up very quickly to test different
     hypotheses.

   * Keep dependencies to a minimum. Right now Flex and Bison are
     required at build-time, leaving `libc` as the only runtime
     dependency. Thus, `ply` is well suited for embedded targets.

If you need more fine-grained control over the kernel/userspace
interaction in your tracing, checkout the [bcc][4] project which
compiles C programs to BPF using LLVM in combination with a python
userspace recipient to give you the full six degrees of freedom.


Examples
--------

Here are some one-liner examples to show the kinds of questions that
`ply` can help answer.

**What is the distribution of the returned sizes from `read(2)`s to the VFS?**
```
ply 'kretprobe:vfs_read { @["size"] = quantize(retval); }'
```

**Which processes are receiving errors when reading from the VFS?**
```
ply 'kretprobe:vfs_read if (retval < 0) { @[pid, comm, retval] = count(); }'
```

**Which files are being opened, by who?**
```
ply 'kprobe:do_sys_open { printf("%v(%v): %s\n", comm, uid, str(arg1)); }'
```

**When sending packets, where are we coming from?**
```
ply 'kprobe:dev_queue_xmit { @[stack] = count(); }'
```

**From which hosts and ports are we receiving TCP resets?**
```
ply 'tracepoint:tcp/tcp_receive_reset {
	printf("saddr:%v port:%v->%v\n",
		data->saddr, data->sport, data->dport);
}'
```


Build and Installation
----------------------
use `AndroidToolsBuild` https://github.com/dodola/AndroidToolsBuild

```
git clone git@github.com:dodola/AndroidToolsBuild.git
cd AndroidToolsBuild
./scripts/download-ndk.sh
make ply
```

❗️For Android
------------------
Only test on `Pixel 4 android 13`

Kernel Branch： `android-msm-coral-4.14-android13`

Some Change 
1. change `index` `rindex` to `strchr` `strrchr` 
2. remove self check
3. Fixed KERNEL VERSION for pixel4（DO NOT USE NDK LINUX HEADER KERNEL VERSION）

Run on android
-------------
push `libply.so` `ply` to `/data/local/tmp`

```
$LD_LIBRARY_PATH:/data/local/tmp/" /data/local/tmp/ply 'kprobe:do_nanosleep { printf("PID %d sleeping...\n", pid);}'
```

sample result
```
flame:/data/local/tmp/bpftools # LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/data/local/tmp/ply/lib" /data/local/tmp/ply/sbin/ply 'kprobe:do_nanosleep { printf("PID %d sleeping...\n", pid);}'
PID 742 sleeping...
PID 742 sleeping...
PID 742 sleeping...
PID 742 sleeping...
PID 742 sleeping...
PID 742 sleeping...
```


Contributing
------------

Contributions are welcome! To help you on your way, the [test/](test/)
directory contains ready-made infrastructure to:

- Test cross-compilation on all supported architectures.
- Run a simple test suite on a range of machines using QEMU system
  emulation.
- Run interactive sessions on QEMU machines.

A GitHub Action is setup to run these jobs. Please make sure to test
your changes locally before opening a PR to avoid unnecessary review
cycles.

Maintainers
-----------

`ply` is developed and maintained by [Tobias Waldekranz][5]. Please
direct all bug reports and pull requests towards the official
[Github][6] repo.

[1]: http://c2.com/cgi/wiki?LittleLanguage
[2]: https://www.kernel.org/doc/Documentation/networking/filter.txt
[3]: https://wkz.github.io/ply
[4]: https://github.com/iovisor/bcc
[5]: mailto://tobias@waldekranz.com
[6]: https://github.com/iovisor/ply
