
# QEMU with arm (https://developer.arm.com/downloads/-/gnu-a)
download: [arm](https://developer.arm.com/-/media/Files/downloads/gnu-a/10.3-2021.07/binrel/gcc-arm-10.3-2021.07-x86_64-arm-none-linux-gnueabihf.tar.xz?rev=302e8e98351048d18b6f5b45d472f406&hash=B981F1567677321994BE1231441CB60C7274BB3D)         
download: [kernel source](https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.1.58.tar.xz)             
download: [busybox](https://busybox.net/downloads/busybox-1.36.1.tar.bz2)           
```
sudo apt update && sudo apt install build-essential libncurses-dev rsync git ninja-build libglib2.0-dev libpixman-1-dev bison flex libssl-dev wget unzip bc file libfdt-dev zlib1g-dev python3 python3-venv
sudo apt-get install qemu-system-arm
sudo apt-get install libgmp-dev libmpfr-dev libmpc-dev
cd kernel-linux
tar -xf gcc-arm-10.3-2021.07-x86_64-arm-none-linux-gnueabihf.tar.xz
mv gcc-arm-10.3-2021.07-x86_64-arm-none-linux-gnueabihf arm-gnu-tool-chain
tar -xf busybox-1.36.1.tar.bz2
tar -xf linux-6.1.58.tar.xz

cd linux-6.1.58
make ARCH=arm CROSS_COMPILE=../arm-gnu-tool-chain/bin/arm-none-linux-gnueabihf- defconfig
make ARCH=arm CROSS_COMPILE=../arm-gnu-tool-chain/bin/arm-none-linux-gnueabihf- -j$(nproc --all)

ls arch/arm/boot
file !:1/Image
file arch/arm/boot/zImage
cd ../busybox-1.36.1
make ARCH=arm CROSS_COMPILE=../arm-gnu-tool-chain/bin/arm-none-linux-gnueabihf- defconfig
make ARCH=arm CROSS_COMPILE=../arm-gnu-tool-chain/bin/arm-none-linux-gnueabihf- menuconfig (select static share library)
make ARCH=arm CROSS_COMPILE=../arm-gnu-tool-chain/bin/arm-none-linux-gnueabihf- -j$(nproc --all)
readelf -d busybox | grep NEEDED (expected not dinamic lib in busybox folder)
make ARCH=arm CROSS_COMPILE=../arm-gnu-tool-chain/bin/arm-none-linux-gnueabihf- -j$(nproc --all) install

cd ..
mkdir -p rootfs/{bin,sbin,etc,proc,sys,usr/{bin/sbin}}
cp -av busybox-1.36.1/_install/* rootfs/
cd rootfs
ln -sf bin/busybox init
ls

mkdir dev
sudo mknod -m 660 mem c 1 1
sudo mknod -m 660 ttyS2 c 4 2 
sudo mknod -m 660 ttyS3 c 4 2 
sudo mknod -m 660 ttyS4 c 4 2 

mv mem dev
mv ttyS* dev

ls dev (output have to mem, ttyS2, ttyS3, ttyS4)
find . -print0 | cpio --null -ov --format=newc | gzip -9 > ../rootfs.cpio.gz

cd kernel-linux
qemu-system-arm -M help
qemu-system-arm -M (your machine choose) -m (number memory provide)M -kernel linux-6.1.58/arch/arm/boot/zImage -initrd rootfs.cpio.gz -append "root=/dev/mem" -nographic

#if error: cannot open /dev/tty*: No such file or directory
cd rootfs/dev
sudo rm -rf tty*
sudo mknod -m 660 tty2 c 4 2 
sudo mknod -m 660 tty3 c 4 3 
sudo mknod -m 660 tty4 c 4 4
cd ..
find . -print0 | cpio --null -ov --format=newc | gzip -9 > ../rootfs.cpio.gz
qemu-system-arm -M help
cd ..
qemu-system-arm -M virt -m 256M -kernel linux-6.1.58/arch/arm/boot/zImage -initrd rootfs.cpio.gz -append "root=/dev/mem" -nographic

ps -a
kill -9 pid-qemu
```