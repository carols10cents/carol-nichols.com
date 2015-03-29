---
layout: post
title:  "TDD Example in Rust"
date:   2015-03-28 19:31:00
categories:
---

This is an example of how I worked through the [Prime Factors Kata](http://www.butunclebob.com/ArticleS.UncleBob.ThePrimeFactorsKata) using Test Driven Development in Rust (specifically rustc 1.0.0-dev 928e2e239 2015-03-25). I'm assuming you've read [The Rust Programming Language Book](http://doc.rust-lang.org/book/) through the Testing chapter, so I won't be explaining much of the Rust syntax.

The kata, as I decided to interpret it, is:

> Write a function that takes an integer greater than 1 and returns a vector of its prime factors, which are the prime numbers that divide that integer exactly, in ascending order. For example, the prime factors of 12 are 2, 2, 3.

If you'd like to try it yourself without spoilers, here is where you should stop reading and pick up again when you're ready!

-----

If you'd like to see the full code at each step that I describe below, [take a look at the commits in this repo](https://github.com/carols10cents/prime_factors/commits/master). Additionally, the output from `cargo test` at that commit is in each commit message.

So! I generated a new library using `cargo new`, then started with a failing test:

```
#[test]
fn prime_factors_of_two() {
    assert_eq!(prime_factors(2), [2]);
}
```

And, if I run `cargo test`, it fails to compile:

```
$ cargo test
   Compiling prime_factors v0.0.1
(file:///Users/carolnichols/Rust/prime_factors)
src/lib.rs:3:16: 3:29 error: unresolved name `prime_factors`
src/lib.rs:3     assert_eq!(prime_factors(2), [2]);
                            ^~~~~~~~~~~~~
<std macros>:1:1: 9:39 note: in expansion of assert_eq!
src/lib.rs:3:5: 3:39 note: expansion site
error: aborting due to previous error
Build failed, waiting for other jobs to finish...
Could not compile `prime_factors`.
```

In my mind, TDD in Rust has two "red" states-- failing to compile and failing tests. I'm in the first kind right now. I don't necessarily hit both states every time, nor do they always happen in the same order. I feel like I've even stopped distinguishing the two in my mind-- it doesn't really matter whether it's the compiler or the tests that are helping me make my code better. The next step is the same: read the first problem and do the simplest thing you can do to respond to that problem.

The first message here is "error: unresolved name `prime_factors`", and the simplest thing I can do is define that function without any arguments and with the unit type return value:

```
#[allow(dead_code)]
fn prime_factors() {
}
```

I also decided to add the attribute to override the dead code lint check since I'm not going to have any non-test code calling this function. Silencing that warning will make the test output clearer. So now I get:

```
src/lib.rs:7:16: 7:32 error: this function takes 0 parameters but 1
parameter was supplied [E0061]
src/lib.rs:7     assert_eq!(prime_factors(2), [2]);
                            ^~~~~~~~~~~~~~~~
```

Easy enough, I do indeed want the function to take 1 integer parameter:

```
fn prime_factors(num: i64) {
}
```

And now I get a warning and a different error:

```
src/lib.rs:2:18: 2:21 warning: unused variable: `num`,
src/lib.rs:2 fn prime_factors(num: i64) {
                              ^~~
<std macros>:5:24: 5:35 error: mismatched types:
 expected `()`,
    found `[_; 1]`
(expected (),
    found array of 1 elements) [E0308]
<std macros>:5 if ! ( ( * left_val == * right_val ) && ( * right_val ==
* left_val ) ) {
                                      ^~~~~~~~~~~
<std macros>:1:1: 9:39 note: in expansion of assert_eq!
```

I'm going to let this warning continue to happen for now because I *should* be using this variable eventually, but I'm not going to address it this step. I'm more concerned with the first error that says in the test, the things I'm trying to assert are equal aren't the same type. One, the left side, is the unit type, and the right side is an array of one element.

So I'm going to fix this error first by just specifying the return type:

```
fn prime_factors(num: i64) -> Vec<i64> {
}
```

which gets me:

```
src/lib.rs:1:1: 2:2 error: not all control paths return a value [E0269]
src/lib.rs:1 fn prime_factors(num: i64) -> Vec<i64> {
```

"Not all"? Try "none", rustc ;) Anyway, let's get this compiling and the test passing with the simplest implementation:

```
fn prime_factors(num: i64) -> Vec<i64> {
    vec![2]
}
```

And I'm green! Woo! There isn't anything to refactor here, so onto another failing test:

```
#[test]
fn prime_factors_of_three() {
    assert_eq!(prime_factors(3), [3]);
}
```

which compiles, but the test fails:

```
---- prime_factors_of_three stdout ----
	thread 'prime_factors_of_three' panicked at 'assertion failed:
`(left == right) && (right == left)` (left: `[2]`, right: `[3]`)',
src/lib.rs:13
```

Yep, as I expected. So... the next simplest thing to get both these tests to pass is returning the argument as a vector, which should work for any prime number:

```
fn prime_factors(num: i64) -> Vec<i64> {
    vec![num]
}
```

This compiles, passes the tests, *and* gets rid of those warnings about not using `num`. Onward!

```
#[test]
fn prime_factors_of_four() {
    assert_eq!(prime_factors(4), [2, 2]);
}
```

Aha, something a bit more interesting since 4 isn't prime. This fails as expected:

