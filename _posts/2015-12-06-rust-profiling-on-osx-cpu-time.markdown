---
layout: post
title:  "Rust Profiling with Instruments and FlameGraph on OSX: CPU/Time"
date:   2015-12-08 20:47:18
categories:
---

I recently decided I wanted to figure out a way to measure the performance of the Rust code I've been writing and see pretty graphs of what parts of the code were slower. There are a bunch of different ways to do this, and I was going to cover more than one of them, but [I am le tired](https://youtu.be/84Ud3V9NPw8?t=42s) so I decided to post this after writing only about Instruments + FlameGraph.

My somewhat-arbitrary constraints used in picking this method of profiling and graphing:

* Profile Rust programs' time (I'd like to profile memory usage too, but that'll be in another post)
* In a way that produces graphs
* On OSX (Linux has other, some may say better, options, but for getting a first look at the performance of my code at all, I don't feel like booting a VM)

If these constraints are the same as yours, great! Read on! However, there are many more tools that didn't meet these constraints (or that I didn't find), so I encourage you to do research specific to your situation as well.

## Prerequisites

For any profiling of Rust code on OSX, it's recommended to do the following:

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

## Protip on inlining

I had to consult with [Jake Goulding](http://jakegoulding.com/) on this one-- I was profiling some code that had a function calling another function, and only the inner function was showing up. I couldn't figure out why. Jake quickly looked at what I was doing and guessed correctly that the compiler was inlining the contents of the outer function as an optimization, so it effectively didn't exist anymore.

Jake taught me about adding the annotation `#[inline(never)]` before the outer function, which will tell the compiler to not inline it so that I could see the profiling for it.

## Instruments

Instruments is a part of XCode. You've already downloaded 2 gigs of XCode, right?

* Open XCode. Don't pick anything on the splash screen.
* From the XCode menu, choose "Open Developer Tool" -> "Instruments".
* Choose a blank profiling template. (Or try others! I haven't explored all the options yet)
* Click the "+" button in the upper right. Choose "Time Profiler" and drag it to the section in the upper left.
* In the upper left, after your computer name (yes, my computer's name is the tableflip emoji), click on where it says "All processes" and click on "Choose Target" in order to choose just your executable instead of all processes. For some reason, populating the list of executables to choose from takes a long time, so don't be alarmed.
  <a href="./assets/img/instruments-screenshot.png"><img src="./assets/img/instruments-screenshot.png" width="560" alt="Screenshot of Instruments showing the location of the Time Profiler, where you drop it, and where you choose your executable" /></a>
* If needed, you can type environment variables and arguments into the "Choose Target" window. Here's an example where I added some arguments:
  <a href="./assets/img/instruments-arguments.png"><img src="./assets/img/instruments-arguments.png" width="560" alt="Screenshot of Choose Target window with some arguments specified" /></a>
* By default, the Time Profiler takes a sample every 1 ms. Because I was using a toy example, I found that I wasn't getting interesting results because my code wasn't being sampled very often, so I changed the sample interval from 1 ms to 40 microseconds in the lower right.
  <a href="./assets/img/instruments-sampling-interval.png"><img src="./assets/img/instruments-sampling-interval.png" alt="Zoomed in screenshot of the sampling interval set to 40 microseconds" style="width: 340px"/></a>
* Click the record button in the upper left and wait for it to run your code. You may need to give administrator privileges to Instruments. You'll know it's done when the button changes back to a record button after being a stop button.
* You do get a little bit of a graph across the top-- From the View menu, choosing "Snap Track to Fit" makes it a little easier to see. It still doesn't show very specific information though.

## Flamegraph

[Flamegraph](http://www.brendangregg.com/flamegraphs.html) is an awesome collection of perl scripts that can take the output of a few different profiling tools and produce a lovely SVG.

* To get the data out of Instruments for Flamegraph to use, I followed [these instructions](https://schani.wordpress.com/2012/11/16/flame-graphs-for-instruments/) which say to:
  * Expand all the nodes in the call tree: Option-click on all the roots to toggle them open
  * From the Instrument menu, choose "Export track" and save the CSV.
* Download the [FlameGraph code](https://github.com/brendangregg/FlameGraph) and run:

  ```
  $ ./stackcollapse-instruments.pl export.csv | ./flamegraph.pl > pretty-graph.svg
  ```
* Open the resulting svg in your browser. You now have a pretty graph!!

[Here's an SVG I made](./assets/img/instruments-flamegraph.svg) from [the toy code in this repo](https://github.com/carols10cents/rust-profiling-example).

Happy profiling!!!
