# Rust Profiling on OSX

Goals:

* Profile Rust programs' time and memory
* Make pretty graphs
* On OSX

## Prerequisites

* Turn on debug symbols in your release profile in Cargo.toml
  * http://doc.crates.io/manifest.html#the-[profile.*]-sections
* Use the system allocator
  * http://doc.rust-lang.org/nightly/book/custom-allocators.html
  * https://github.com/rust-lang/rust/issues/28224
  * https://github.com/alexcrichton/rfcs/blob/less-jemalloc/text/0000-swap-out-jemalloc.md
  * https://www.reddit.com/r/rust/comments/3p0ljg/forcing_rustc_to_use_the_system_allocator/

## Instruments + Flamegraph

http://www.brendangregg.com/flamegraphs.html

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

## kcachegrind

## Dtrace (better than instruments alone?)