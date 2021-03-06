# Custom Block Device Driver for iOS Kernel

Up until now, we used the Ramdisk as the root mount. The ramdisk block device has two big disadvantages:
1. No support for disk larger than 1.5 GB.
2. We have only 1 block device in the system with the Ramdisk, but we need 2: one for the read only root mount and another for the r/w data mount.

So we decided to write a block device driver by ourselves

To execute the driver we will hook one of the kernel's functions.

We will use the same technique that was described in [pic-binary](https://github.com/alephsecurity/xnu-qemu-arm64-tools/blob/master/pic-binary/README.md).

### Instructions

1. Get the XNU sources from Apple's Github:
```
$ git clone https://github.com/apple/darwin-xnu.git 
```
2. Get the xnu-qemu-arm64 sources from Aleph:
```
$ git clone git@github.com:alephsecurity/xnu-qemu-arm64.git
```
3. Get the xnu-qemu-arm64-tools sources:
```
$ git clone git@github.com:alephsecurity/xnu-qemu-arm64-tools.git
```
4. Get the Cross Compiler for AARCH64. We will use the toolchain provided by @github/SergioBenitez:
```
$ brew tap SergioBenitez/osxct
$ brew install aarch64-none-elf
```
5. To be able to use the functions from the iOS Kernel within the driver code, we need to link the driver along with the symbols from the kernel. We can extract the symbols with `nm` and use them for this linkage process.
```
$ nm kernelcache.release.n66.out > symbols.nm
```

6. Create the Environment Variables
```
$ export XNU_SOURCES=full_path_to_darwin-xnu
$ export KERNEL_SYMBOLS_FILE=full_path_to_symbols.nm
$ export QEMU_DIR=full_path_to_xnu-qemu-arm64
$ export NUM_BLOCK_DEVS=2
```
7. Build
```
$ make -C xnu-qemu-arm64-tools/aleph_bdev_drv
```

The compilation process generates a flat `bin/aleph_bdev_drv.bin` binary, the compiled code of the driver.

Now that we have the driver compiled we can use it with the hook mechanism implemented in the [alephsecurity/xnu-qemu-arm64](https://github.com/alephsecurity/xnu-qemu-arm64)
