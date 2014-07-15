---
layout: post
title:  "Ruby, Rust, and Concurrency"
date:   2014-07-14 21:48:00
categories:
---

I mostly do Ruby these days, but I've just recently started getting into Rust. I'm finding it super
interesting! It's definitely expanding the way I think about programming, and I'm excited about the
future it's foreshadowing.

I'm a visual and experiental learner though, so when I see statements like "Race conditions are
compile-time errors" I wonder what that looks like in the code, what kind of error message you get,
and how that differs from what I know in Ruby.

So I went out hunting for a trivial race condition in Ruby that I could port to Rust to see what would happen.
[Stack overflow to the rescue!](https://stackoverflow.com/questions/18574254/generating-a-race-condition-with-mri/18576777#18576777) Here's a slightly modified version of the Ruby code in that post that suited my purposes
better. Note that this is totally contrived and you would never actually want to have a bunch of
threads help you increment a number from 1 to 500:

{% highlight ruby %}
THREADS = 10
COUNT   = 50

$x = 1

THREADS.times.map { |t|
  Thread.new {
    COUNT.times { |c|
      a  = $x + 1
      sleep 0.000001
      puts "Thread #{t} wrote #{a}"
      $x =  a
    }
  }
}.each(&:join)

if $x != THREADS * COUNT + 1
  puts "Got $x = #{$x}."
  puts "Expected to get #{THREADS * COUNT + 1}."
else
  puts "Did not reproduce the issue."
end
{% endhighlight %}

This causes a race condition most of the time that I run it in Ruby 2.0-- I've got a global
variable that the threads read, then they sleep, then they increment, and in the time that one
thread sleeps, another thread reads and/or writes that global. Stuff happens all out of order.
Here's parts of the output from one of the times I ran it:

{% highlight ruby %}
Thread 0 wrote 2
Thread 0 wrote 3
...
Thread 1 wrote 78
Thread 3 wrote 29
...
Thread 3 wrote 59
Thread 4 wrote 29
...
Thread 4 wrote 78
Thread 3 wrote 60
...
Thread 0 wrote 50
Thread 0 wrote 51
Got $x = 51.
Expected to get 501

{% endhighlight %}

So what would it look like if I tried to do this in Rust, as faithfully as possible to the
contrived example? Here's my attempt, critiques welcome:

{% highlight rust %}

use std::io::timer::sleep;

fn main() {
  let threads = 10;
  let count   = 50;

  let mut x = 1u;
  for num in range(0u, threads) {
    spawn(proc() {
      for num in range(0u, count) {
        let a = x + 1;
        sleep(1);
        x = a; // error: cannot assign to immutable
               // captured outer variable in a proc `x`
      }
    });
  }

  println!("The result is {}", x);
}

{% endhighlight %}

Sure enough, if I try to compile this with rust 0.11.0, I get a compile error! Cool! I've marked
its location and text with a comment. This error says that the compiler captures `x` and makes it
immutable while in the `proc`. I can write to it all I want in `main` where it's declared as
mutable, but because I could create race conditions if I was allowed to write to it in a
spawned thread, Rust doesn't let me do that.

I tried to make a version of this program in Rust done The Right Way that would compile and run,
but a lot of what I'm seeing is basically saying "don't do that" to me (and this is a super
contrived example). I tried a little bit with mutexes, a little with futures, but they ended up
changing the logic happening so that one thread was locking the value for its whole existence, then
the next thread would lock it, etc. I never got it to compile, run, and illustrate everything I
wanted it to, so I'm leaving that as an exercise for the reader :) [This experiment in fork/join parallelism in Rust](http://www.reddit.com/r/rust/comments/29teem/an_experiment_in_forkjoin_parallelism_in_rust/)
looks relevant to my interests but I haven't digested it all yet.

Hope this helps someone else to understand the different choices that Ruby and Rust make in this
area! I'd love to hear what you think, how I could make this example better, or how you'd implement
this example to compile and run in Rust :) <3