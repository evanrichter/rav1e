# Build Stage
FROM ubuntu:22.04 as builder

## Dependencies
RUN apt update && apt install -y curl pkg-config cmake build-essential libdav1d-dev nasm && \
    curl https://sh.rustup.rs -sSf | sh -s -- -y --default-toolchain nightly && \
    ${HOME}/.cargo/bin/cargo install -f cargo-fuzz

## Add source code to the build stage.
ADD . /src
WORKDIR /src

RUN echo building instrumented harnesses && \
    # ubuntu doesn't have a new enough version of libdav1d for some harnesses
    sed -i 's/^features = \[\"decode_test_dav1d\"\]//' fuzz/Cargo.toml && \
    bash -c "pushd fuzz && ${HOME}/.cargo/bin/cargo +nightly -Z sparse-registry fuzz build encode && popd" && \
    bash -c "pushd fuzz && ${HOME}/.cargo/bin/cargo +nightly -Z sparse-registry fuzz build construct_context && popd" && \
    mv fuzz/target/x86_64-unknown-linux-gnu/release/encode /encode && \
    mv fuzz/target/x86_64-unknown-linux-gnu/release/construct_context /construct_context && \
    echo done

RUN echo building non-instrumented harnesses && \
    export RUSTFLAGS="--cfg fuzzing -Clink-dead-code -Cdebug-assertions -C codegen-units=1" && \
    bash -c "pushd fuzz && ${HOME}/.cargo/bin/cargo +nightly -Z sparse-registry build --bin encode --release && popd" && \
    bash -c "pushd fuzz && ${HOME}/.cargo/bin/cargo +nightly -Z sparse-registry build --bin construct_context --release && popd" && \
    mv fuzz/target/release/encode /encode_no_inst && \
    mv fuzz/target/release/construct_context /construct_context_no_inst && \
    echo done

# Package Stage
FROM ubuntu:22.04

COPY --from=builder /encode /encode_no_inst /
COPY --from=builder /construct_context /construct_context_no_inst /
