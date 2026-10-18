# openwrt-iusebash
Use bash as the default terminal

## Releases
You can find the prebuilt-ipks [here](https://fantastic-packages.github.io/releases/)

## Build

```shell
# Take the x86_64 platform as an example
tar -I zstd -xf openwrt-sdk-25.12.0-x86-64_gcc-14.3.0_musl.Linux-x86_64.tar.zst
# Go to the SDK root dir
cd OpenWrt-sdk-*-x86_64_*
# First run to generate a .config file
make menuconfig
./scripts/feeds update -a
./scripts/feeds install -a
# Get Makefile
git clone --depth 1 --branch master --single-branch --no-checkout https://github.com/muink/openwrt-iusebash.git package/iusebash
pushd package/iusebash
umask 022
git checkout
popd
# Select the package Utilities -> iusebash
make menuconfig
# Start compiling
make package/iusebash/compile V=99
```

## License
This project is licensed under the MIT License
