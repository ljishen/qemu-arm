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

## Miscellaneous
- The emulator sets up the [user networking](https://wiki.qemu.org/Documentation/Networking#User_Networking_.28SLIRP.29) by default so the ICMP traffic does not work (e.g. no ping), but it is good for simple web access for the start.
- Credit for this method goes to [this blog](https://translatedcode.wordpress.com/2017/07/24/installing-debian-on-qemus-64-bit-arm-virt-board/). You can also follow its guide if you want to create you own disk image in file. See the [docker/Dockerfile](https://github.com/ljishen/qemu-cortex-a53/blob/master/docker/Dockerfile) for the details of system default configurations.

The QEMU emulator `2.8.1` environment can be launched with
```bash
docker run -it \
    -v `pwd`/system:/root/system \
    ljishen/qemu-cortex-a53 bash
```
