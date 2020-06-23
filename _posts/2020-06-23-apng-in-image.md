---
layout: post
author: HeroicKatora
title: "View APNG with emulsion and image"
---

[Emulsion] is a fast and minimalistic image viewer that runs on modern graphic
interfaces and uses `image` for file loading. Since the release of `4.0` it
also supports viewing animated PNGs (APNG). This is enabled by the new `Apng`
interface added in release `0.23.6` of `image`. You can find more information
and the newest releases here:

| Website | Crates.io | Command |
| [![emulsion-icon][emulsion-icon]][emulsion-website] | [![crates.io](https://img.shields.io/crates/v/emulsion.svg)](https://crates.io/crates/emulsion) | `cargo install emulsion` |
| <a href="https://github.com/image-rs/image"><img src="https://github.com/image-rs.png?size=20" width="48"/></a> | [![crates.io](https://img.shields.io/crates/v/image.svg)](https://crates.io/crates/image) | `image = "0.23.6"` |

Another piece of news is that `png` has now been fuzzed on 32-bit
architectures, specifically the `i686-unknown-linux-gnu` target. This revealed
a few nasty edge cases of integer overflows subverting size and limit checks
but nothing too spectacular or unsound. These are now dealt with more
rigorously by doing checked arithmetic in advance with the full precision of
`u64` as well as by using Rust's `TryFrom` trait to convert the dimensions from
the untrusted file input to the platform native size type `usize`, which
explicitly returns errors in case of overflows.

[Emulsion]: https://crates.io/crates/emulsion
[emulsion-website]: https://arturkovacs.github.io/emulsion-website/
[emulsion-icon]: https://raw.githubusercontent.com/ArturKovacs/emulsion/master/resource/emulsion48.png
[image]: https://crates.io/crates/image
[image-website]: https://github.com/image-rs/image
