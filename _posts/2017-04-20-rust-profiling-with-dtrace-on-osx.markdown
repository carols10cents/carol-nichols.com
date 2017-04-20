---
layout: post
title:  "Rust Profiling with DTrace and FlameGraph on OSX"
date:   2017-04-20
categories:
---

This is an alternative to my post [Rust Profiling with Instruments and
FlameGraph on OSX: CPU/Time](/2015/12/09/rust-profiling-on-osx-cpu-time/),
since XCode disabled exporting of the Time/CPU data from Instruments. Many
thanks to [this post of BrendanGregg's
](http://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html) that has
examples of using DTrace to get the data to feed into [his FlameGraph
generation scripts](https://github.com/brendangregg/FlameGraph), and
[GolDDranks on Stack Overflow](http://stackoverflow.com/a/43491361/51683) who
figured out how to get OSX's DTrace to find Rust debugging symbols -- I didn't
have the same problem, I'm not sure why, but they inspired me to try DTrace
instead of Instruments and write an update blog post!

So here's the new workflow that worked for me!

## Prerequisites

These are the same.

* Turn on debug symbols in your release profile in Cargo.toml
  * This lets the profiling tools match up to your source code.
  * As documented on [crates.io](http://doc.crates.io/manifest.html#the-[profile.*]-sections), this is done by adding this to your `Cargo.toml`:

    ```
    [profile.release]
    debug = true
    ```
* Build with `cargo build --release` then run `./target/release/[binary]`; or use `cargo run --release`
  * Forgetting to build in release mode (which turns on optimizations) is the #1 cause of unexpected Rust slowness.
  * By default, cargo builds in debug mode, which is faster to compile but runs more slowly.
  * We want to profile the optimized code.

## DTrace

OSX (yes yes I know it's macOS now but it'll always be OSX to me) comes with `dtrace`! I'm on Sierra, and `dtrace -V` says `dtrace: Sun D 1.13` for me.

TL;DR: the command I ran to profile my program named `zopfli` was:

```
sudo dtrace -c './zopfli large-file' -o out.stacks -n 'profile-997 /execname == "zopfli"/ { @[ustack(100)] = count(); }'
```

Here's the explanation of all the pieces:

- **sudo**: Because of Reasons™, you have to run `dtrace` as root on OSX.

- **`-c`**: A lot of the tutorials and blog posts I'm looking at use `dtrace` by running the program, then finding the PID of the running process, then passing the PID as an argument to `dtrace`-- I don't want to do all that, I want to have `dtrace` run my program and watch just what it's running. Luckily, that's what `-c` does! I have a bunch of symlinks set up because Other Reasons™, so the command I use to run my Rust program is `./zopfli large-file`. So that's what I'm doing with the `-c './zopfli large-file'` part, replace my command with the one you want to run under dtrace.

- **`-o`**: This is the output filename that we'll be passing to the flamegraphs scripts in the next section.

- **`-n`**: This argument specifies the probe names that you want to enable. Breaking this down:

  - `profile-997`: interrupt the program 997 times per second. See [the profile provider in the DTrace book](http://dtrace.org/guide/chp-profile.html) for more info.
  - `/execname == "zopfli"/` is a predicate that will fire if the name of the current process's executable file is equal to the string we've specified; you'll want to change this to be your program. I'm not sure why this is needed exactly since `dtrace` should know the executable from the `-c` argument, but I couldn't figure out the right syntax to get the `-n` argument to work without this.
  - `{ @[ustack(100)] = count(); }` is the code to run if the predicate is true. This code stores an aggregate (`@[]`) count (`count()`) of the number of times that `dtrace` sees a particular user stack trace (`ustack`) 100 frames deep.

## FlameGraph

Now that you have the output from dtrace in a file, the flamegraph steps are
similar. Once you have, the [FlameGraph
code](https://github.com/brendangregg/FlameGraph), and assuming you gave `-o`
the argument `out.stacks`, run:

```
$ ./stackcollapse.pl out.stacks | ./flamegraph.pl > pretty-graph.svg
```

Tada!!! You have pretty flame graph SVGs again!!!
