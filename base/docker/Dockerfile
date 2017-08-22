# VERSION 1.0
FROM debian:stretch-slim
MAINTAINER Jianshen Liu <jliu120@ucsc.edu>

RUN apt-get update \
    && apt-get install -y qemu-system-arm

# Clean Up
RUN apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
