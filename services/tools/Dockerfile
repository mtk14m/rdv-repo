FROM ubuntu:24.04

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        procps \
        sysstat \
        stress-ng \
        iproute2 \
        dnsutils \
        curl \
        tcpdump \
        ca-certificates \
    && rm -rf /var/lib/apt/lists/*

CMD ["bash"]
