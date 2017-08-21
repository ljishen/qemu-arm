# VERSION 1.0
FROM debian:stretch-slim
MAINTAINER Jianshen Liu <jliu120@ucsc.edu>

RUN apt-get update \
    && apt-get install -y qemu-system-arm

# Clean Up
RUN apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV MEMORY 2G
ENV SMP 2
ENV MACADDR 52:54:00:12:34:56
ENV VMLINUXZ /root/system/vmlinuz-4.9.0-3-arm64
ENV INITRD_IMG /root/system/initrd.img-4.9.0-3-arm64
ENV DRIVE_FILE /root/system/hda.qcow2

CMD ["sh", "-c", \
     "qemu-system-aarch64 \
      -M virt \
      -m $MEMORY \
      -smp $SMP \
      -cpu cortex-a53 \
      -kernel $VMLINUXZ \
      -initrd $INITRD_IMG \
      -append 'root=/dev/vda1 console=ttyAMA0' \
      -drive if=none,file=${DRIVE_FILE},format=qcow,id=hd0 \
      -device virtio-blk-pci,drive=hd0 \
      -netdev user,id=net0,hostfwd=tcp::22-:22 \
      -device virtio-net-pci,netdev=net0 \
      -nographic"]
