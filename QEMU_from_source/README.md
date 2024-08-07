# build qemu from source
```
apt install python3
apt install python3-venv
python3 install pip
pip install tomli
git clone https://github.com/qemu/qemu.git
cd qemu
mkdir build
cd build
../configure --help
../configure --target-list=arm-softmmu,arm-linux-user
make -j$(nproc)

./qemu-arm ../../busybox-1.36.1/busybox
./qemu-arm ../../busybox-1.36.1/busybox echo "hello world"

#compile system for arm
./qemu-system-arm -M help
./qemu-system-arm -M virt -m 256M -kernel ../../linux-6.1.58/arch/arm/boot/zImage -initrd ../../rootfs.cpio.gz -append "root=/dev/mem" -nographic
```