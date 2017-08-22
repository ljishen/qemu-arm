# VERSION 1.0
FROM ljishen/qemu-arm
MAINTAINER Jianshen Liu <jliu120@ucsc.edu>

ENV MEMORY 4G
ENV SMP 4
ENV VMLINUXZ /root/system/vmlinuz-4.9.0-3-armmp-lpae
ENV INITRD_IMG /root/system/initrd.img-4.9.0-3-armmp-lpae
ENV DRIVE_FILE /root/system/hda.qcow2

CMD ["sh", "-c", \
     "qemu-system-aarch64 \
      -M virt \
      -m $MEMORY \
      -smp $SMP \
      -cpu cortex-a15 \
      -kernel $VMLINUXZ \
      -initrd $INITRD_IMG \
      -append 'root=/dev/vda1 console=ttyAMA0' \
      -drive if=none,file=${DRIVE_FILE},format=qcow,id=hd0 \
      -device virtio-blk-device,drive=hd0 \
      -netdev user,id=net0,hostfwd=tcp::22-:22 \
      -device virtio-net-device,netdev=net0 \
      -nographic"]
