# Fuzzing

To fuzz this crate:

    cargo install cargo-fuzz
    RUSTFLAGS="-Cpanic=unwind" cargo +nightly fuzzer run --sanitizer=leak decode
    ^C

Selecting a nightly toolchain with `+nightly` is a feature of
[rustup](https://rustup.rs/).

Leak sanitizer is used here instead of the default address sanitizer, because
address sanitizer will report crashes that we are not interested in, so we
select a different sanitizer. Thread sanitizer would work as well.

Just fuzzing without seeding the corpus will likely not discover anything. The
number of program paths covered (the `cov` value in the output) will be small.
To seed the corpus, copy a few raw files into `fuzz/corpus/decode`. Now running
`cargo +nightly fuzzer run decode` should quickly discover more paths.

In debug mode the fuzzing target it quite slow. Because of the low throughput,
fuzzing will not be effective. To get better throughput, fuzz in release mode
with `--release`. To detect hangs, reduce the timeout from the default 1200
seconds to 2 seconds:

    RUSTFLAGS="-Cpanic=unwind" cargo +nightly fuzzer run --release --sanitizer=leak decode -- -max_len=1024 -timeout=2

It can be useful to have small inputs to reproduce a hang. To do so, limit the
input length:

    RUSTFLAGS="-Cpanic=unwind" cargo +nightly fuzzer run --sanitizer=leak decode -- -max_len=1024

At some point the fuzzer might not discover new program paths any more, because
a file that is too small cannot trigger all paths. Increase `max_len` to
continue searching for more complex examples.