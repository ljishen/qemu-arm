# qemu-cortex-a53

[![](https://images.microbadger.com/badges/image/ljishen/qemu-cortex-a53.svg)](http://microbadger.com/images/ljishen/qemu-cortex-a53)

This is the closet Raspberry Pi 3 machine emulation which is not running `Raspbian` system but `Debian GNU/Linux 9 (stretch)`.

## Motivation
- The Raspberry Pi 3 shipped with an ARM `Cortex-A53` CPU which is not included in the QEMU supported machines yet.
- For the Raspberry Pi 2, though the mathine is in the [supported list](https://wiki.qemu.org/Documentation/Platforms/ARM#Supported_Machines), I failed to emulate with `raspi2` using any of these methods [1](https://blogs.msdn.microsoft.com/iliast/2016/11/10/how-to-emulate-raspberry-pi/) (black screen), [2](https://raspberrypi.stackexchange.com/a/71172) (CPUX: failed to come online), [3](http://blog.3mdeb.com/2015/12/30/emulate-rapberry-pi-2-in-qemu/) (only shows a raspberry logo). (Checked on Aug 16, 2017)
- The method introduced in the [msdn blog](https://blogs.msdn.microsoft.com/iliast/2016/11/10/how-to-emulate-raspberry-pi/) using `versatilepb` machine only supports up to 256MB of RAM which states clearly in the [QEMU wiki](https://wiki.qemu.org/Documentation/Platforms/ARM#Guidelines_for_choosing_a_QEMU_machine). Such configuration is hardly usable.
- Therefore, currently my final cloest solution is to emulate the ARM `Cortex-A53` CPU with machine `virt` running the Debian 9 system.

## Usage
```bash
docker run -it \
    -v `pwd`/system:/root/system \
    ljishen/qemu-cortex-a53
```

## System Defaults

### Credentials
- Ordinary User: `pi` with password `raspberry`
- Root: no password

### Configurations
- 2GB memory
- 2 aarch64 CPUs (BogoMIPS: 125.00)
- 8GB disk of image
- System info: `Linux debian 4.9.0-3-arm64 #1 SMP Debian 4.9.30-2+deb9u3 (2017-08-06) aarch64 GNU/Linux`

You can use environment variables `MEMORY` and `CPUS` to change the default configurations:
```bash
docker run -it \
    -e MEMORY=32G -e CPUS=8 \
    -v `pwd`/system:/root/system \
    ljishen/qemu-cortex-a53
```

## Miscellaneous
- The emulator sets up the [user networking](https://wiki.qemu.org/Documentation/Networking#User_Networking_.28SLIRP.29) by default so the ICMP traffic does not work (e.g. no ping), but it is good for simple web access for the start.
- Credit for this method goes to [this blog](https://translatedcode.wordpress.com/2017/07/24/installing-debian-on-qemus-64-bit-arm-virt-board/). I will briefly summarize the steps as follows in case the link breaks. You can always follow the guide if you want to create you own disk image in file. See the [docker/Dockerfile](https://github.com/ljishen/qemu-cortex-a53/blob/master/docker/Dockerfile) for the details of system default configurations.

  1. Launch the QEMU emulator `2.8.1` environment with
  ```bash
  docker run -it \
      -v `pwd`/system:/root/system \
      -v /lib/modules:/lib/modules \
      -v /boot:/boot \
      ljishen/qemu-cortex-a53 \
      bash
  ```

  2. Get the installer files
  ```bash
  cd ~
  wget -O installer-linux http://http.us.debian.org/debian/dists/stretch/main/installer-arm64/current/images/netboot/debian-installer/arm64/linux
  wget -O installer-initrd.gz http://http.us.debian.org/debian/dists/stretch/main/installer-arm64/current/images/netboot/debian-installer/arm64/initrd.gz
  ```

  3. Create an empty disk image
  ```bash
  qemu-img create -f system/qcow hda.qcow2 8G
  ```

  4. Run the OS installer (will take several hours)
  ```bash
  qemu-system-aarch64 -M virt -m 2G -smp `nproc` -cpu cortex-a53 \
      -kernel installer-linux \
      -initrd installer-initrd.gz \
      -drive if=none,file=system/hda.qcow2,format=qcow,id=hd \
      -device virtio-blk-pci,drive=hd \
      -netdev user,id=mynet \
      -device virtio-net-pci,netdev=mynet \
      -nographic -no-reboot
  ```
  You will finally exit the QEMU env as the `-no-reboot` option supplies.

  5. Extract the kernel to the `system` dir
  ```bash
  # Install libguestfs (will ask a few questions)
  apt-get update && apt-get install libguestfs-tools

  # Check the content of our disk image first
  virt-ls -a system/hda.qcow2 /boot

  virt-copy-out -a system/hda.qcow2 /boot/vmlinuz-4.9.0-3-arm64 /boot/initrd.img-4.9.0-3-arm64 system
  
  # Exit docker container
  exit
  ```

  6. Now you are good to boot your own system using the same boot command as in the `Usage` section.
