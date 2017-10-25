# An Approximation Emulator for Raspberry Pi 3 on Cortex-A53

[![](https://images.microbadger.com/badges/image/ljishen/qemu-cortex-a53.svg)](http://microbadger.com/images/ljishen/qemu-cortex-a53)

This is a Raspberry Pi 3 emulator on the same CPU architecture Cortex-A53. It runs `Debian GNU/Linux 9 (stretch)` instead of `Raspbian`.

## Motivation
- The Raspberry Pi 3 shipped with an ARM `Cortex-A53` CPU which is not included in the QEMU supported machines yet.
- For the Raspberry Pi 2, though the mathine is in the [supported list](https://wiki.qemu.org/Documentation/Platforms/ARM#Supported_Machines), I failed to emulate with `raspi2` using any of these methods [1](https://blogs.msdn.microsoft.com/iliast/2016/11/10/how-to-emulate-raspberry-pi/) (black screen), [2](https://raspberrypi.stackexchange.com/a/71172) (CPUX: failed to come online), [3](http://blog.3mdeb.com/2015/12/30/emulate-rapberry-pi-2-in-qemu/) (only shows a raspberry logo). (Checked on Aug 16, 2017)
- The method introduced in the [msdn blog](https://blogs.msdn.microsoft.com/iliast/2016/11/10/how-to-emulate-raspberry-pi/) using `versatilepb` machine only supports up to 256MB of RAM which states clearly in the [QEMU wiki](https://wiki.qemu.org/Documentation/Platforms/ARM#Guidelines_for_choosing_a_QEMU_machine). Such configuration is hardly usable.
- Therefore, currently the feasible approximation solution is to emulate the ARM `Cortex-A53` CPU with machine `virt` running the Debian 9 system.

## Prerequisite
- x86 host

## Usage
```bash
docker run -it \
    -p 5555:22 \
    -v `pwd`/system:/root/system \
    ljishen/qemu-cortex-a53
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
- System info: `Linux debian 4.9.0-3-arm64 #1 SMP Debian 4.9.30-2+deb9u3 (2017-08-06) aarch64 GNU/Linux`

You can use environment variables `MEMORY` and `SMP` to change the default configurations:
```bash
docker run -it \
    -e MEMORY=32G -e SMP=8 \
    -v `pwd`/system:/root/system \
    ljishen/qemu-cortex-a53
```

## Miscellaneous
- The emulator sets up the [user networking](https://wiki.qemu.org/Documentation/Networking#User_Networking_.28SLIRP.29) by default so the ICMP traffic does not work (e.g. no ping), but it is good for simple web access for the start.
- Credit for this method goes to [this blog](https://translatedcode.wordpress.com/2017/07/24/installing-debian-on-qemus-64-bit-arm-virt-board/). I will briefly summarize the steps as follows in case the link gets broken. You can always follow the guide if you want to create you own disk image in file. See the [docker/Dockerfile](https://github.com/ljishen/qemu-arm/blob/master/qemu-cortex-a53/docker/Dockerfile) for the details of system default configurations.

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
  wget http://http.us.debian.org/debian/dists/stretch/main/installer-arm64/current/images/netboot/debian-installer/arm64/linux
  wget http://http.us.debian.org/debian/dists/stretch/main/installer-arm64/current/images/netboot/debian-installer/arm64/initrd.gz
  ```
<!---
File information of last check on:

initrd.gz                                          19-Jul-2017 18:10            21500759
linux                                              19-Jul-2017 18:10            14080512
-->

  3. Create an empty disk image
  ```bash
  qemu-img create -f qcow system/hda.qcow2 8G
  ```

  4. Run the OS installer (will take several hours)
  ```bash
  qemu-system-aarch64 \
      -M virt \
      -m 4G \
      -smp $((`nproc`>4?4:`nproc`)) \
      -cpu cortex-a53 \
      -kernel linux \
      -initrd initrd.gz \
      -drive if=none,file=system/hda.qcow2,format=qcow,id=hd0 \
      -device virtio-blk-pci,drive=hd0 \
      -netdev user,id=net0 \
      -device virtio-net-pci,netdev=net0 \
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

  virt-copy-out -a system/hda.qcow2 /boot/vmlinuz-4.9.0-3-arm64 /boot/initrd.img-4.9.0-3-arm64 system

  # Exit docker container
  exit
  ```

  6. Now you are good to boot your own system using the same boot command in the `Usage` section.
