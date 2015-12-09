---
layout: post
title:  "Rust Profiling on OSX: CPU/Time"
date:   2015-12-08 20:47:18
categories:
---

I recently decided I wanted to figure out how to measure the performance of the Rust code I've been writing and see pretty graphs of what parts of the code were slower. I did a bunch of research and here's what I found out!

My somewhat-arbitrary constraints were:

* Profile Rust programs' time (I'd like to see memory usage too, but that'll be in another post)
* In a way that produces graphs
* On OSX (Linux has other, some may say better, options, but for getting a first look at the performance of my code at all, I don't feel like booting a VM)

If these constraints are the same as yours, great! Read on! However, there are many more tools that didn't meet these constraints (or that I didn't find), so I encourage you to do research specific to your situation as well.

## Prerequisites

For all of the profiling methods I'm covering, I did the following:

* Build with `cargo build --release` then run `./target/release/[binary]`; or use `cargo run --release`
  * Forgetting to build in release mode (which turns on optimizations) is the #1 cause of unexpected Rust slowness.
  * By default, cargo builds in debug mode, which is faster to compile but runs more slowly.
  * We want to profile the optimized code.
* Turn on debug symbols in your release profile in Cargo.toml
  * This lets the profiling tools match up to your source code.
  * As documented on [crates.io](http://doc.crates.io/manifest.html#the-[profile.*]-sections), this is done by adding this to your `Cargo.toml`:

    ```
    [profile.release]
    debug = true
    ```

* Use OSX's system allocator instead of jemalloc
  * This requires using nightly Rust.
  * I'm not going to pretend to understand this, but it seems like [valgrind might not report allocations correctly with jemalloc](https://github.com/rust-lang/rust/issues/28224)? It didn't seem to hurt anything, but I'm not sure how important this is.
  * [More discussion on /r/rust about the impact of this on benchmarks](https://www.reddit.com/r/rust/comments/3p0ljg/forcing_rustc_to_use_the_system_allocator/).
  * As documented in [the book](http://doc.rust-lang.org/nightly/book/custom-allocators.html), this is done by adding this code to your crate:

    ```
    #![feature(alloc_system)]
    extern crate alloc_system;
    ```

## Option 1: Instruments + Flamegraph

Instruments is a part of XCode, and [Flamegraph](http://www.brendangregg.com/flamegraphs.html) is an awesome collection of perl scripts that can take the output of a few different profiling tools and produce a lovely SVG.



Time:

* xcode menu -> open developer tools -> instruments
* choose blank
* add a thing - time profiler,
* click on all processes, wait a long time
* choose your release executable
* if needed, specify arguments
* change sample interval from 1 ms to 40 microseconds
* record button
* View -> Snap Track to Fit
* https://schani.wordpress.com/2012/11/16/flame-graphs-for-instruments/
* Expand all the nodes in the call tree: Option-click on all roots
* Instrument -> Export track
* Using https://github.com/brendangregg/FlameGraph/pull/79, `./stackcollapse-instruments.pl ~/rust/profiling/instruments-data.csv  | ./flamegraph.pl > instruments-data.svg`
* Open svg in your browser = Pretty graph!!

Memory:

* In Instruments, click on the plus sign in the upper right
* Choose "VM Tracker" (double click or drag and drop to the upper left where your CPU Usage is)
* Record button
* View -> Snap Track to Fit
* Hover over time bar at the top to see numbers
* See that the graph of memory usage is going up over time!
* Export for this track is greyed out; can't figure out how to get this data to generate memory flamegraph http://www.brendangregg.com/FlameGraphs/memoryflamegraphs.html

## Valgrind + massif

* brew install valgrind (massif is part of it)
* `valgrind --tool=massif --time-unit=B ./target/release/profiling`
* http://valgrind.org/docs/manual/ms-manual.html
* `ms_print massif.out.#####`
* pretty ascii graph!

## kcachegrind

## Dtrace (better than instruments alone?)