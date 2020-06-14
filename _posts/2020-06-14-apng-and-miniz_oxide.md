---
layout: post
author: HeroicKatora
title: "APNG support and `miniz_oxide`"
---

The release of `png` version `0.16.5` was one of the most personally exciting
ones recently. It contains APNG support and large decoding improvements that
will benefit everyone, both of which have been anticipated and planned for
quite a while.

Firstly, the release marks the begin of actual, official [APNG] decoding
support! While the basic corresponding chunks types were already known and
being decoded for some time, you will now be able to retrieve the layers and
validated control information of all animation frames with the same interface
as the initial decoding. (Note that compositing of the complete frame needs to
be performed by the caller. The next release of `image` will have these
compositing capabilities built-in with the same interface as its `gif`
decoder.) In the wake of these changes we fixed a few bugs. It is no longer
required that the APNG data is valid if only the initial still image is
requested.

A big Thank You to everyone who was involved, with code, bug reports,
reproduction cases, testing, and sanity checks. It's always a pleasure if bug
reports come with insightful debugging already finished.

The encoding support will arrive at a later date, an implementation draft with
corresponding PR already exists but needs more polishing. Meanwhile encoding of
palette-based pixel data has been integrated and is being released with this
version.

The second great news is that the zlib/deflate decoder has been switched to
`miniz_oxide`. It improves upon the previous library in almost every way: It is
safer–forbidding `unsafe` code; it is faster–with typically between 1.3× to 3×
decoding throughput; and it has a more detailed interface–allowing more control
over buffer allocations and limits.

## Want to help?

The community member {% include gitavatar.html author="Shnatsel" %}, who
suggested going for `miniz_oxide` initially, has recently compiled a list of
important issues that could be tackled by new contributors across all of
`image-rs`:

* <https://github.com/image-rs/image-png/issues/206>
* <https://github.com/image-rs/jpeg-decoder/issues/96>
* <https://github.com/image-rs/image/issues/1238>
* <https://github.com/image-rs/image/issues/1165>
* <https://github.com/image-rs/image/issues/862>

Beyond these we are always grateful for further performance improvements, the
story is nowhere near finished. There are a number of buffer copies and other
inefficiencies that add up. Everything is tested with some internal and
integration benchmarks that You can target to improve.

[APNG]: https://wiki.mozilla.org/APNG_Specification
