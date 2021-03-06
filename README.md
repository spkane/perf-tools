perf-tools
==========

A miscellaneous collection of in-development and unsupported performance analysis tools for Linux perf_events, aka the "perf" command, and ftrace. Both perf_events and ftrace are core Linux tracing tools, and are included in the Linux kernel source.

These tools are designed to be simple to use, easy to install, and provide advanced performance observability. This collection was written by Brendan Gregg (author of the DTraceToolkit).

Many of these tools employ workarounds so that functionality is possible on existing Linux kernels. Because of this, many tools have caveats (see man pages), and their implementation should be considered a placeholder until future kernel features, or new tracing subsystems, are added.

## Contents

Using perf_events:

- misc/[perf-stat-hist](misc/perf-stat-hist): power-of aggregations for tracepoint variables. [Examples](examples/perf-stat-hist_example.txt).
- disk/[bitesize](disk/bitesize): histogram summary of disk I/O size. [Examples](examples/bitesize_example.txt).

Using ftrace:

- [iosnoop](iosnoop): trace disk I/O with details including latency. [Examples](examples/iosnoop_example.txt).
- [iolatency](iolatency): summarize disk I/O latency as a histogram. [Examples](examples/iolatency_example.txt).
- [execsnoop](execsnoop): trace process exec() with command line argument details. [Examples](examples/execsnoop_example.txt).
- [opensnoop](opensnoop): trace open() syscalls showing filenames. [Examples](examples/opensnoop_example.txt).
- kernel/[funccount](kernel/funccount): count kernel functions that match a string. [Examples](examples/funccount_example.txt).
- kernel/[functrace](kernel/functrace): trace kernel functions that match a string. [Examples](examples/functrace_example.txt).
- kernel/[kprobe](kernel/kprobe): trace a given kprobe definition. [Examples](examples/kprobe_example.txt).
- tools/[reset-ftrace](tools/reset-ftrace): reset ftrace state if needed. [Examples](examples/reset-ftrace_example.txt).

## Prerequisites

The intent is as few as possible. Eg, a Linux 3.2 server without debuginfo. See the tool man page for specifics.

### perf_events

Requires the "perf" command to be installed. This is in the linux-tools-common package. After installing that, perf may tell you to install an additional linux-tools package (linux-tools-_kernel-version_). perf can also be built under tools/perf in the kernel source. See [perf_events Prerequisites](http://www.brendangregg.com/perf.html#Prerequisites) for more details.

### ftrace

FTRACE configured in the kernel. You may already have this configured and available in your kernel version, as FTRACE was first added in 2.6.27. This requires CONFIG_FTRACE and other FTRACE options depending on the tool. Some tools (eg, funccount) require CONFIG_FUNCTION_PROFILER.

## Install

These are just scripts. Either grab everything:

```
git clone https://github.com/brendangregg/perf-tools
```

Or use the raw links on github to download individual scripts. Eg

```
wget https://raw.githubusercontent.com/brendangregg/perf-tools/master/iosnoop
```

This preserves tabs (which copy-n-paste can mess up).

## Internals, Warnings, and Overhead

perf_events is evolving. This collection began development circa Linux 3.16, where perf_events lacks certain programmatic capabilities (eg, custom in-kernel aggregations). It's possible these will be added in a forthcoming kernel release. Until then, many of these tools employ workarounds, tricks, and hacks in order to work. This includes passing event data to user space for post-processing, which costs much higher overhead than in-kernel aggregations.

__WARNING__: In extreme cases, your target application may run 5x slower when using these tools. Depending on the tool and kernel version, there may also be the risk of kernel panics. Read the program header for warnings, and test before use.

If the overhead is a problem, these tools can be improved. If a tool doesn't already, it could be rewritten in C to use perf_events_open() and mmap() for the trace buffer. It could also implement frequency counts in C, and operate on mmap() directly, rather than using awk/Perl/Python. Additional improvements are possible for ftrace-based tools, such as use of snapshots and per-instance buffers.

Some of these tools are intended as short-term workarounds until more kernel capabilities exist, at which point they can be substantially rewritten. Older versions of these tools will be kept in this repository, for older kernel versions.

As my main target is a fleet of Linux 3.2 servers that do not have debuginfo, these tools try not to require it. At times, this makes the tool more brittle than it needs to be, as I'm employing workarounds (that may be kernel version and platform specific) instead of using debuginfo information (which can be generic). See the man page for detailed prerequisites for each tool.

Since things are changing, it's very possible you may find some tools don't work on your Linux kernel version. Some expertise and assembly will be required to fix them.
