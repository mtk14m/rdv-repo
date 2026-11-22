FROM ubuntu:24.04

RUN apt-get update \
    && apt-get install --no-install-recommends -y \
        procps \
        sysstat \
        stress-ng \
    && rm -rf /var/lib/apt/lists/*

CMD ["bash"]