```
---- prime_factors_of_four stdout ----
	thread 'prime_factors_of_four' panicked at 'assertion failed:
`(left == right) && (right == left)` (left: `[4]`, right: `[2, 2]`)',
src/lib.rs:18
```

I decided to go with a recursive solution here, but I didn't really like it at this point. It also took me a while to get the syntax for "return the combination of two vectors as a new vector without any variables" to work right, but it turns out `+` with a reference to another vec works the way I want:

```
fn prime_factors(num: i64) -> Vec<i64> {
    if num % 2 == 0 && num > 2 {
        vec![2] + &prime_factors(num / 2)
    } else {
        vec![num]
    }
}
```

There's a lot of conditionals and a lot of "2"s duplicated around. I could have refactored some of those at this point, but I didn't. This is where experience and judgement calls come in... I knew this wasn't the general solution, and I knew I didn't really like my implementation anyway. Thus, I knew there was going to be a lot of change coming up, and refactoring at this point would potentially do more harm than good and just have to be undone later. But don't let this become an excuse to not refactor!

If you look at the repo, you'll see I went ahead and added tests for 5, 6, 7, and 8 that all passed without changing the code. In [Uncle Bob's solution]((http://www.butunclebob.com/ArticleS.UncleBob.ThePrimeFactorsKata), he skips 5 and 7, ostensibly because he is confident enough in his coverage of prime numbers with tests for 2 and 3 that tests for more primes are no longer interesting. Again, judgement call. It's also totally ok to write a whole bunch of tests while you're test driving, but once you understand the domain better, you delete some tests that are covering the same cases before finishing so that you have fewer to wait for to run and fewer that will all break at once if something changes.

And actually, before I ran my test for 6 for the first time, I expected it to fail since 6 isn't made up of only factors of 2, but my solution was more general than I thought it was!

But 9 is the next interesting test case that fails because it isn't prime and 2 is not any of its prime factors:

```
---- prime_factors_of_nine stdout ----
	thread 'prime_factors_of_nine' panicked at 'assertion failed:
`(left == right) && (right == left)` (left: `[9]`, right: `[3, 3]`)',
src/lib.rs:47
```

So. Because I was feeling a little meh about the recursive solution, I decided to switch to an iterative approach. This means I made 3 things mutable: the argument, a vector to hold the result, and a temporary variable that iterates over potential factors. Now, Rust tries to discourage you from making things mutable by making all variables immutable by default, but that doesn't mean it's *bad* or *wrong* necessarily. It's just a bit wordier.

```
fn prime_factors(mut num: i64) -> Vec<i64> {
    let mut result = vec![];
    while num != 1 {
        let mut i = 2;
        while i <= num {
            if num % i == 0 {
                result.push(i);
                num = num / i;
                break;
            } else {
                i += 1;
            }
        }
    }
    result
}
```

Now I'm starting to feel like I have a general solution. 9 passes; in [the repo](https://github.com/carols10cents/prime_factors/commits/master) you'll see I added two more tests-- one for a larger prime (101 => [101]) and one for a larger non-prime (48 => [2, 2, 2, 2, 3]). Again, these aren't really necessary since they aren't covering any cases that aren't already covered (and they do pass right away). But in this case, they don't add much time to the test run and they could have caught problems that might not show up with smaller numbers. It all depends.

Refactoring time! I wanted to see if I could get rid of the mutable `i` variable by switching the `while` loop to be a `for` loop over a `range`, since that's all it's doing anyway.

```
fn prime_factors(mut num: i64) -> Vec<i64> {
    let mut result = vec![];
    while num != 1 {
        for i in 2..num + 1 {
            if num % i == 0 {
                result.push(i);
                num = num / i;
                break;
            }
        }
    }
    result
}
```

Still passes tests, looks a little better. I still wasn't happy about it though, since I read in [Uncle Bob's description](http://www.butunclebob.com/ArticleS.UncleBob.ThePrimeFactorsKata) that he was able to solve the kata in 3 lines of code. I couldn't spot anything I could change, so at this point I gave up and looked at his solutions in his slides. Now, he did it in Java and his solution was:

```
for (int candidate = 2; n > 1; candidate++)
    for (; n%candidate == 0; n/=candidate)
        primes.add(candidate);

```

And now it makes sense. This feels a bit too clever, though, with the for loop conditions not matching the iterator index. It's neat code golf though. I couldn't get it down quite this small since Rust doesn't have this for loop syntax, but this is probably the closest translation and is better than what I ended up with in that it's not going back out to the outer loop if we can continue to divide by the candidate factor over and over:

```
let mut result = vec![];
let mut i = 2;
while num > 1 {
    while num % i == 0 {
        result.push(i);
        num /= i;
    }
    i += 1;
}
result
```

But is this the most idiomatic Rust solution? It didn't really feel like it, with all those `mut`s still in there. So at this point I decided to give the recursive solution a try again:

```
fn prime_factors(num: i64) -> Vec<i64> {
    for i in 2..num {
        if num % i == 0 {
            return vec![i] + &prime_factors(num / i)
        }
        i += 1;
    }
    vec![num]
}
```

This feels better to me, after having explored the iterative solutions. What do you think? Again, there's no one right answer :)

I've been meaning to learn how to, and write a post on, how to benchmark Rust and analyze the runtime and memory usage, perhaps I'll use these different implementations as material for that :)
