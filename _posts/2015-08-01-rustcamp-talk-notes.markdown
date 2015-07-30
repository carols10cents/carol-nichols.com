---
layout: post
title:  "Rustcamp Talk Notes: Navigating the Open Seas"
date:   2015-08-01 15:09:18
categories:
---

I gave a talk called "Navigating the Open Seas" at [Rustcamp](http://rustcamp.com) today-- loosely
based on [my last blog post](/2015/05/10/rustc-discovery/) :) Here are links to everything I mentioned in the talk in case you'd
like to learn more about these parts of Rust's history!

- Introduction

  - [RFC 134](https://github.com/rust-lang/rfcs/pull/134) that proposed removing the `'` sigil from the lifetime syntax. I called out [this comment of Patrick's in particular](https://github.com/rust-lang/rfcs/pull/134#issuecomment-47054728) to illustrate the value of Rust's open development to future language designers who will be in this hypothetical state he proposed for every decision they'll be making.

- Exploration #1 - into the rust-dev mailing list

  - [The rust-dev mailing list](https://mail.mozilla.org/pipermail/rust-dev/), active from July 2010-Jan 2015. Replaced by the [users](https://users.rust-lang.org/) and [internals](https://internals.rust-lang.org/) discourse installations.

  - [The rust-dev mailing list on nabble](http://rust-dev.1092773.n5.nabble.com/), which I prefer because of the search available there.

  - [Oct 28, 2011 Renaming "tag" and "log_err" mailing list thread](http://rust-dev.1092773.n5.nabble.com/Renaming-quot-tag-quot-and-quot-log-err-quot-td991.html) - the 21 message long mailing list thread that announces that the std library now has print/println, wonders if Rust should have a prelude, and discusses the renaming of "tag", which is what ends up as "enum" today.

- Exploration #2 - where did the lifetime elision rules come from?

  - [The lifetime elision rules in TRPL](https://doc.rust-lang.org/stable/book/lifetimes.html#lifetime-elision)

  - [The markdown source of the lifetime page in the book](https://github.com/rust-lang/rust/blob/master/src/doc/trpl/lifetimes.md)

  - [Github's rendering of `git log src/doc/trpl/lifetimes.md`](https://github.com/rust-lang/rust/commits/master/src/doc/trpl/lifetimes.md)

  - [Github's rendering of `git blame src/doc/trpl/lifetimes.md`](https://github.com/rust-lang/rust/blame/master/src/doc/trpl/lifetimes.md)

  - [Github's rendering of `git blame -L 269 src/doc/trpl/lifetimes.md`](https://github.com/rust-lang/rust/blame/master/src/doc/trpl/lifetimes.md#L269)

  - [Github's rendering of `git show ab3cb8c5`](https://github.com/rust-lang/rust/commit/ab3cb8c5ae3b9fe86faa1dfa9402145788a005f5)

  - [Github's rendering of `git blame ab3cb8c5~ src/doc/trpl/ownership.md`](https://github.com/rust-lang/rust/commit/ab3cb8c5ae3b9fe86faa1dfa9402145788a005f5)

  - [Github's rendering of `git show a56e7aee`](https://github.com/rust-lang/rust/commit/a56e7aee81733485d6edd415ab383347232e3c36), where the lifetime elision section was added to the docs

  - [Issue #19662 for adding documentation about all forms of lifetime elision](https://github.com/rust-lang/rust/issues/19662)

  - [The rules as proposed in RFC 141](https://github.com/rust-lang/rfcs/blob/master/text/0141-lifetime-elision.md#the-rules)

  - [RFC 141 PR discussion](https://github.com/rust-lang/rfcs/pull/141)

- Exploration #3 - What happened to iface?

  - [A sample message on the rust-dev mailing list that mentions `iface`, that doesn't exist today](http://rust-dev.1092773.n5.nabble.com/RFC-Removing-as-a-WIP-tp1910.html)

  - [Commit eb834fdb](https://github.com/rust-lang/rust/commit/eb834fdb) that stopped parsing `iface`

  - [Lindsey Kuper's intern presentation video](https://air.mozilla.org/rust-typeclasses/) that talked about her work unifying `trait` and `iface`

  - [The rust-wiki-backup repository](https://github.com/rust-lang/rust-wiki-backup)

  - [The proposal for unifying traits and interfaces](https://github.com/rust-lang/rust-wiki-backup/blob/95602c07a543404117685439c58586af756c918f/Proposal-for-unifying-traits-and-interfaces.md) in the wiki history

- Conclusion

  - [Where the IRC logs before April 2013 would be if irclog.gr hadn't loaded the data with ajax](https://web.archive.org/web/20130530235831/http://irclog.gr/#browse/irc.mozilla.org/rust)

