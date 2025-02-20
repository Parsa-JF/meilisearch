FROM rustlang/rust:nightly as builder

RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y build-essential clang-11 llvm-11-tools cmake clang curl binutils-dev libbfd-dev libunwind8-dev libblocksruntime-dev pkg-config libudev-dev libssl-dev && \
    ln -s /usr/bin/llvm-config-11 /usr/bin/llvm-config

RUN rustup component add rust-src
RUN cargo install -f cargo-fuzz
RUN cargo install -f afl

ADD . /meilisearch
WORKDIR /meilisearch

RUN cd ./filter-parser/fuzz/ && cargo +nightly fuzz build
RUN cd ./flatten-serde-json/fuzz/ && cargo +nightly fuzz build
RUN cd ./json-depth-checker/fuzz/ && cargo +nightly fuzz build


# Move Fuzzers to a clean image
FROM ubuntu:20.04

COPY --from=builder meilisearch/filter-parser/fuzz/target/x86_64-unknown-linux-gnu/release/parse /
COPY --from=builder meilisearch/flatten-serde-json/fuzz/target/x86_64-unknown-linux-gnu/release/flatten /
COPY --from=builder meilisearch/json-depth-checker/fuzz/target/x86_64-unknown-linux-gnu/release/depth /