# Experimental Reed-Solomon in Rust

## Background

This evolved out of an exercise in [A Programmer's Introduction to
Mathematics](https://www.amazon.co.uk/Programmers-Introduction-Mathematics-Dr-Jeremy/dp/1727125452)
from the chapter on Polynomials. The exercise had to do with [Shamir's secret
sharing algorithm](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing), but
I was familiar enough with Reed-Solomon to realize they were probably based on
the same concepts. I was looking for a programming project while reading that,
so I decided to make a library to do RS encoding, and [later actually did the
Shamir's secret sharing portion](https://github.com/atomictom/shamir).
Additionally, this was a chance to learn more Rust.

Currently the code is quite messy since I used a very slow method of computing
RS at first (Lagrangian interpolation of polynomials) and only later added in a
faster way (using Vandermonde matrices). Because it was a learning exercise, I
kept both approaches. Additionally, my inexperience with Rust meant that some of
my interfaces were subpar or I had to make compromises to make things compile.
Lastly, the code has been written in fits and bursts often on airplanes or when
tired so I wouldn't say it's my highest quality code!

## Functionality

Right now it's very half baked. There is a pseudo-library for doing RS encoding,
but it's not very easy to use. There are some functions for doing a limited
Shamir setup on top of the RS library, though this functionality has now been
split off into [another respository](https://github.com/atomictom/shamir).

## Performance

The original implementation (Lagrangian interpolation) did about 1MiB/s (quite
bad!). It currently does around 40MiB/s based on benchmarks (single threaded on
a Ryzen 5 3600), which is still fairly abysmal. I suspect a purely code-based
approach should be able to do around 400MiB/s based on other work I've seen. To
do so, I would likely need to use memory more smartly to avoid copies and
improve cache locality, and use CPU intrinsics (e.g.  SIMD and CLMUL).

There are also ways using Cauchy matrices and a technique that turns
multiplication into just a few XORs and ANDs that can improve performance
described in [An XOR based erasure resilient coding
scheme](https://www.cs.utexas.edu/~diz/Sub%20Websites/Research/An_XOR_based_erasure_resilient_coding_scheme.pdf).
Going further, [Optimizing Cauchy Reed-Solomon Codes for Fault-Tolerant Network
Storage
Applications](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.140.2267&rep=rep1&type=pdf)
describes how to find an optimal Cauchy matrix which reduces the number of XORs
further still.

## Future Work

At some point I'd like to refactor the interface to be simpler to use and I'd
like to clean up the code to remove some of the cruft that came from learning
RS.

As mentioned above, it'd be cool to improve the performance to try to get to
1GiB/s.

## Demo

```
$ cargo run
...

-- RS Encoding --
Encoding: "Test string"
Bytes: [84, 101, 115, 116, 32, 115, 116, 114, 105, 110, 103]
Length: 11
Encoding: Encoding { data_chunks: 6, code_chunks: 4 }
Codes: [[84, 101, 115, 116, 32, 115, 36, 1, 160, 213], [116, 114, 105, 110, 103, 0, 2, 42, 239, 192]]

-- RS Decoding --
Destroying data in column 0
Destroying data in column 1
Destroying data in column 8
Destroying data in column 9
Damaged stream: RSStream { length: 11, encoding: Encoding { data_chunks: 6, code_chunks: 4 }, codes: [[0, 0, 115, 116, 32, 115, 36, 1, 0, 0], [0, 0, 105, 110, 103, 0, 2, 42, 0, 0]], valid: [false, false, true, true, true, true, true, true, false, false] }
Recovered string: "Test string"

-- Failed RS Decoding --
Destroying data in column 2
Damaged stream: RSStream { length: 11, encoding: Encoding { data_chunks: 6, code_chunks: 4 }, codes: [[0, 0, 0, 116, 32, 115, 36, 1, 0, 0], [0, 0, 0, 110, 103, 0, 2, 42, 0, 0]], valid: [false, false, true, true, true, true, true, true, false, false] }
Recovered string: "Got utf8 parsing error: FromUtf8Error { bytes: [25, 163, 0, 116, 32, 115, 232, 126, 0, 110, 103], error: Utf8Error { valid_up_to: 1, error_len: Some(1) } }"

-- Shamiring it up --
Shards: 5, required: 3
Password: cult date rerun flint hump spree scoff front angle bash
Shard 1: rack syrup cola twine spew stunt tiger spree salon pull
Shard 2: given moan elbow flip dance cheer panty decal lash fling
Shard 3: ebay cult panty quota flint scowl angel work delay bash
Shard 4: fifth pecan stud legal stunt ozone shun pep ditch from
Shard 5: fifty panic tank diner rake remix olive clap motor rerun

-- Unshamiring it down --
Input shards:
        Shard 0: None
        Shard 1: Some("rack syrup cola twine spew stunt tiger spree salon pull")
        Shard 2: None
        Shard 3: Some("ebay cult panty quota flint scowl angel work delay bash")
        Shard 4: None
        Shard 5: Some("fifty panic tank diner rake remix olive clap motor rerun")
Valid: [false, true, false, true, false, true]
Length: 10
Encoding: Encoding { data_chunks: 3, code_chunks: 3 }
Shards: 6, required: 3
Password: cult date rerun flint hump spree scoff front angle bash
```
