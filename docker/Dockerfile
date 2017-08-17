# VERSION 1.0

FROM debian:stretch-slim

MAINTAINER Jianshen Liu <jliu120@ucsc.edu>

RUN apt-get update \
    && apt-get install -y qemu

# Clean Up
RUN apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV MEMORY 2G
ENV VMLINUXZ /root/system/vmlinuz-4.9.0-3-arm64
ENV INITRD_IMG /root/system/initrd.img-4.9.0-3-arm64
ENV DRIVE_FILE /root/system/hda.qcow2 

CMD ["sh", "-c", \
     "qemu-system-aarch64 -M virt -m $MEMORY -smp 2 -cpu cortex-a53 \
      -kernel $VMLINUXZ \
      -initrd $INITRD_IMG \
      -append 'root=/dev/vda1' \
      -drive if=none,file=${DRIVE_FILE},format=qcow,id=hd \
      -device virtio-blk-pci,drive=hd \
      -netdev user,id=mynet \
      -device virtio-net-pci,netdev=mynet \
      -nographic"]
