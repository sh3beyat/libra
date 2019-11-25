FROM debian:buster AS toolchain

# To use http/https proxy while building, use:
# docker build --build-arg https_proxy=http://fwdproxy:8080 --build-arg http_proxy=http://fwdproxy:8080

RUN echo "deb http://deb.debian.org/debian buster-backports main" > /etc/apt/sources.list.d/backports.list \
    && apt-get update && apt-get install -y protobuf-compiler/buster cmake curl clang git \
    && apt-get clean && rm -r /var/lib/apt/lists/*

RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --default-toolchain none
ENV PATH "$PATH:/root/.cargo/bin"

WORKDIR /libra
COPY rust-toolchain /libra/rust-toolchain
RUN rustup install $(cat rust-toolchain)

FROM toolchain AS builder

COPY . /libra

RUN cargo build --release -p config-builder -p generate-keypair && cd target/release && rm -r build deps incremental
RUN strip target/release/libra-config target/release/generate-keypair

### Production Image ###
FROM debian:buster AS prod

RUN apt-get update && apt-get install gettext-base && apt-get clean && rm -r /var/lib/apt/lists/*

RUN mkdir -p /opt/libra/bin
COPY --from=builder /libra/target/release/libra-config /opt/libra/bin
COPY --from=builder /libra/target/release/generate-keypair /opt/libra/bin

ARG BUILD_DATE
ARG GIT_REV
ARG GIT_UPSTREAM

LABEL org.label-schema.schema-version="1.0"
LABEL org.label-schema.build-date=$BUILD_DATE
LABEL org.label-schema.vcs-ref=$GIT_REV
LABEL vcs-upstream=$GIT_UPSTREAM