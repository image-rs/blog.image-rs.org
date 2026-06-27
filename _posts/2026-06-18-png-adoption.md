---
layout: post
author: Shnatsel
title: "Rust PNG crate gets even faster, used by GNOME and Chromium"
---

Rust [png](https://crates.io/crates/png) crate, also known as [image-png](https://github.com/image-rs/image-png/), implements PNG encoding and decoding in safe Rust. It is compliant with the [third edition of the PNG specification](https://www.w3.org/TR/png-3/), including [APNG](https://en.wikipedia.org/wiki/APNG) support.

A year ago it was already the fastest PNG decoder in the world. Since then it has gotten even faster.

The adoption milestones over the past year are just as impressive as the technical ones.

## Adoption in Chromium

image-png is used as the default PNG implementation in Chromium [since M139](https://chromium-review.googlesource.com/c/chromium/src/+/6085801) (August 2025) across all platforms, both on desktop and on mobile.

This required meeting a _remarkably_ high bar in terms of features, performance, correctness, compatibility and security. It would only ship if there were no regressions in _any_ of those categories!

Outperforming every other off-the-shelf PNG implementation was not enough: Chromium has its own fork of zlib optimized specifically for Chromium's use cases. More work was needed to match its performance, both due to differences in workloads (e.g. lots of very small images instead of a single large one) and in the hardware (e.g. low-end ARM chips in older Android phones).

Compatibility and correctness required surprisingly little work. The library is extensively tested, and we've already been regularly testing the code on tens of thousands of real-world images to make sure we can handle even non-compliant images that other implementations just so happen to decode. We only had to adjust the error reporting in case of corrupt chunk and/or invalid chunk orders to match libpng, and fix a bug in streaming decoding mode with unusual buffer sizes. [OSS-fuzz](https://github.com/google/oss-fuzz) continuously validates that the input is always the same regardless of buffer sizes to prevent such bugs from reoccurring.

In terms of features, we had to implement displaying intermediate decoding states for partially downloaded PNGs and add support for `mDCV` and `cLLI` chunks that were being standardized while we were working on this, but with that we're at feature parity with Chromium's memory-unsafe PNG code.

The switch to image-png for billions of users across all of Chromium's supported platforms is the ultimate vote of confidence in the code, and we hope this will encourage further adoption of image-png and other memory-safe image processing across the industry.

## Adoption in GNOME

In 2023, [GNOME 45](https://release.gnome.org/45/) was released with a new default Image Viewer. This app used GNOME's new image loading library [glycin](https://gitlab.gnome.org/GNOME/glycin/), which uses [image-rs](https://github.com/image-rs/image) for loading most of the supported image formats, including image-png.

[GNOME 49](https://release.gnome.org/49/developers/) was released in September 2025. In this release GNOME's legacy image-loading library, [gdk-pixbuf](https://gitlab.gnome.org/GNOME/gdk-pixbuf), was converted to build as a thin wrapper around glycin on Linux. Multiple distributions (Fedora, Arch, OpenSuse Tumbleweed, Ubuntu, Debian testing) have already adopted the change of gdk-pixbuf to a simple shim. This switches most GNOME apps to using glycin and with that image-rs by default.

Improved performance of PNG handling was not a major selling point for GNOME. With large PNG images being relatively rare, libpng was already fast enough for most uses. The primary reasons for this migration were **memory safety** and **new features.**

All modern web browsers [support](https://caniuse.com/apng) animated PNG, but it isn't implemented in upstream libpng, so web browsers have to carry patches on top of the original code. Since most Linux distributions do not ship a patched libpng, GNOME apps could not support APNG either.

image-png has supported APNG since 2020, so migrating to it enabled APNG support first in the image viewer, and now in nearly all other GNOME apps.

## Benchmarks

Due to the differences between implementations, one can always find a single image for which a given implementation performs especially well or especially poorly. 

To avoid bias, we compare performance across all 2848 images from the independent, third-party [QOI benchmark corpus](https://qoiformat.org/benchmark/) that contains many different kinds of images. We also calculate and report the geometric mean to avoid skewing the results due to outliers. You can find the benchmarking harness used for the measurements [here](https://github.com/fintelia/corpus-bench/tree/png-0.18.0).

### Ryzen 9 7950X

```
image-png:          410.8 MP/s (average)   339.9 MP/s (geomean)
zune-png:           394.6 MP/s (average)   313.2 MP/s (geomean)
wuffs-png:          381.4 MP/s (average)   290.9 MP/s (geomean)
libpng:             218.5 MP/s (average)   180.4 MP/s (geomean)
stb_image:          240.8 MP/s (average)   174.9 MP/s (geomean)
spng:               298.7 MP/s (average)   233.1 MP/s (geomean)
```

### Apple M4

```
image-png:          387.7 MP/s (average)   310.5 MP/s (geomean)
zune-png:           327.6 MP/s (average)   253.7 MP/s (geomean)
wuffs-png:          317.5 MP/s (average)   246.7 MP/s (geomean)
libpng:             207.8 MP/s (average)   177.6 MP/s (geomean)
stb_image:          228.3 MP/s (average)   163.4 MP/s (geomean)
spng:               209.4 MP/s (average)   159.1 MP/s (geomean)
```

_all numbers are in megapixels per second, higher is better_

As you can see, _all three_ memory-safe PNG implementations ([image-png](https://github.com/image-rs/image-png/), [zune-png](https://crates.io/crates/zune-png), [wuffs-png](https://github.com/google/wuffs/)) enjoy a commanding lead over the C implementations ([libpng](http://www.libpng.org/), [stb_image](https://github.com/nothings/stb), [spng](https://libspng.org/)). This goes to show you don't have to pick between memory safety and performance - you can have both!

Note that the C implementations are built with [zlib-ng](https://github.com/zlib-ng/zlib-ng) to give them the best possible chance. As of this writing zlib-ng is not widely deployed, so for most users the performance gap between memory-safe and C implementations is _even wider._

## How is this possible?

PNG format is just [DEFLATE](https://en.wikipedia.org/wiki/Deflate) compression (same as in `gzip`) plus [PNG-specific filters](https://en.wikipedia.org/wiki/PNG#Filtering) that try to make image data easier for DEFLATE to compress.

The most notable performance gains compared to the previous release are:

1. [Performing unfiltering in-place](https://github.com/image-rs/image-png/pull/590) instead of copying data to another buffer first. This requires [extra care](https://github.com/image-rs/image-png/pull/664) for partial data.
1. Reducing internal buffer sizes in [several](https://github.com/image-rs/image-png/pull/588) [places](https://github.com/image-rs/image-png/pull/595) keeps data in faster cache tiers between decompression and unfiltering.
1. [Interlacing](https://en.wikipedia.org/wiki/PNG#Interlacing) was not heavily exercised before adoption by Chromium, and had plenty of low-hanging fruit for optimization.
1. We've benefited from ecosystem improvements: [simd_adler32](https://crates.io/crates/simd-adler32) and [crc32fast](https://crates.io/crates/crc32fast) crates we use for checksum calculation now utilize AVX-512 and NEON intrinsics after Rust stabilized them.

There are other changes that don't have much of an impact on their own, but add up when taken together.

## Can we go even faster?

Despite these already impressive results, it should be possible to push performance even further.

First, we currently rely on automatic vectorization by the compiler as opposed to writing explicit SIMD code. Chromium engineers have contributed explicit SIMD codepaths using [`std::simd`](https://doc.rust-lang.org/stable/std/simd/index.html). This improves performance on x86, but turns out to be a mixed bag on ARM, where it tends to improve performance on the efficiency cores but regress on high-performance cores of that same chip. This is not enabled by default and is not included in the above benchmarks, but you can already try it today by enabling the "unstable" feature.

Second, unlike the C libraries, we do not use any platform-specific intrinsics in our code. Previously using SIMD intrinsics would have required `unsafe` code, but since rust 1.88 this is no longer the case. Experiments indicate that this may unlock further performance gains for some of the filters.

Third, on x86 detecting available CPU features at runtime will be beneficial for at least some images. Right now image-png is confined to the baseline SSE2 instruction set. We've measured that SSE4.2 would be beneficial at least to Paeth unfiltering.

## Encoding performance

Encoding performance is more difficult to measure. Unlike with decoding, where the expected result is known, different encoding implementations may make different trade-offs. For example, is it better or worse to run a little faster but produce larger images? If so, by how much? Compressing which data patterns well should be prioritized?

A clear advantage that we already have today is the ultra-fast compression mode inspired by [fpnge](https://github.com/veluca93/fpnge), which produces lightly compressed images an order of magnitude faster than traditional encoders. But that trade-off between speed and compression ratio is not always appropriate.

Our default DEFLATE compression backend is [miniz_oxide](https://crates.io/crates/miniz_oxide). It provides reasonable compression performance without any `unsafe` code. Combined with our optimized filter implementations, it provides performance roughly on par with Chromium's fork of zlib - somewhat slower or faster depending on the hardware.

You can configure image-png to use zlib-rs as the DEFLATE compression backend. This provides [outstanding performance](https://trifectatech.org/blog/zlib-rs-is-faster-than-c/) at the cost of some `unsafe` code: the constraint of being a drop-in replacement for a C library necessitates some `unsafe` parts.

We want to provide world-leading performance without compromises: performance, guaranteed safety and high compression ratio all at once. We have already achieved this for decoding, in part thanks to our [custom DEFLATE implementation](https://crates.io/crates/fdeflate). We are now working on extending it for compression as well, and the early tests are very promising!

## Acknowledgements

We appreciate Chromium engineers engaging with the project via discussions, reporting issues and contributing fixes and optimizations. Even the changes that didn't end up getting merged were invaluable exploratory work that is crucial for pushing the boundaries of performance. Engaging with us early, and always communicating clearly and transparently was very welcome.

Some of the maintainers of image-png were only able to dedicate time to collaborating with Chromium engineers and meeting GNOME's needs thanks to an investment by [Sovereign Tech Agency](https://www.sovereign.tech). We hope that putting memory-safe PNG decoding into the hands of billions of people is a good return on their investment!
