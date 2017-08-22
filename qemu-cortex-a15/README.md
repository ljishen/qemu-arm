# QEMU ARM Emulator on Cortex-A15

[![](https://images.microbadger.com/badges/image/ljishen/qemu-cortex-a15.svg)](http://microbadger.com/images/ljishen/qemu-cortex-a15)

This is a closet Raspberry Pi 2 machine emulation which is not running `Raspbian` system but `Debian GNU/Linux 9 (stretch)`.

## Motivation
- Even though the `raspi2` is in the QEMU supported machine list, it is not able to run the latest Raspbian system. These are the methods I have tried [1](https://blogs.msdn.microsoft.com/iliast/2016/11/10/how-to-emulate-raspberry-pi/) (black screen), [2](https://raspberrypi.stackexchange.com/a/71172) (CPUX: failed to come online), [3](http://blog.3mdeb.com/2015/12/30/emulate-rapberry-pi-2-in-qemu/) (only shows a raspberry logo) but none of them success. (Checked on Aug 16, 2017)
- The method introduced in the [msdn blog](https://blogs.msdn.microsoft.com/iliast/2016/11/10/how-to-emulate-raspberry-pi/) using `versatilepb` machine only supports up to 256MB of RAM which states clearly in the [QEMU wiki](https://wiki.qemu.org/Documentation/Platforms/ARM#Guidelines_for_choosing_a_QEMU_machine). Such configuration is hardly usable.
- Therefore, currently my final cloest solution is to emulate the ARM `Cortex-A15` CPU with machine `virt` running the Debian 9 system.

## Prerequisite
- x86 host

## Usage
```bash
docker run -it \
    -p 5555:22 \
    -v `pwd`/system:/root/system \
    ljishen/qemu-cortex-a15
```
We bind port 22 of the system emulator (the guest) to port 5555 of the host machine so that we can SSH access to the guest via:
```bash
ssh pi@localhost -p 5555
```

## System Defaults

### Credentials
- Ordinary User: `pi` with password `raspberry`
- Root: no password

### Configurations
- 4GB memory
- 4 aarch64 CPUs (BogoMIPS of each: 125.00)
- 8GB disk of image
- System info: `Linux debian 4.9.0-3-armmp-lpae #1 SMP Debian 4.9.30-2+deb9u3 (2017-08-06) armv7l GNU/Linux`

You can use environment variables `MEMORY` and `SMP` to change the default configurations:
```bash
docker run -it \
    -e MEMORY=8G -e SMP=6 \
    -v `pwd`/system:/root/system \
    ljishen/qemu-cortex-a15
```

## Miscellaneous
- The emulator sets up the [user networking](https://wiki.qemu.org/Documentation/Networking#User_Networking_.28SLIRP.29) by default so the ICMP traffic does not work (e.g. no ping), but it is good for simple web access for the start.
- Credit for this method goes to [this blog](https://translatedcode.wordpress.com/2016/11/03/installing-debian-on-qemus-32-bit-arm-virt-board/). I will briefly summarize the steps as follows in case the link breaks. You can always follow the guide if you want to create you own disk image in file. See the [docker/Dockerfile](https://github.com/ljishen/qemu-arm/blob/master/qemu-cortex-a15/docker/Dockerfile) for the details of system default configurations.

  1. Launch the QEMU emulator `2.8.1` environment with
  ```bash
  docker run -it \
      -v `pwd`/system:/root/system \
      -v /lib/modules:/lib/modules \
      -v /boot:/boot \
      ljishen/qemu-arm \
      bash
  ```

  2. Get the installer files
  ```bash
  cd ~
  apt-get update && apt-get install -y wget
  wget http://http.us.debian.org/debian/dists/stretch/main/installer-armhf/current/images/netboot/vmlinuz
  wget http://http.us.debian.org/debian/dists/stretch/main/installer-armhf/current/images/netboot/initrd.gz
  ```
<!---
File information of last check on:

vmlinuz	        19-Jul-2017 14:10	3.5M
initrd.gz	19-Jul-2017 14:10	20M
-->

  3. Create an empty disk image
  ```bash
  qemu-img create -f qcow system/hda.qcow2 8G
  ```

  4. Run the OS installer (will take several hours)
  ```bash
  qemu-system-arm \
      -M virt \
      -m 4G \
      -smp $((`nproc`>4?4:`nproc`)) \
      -cpu cortex-a15 \
      -kernel vmlinuz \
      -initrd initrd.gz \
      -drive if=none,file=system/hda.qcow2,format=qcow,id=hd0 \
      -device virtio-blk-device,drive=hd0 \
      -netdev user,id=net0 \
      -device virtio-net-device,netdev=net0 \
      -nographic \
      -no-reboot
  ```
  You will finally exit the QEMU env as the `-no-reboot` option supplies.

  5. Extract the kernel to the `system` dir
  ```bash
  # Install libguestfs (will ask a few questions)
  apt-get install -y libguestfs-tools

  # Check the content of our disk image first
  virt-ls -a system/hda.qcow2 /boot

  virt-copy-out -a system/hda.qcow2 /boot/vmlinuz-4.9.0-3-armmp-lpae /boot/initrd.img-4.9.0-3-armmp-lpae system

  # Exit docker container
  exit
  ```

  6. Now you are good to boot your own system using the same boot command as in the `Usage` section.
