# LinuxKernel
Building Linux kernel and making changes in kernel.

## 1. Import codebase.
- Like Linux 5.x

## 2. Create config file and make required changes.
```
make localmodconfig
make menuconfig
```
- You might need to install some packages for doing make.
```
apt-get install make build-essential libncurses-dev bison flex libssl-dev libelf-dev
apt-get install libssl-dev 
apt-get install openssl
```

## 3. Run build commands:
```
make
make modules
make modules_install install
```

## 4. Reboot and check if correct OS is loaded
```
reboot

uname -r  //for checking kernel version
```
