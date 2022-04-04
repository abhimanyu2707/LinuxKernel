# CSE-506 (Spring 2022) HW1

## Setup Steps Performed:
### 1. Codebase setup
- Logged in into the VM provided by professor with the steps given in assignement.
- Cloned linux codebase in /usr/src which created "hw1-akugupta" with codebase.
- A new branch was created to commit changes for this assignment -- **hw1**
- All the new files(codes/README/config) are in directory -- **CSE-506**

### 2. Config setup
> Created a localmodconfig and made the required changes so that it boots in the correct kernel.
```
//Installed some required packages
apt-get install make build-essential libncurses-dev bison flex libssl-dev libelf-dev
apt-get install libssl-dev
apt-get install openssl

make localmodconfig
make menuconfig // Turned off CONFIG_MODVERSIONS (for Loadable Module support).
                // Other changes were made as per course website and for minimal config.
```

### 3. Building kernel
> Below code used to build the kernel and reboot.
```
make -j12 <clear all>
make modules
make modules_install install
reboot
uname -r  //for checking kernel version
```

### 4. Patch setup + New Files
> Used patch provided by professor with some minor changes to make lodable kernel module and user interface.
- Files changed in kernel to support syscalls:
```
///////////////////////////////////////
// syscall_64.tbl //Entry like below is needed here.
// 335     64      cryptocopy              __x64_sys_cryptocopy
//Note: Needed to copy a line from same file then edit (Space issues maybe).
///////////////////////////////////////
// fs/open.c
// Export the pointer used in CSE-506/sys_cryptocopy.c
// asymlinkage, EXPORT_SYMBOL, SYSCALL_DEFINE1 are some implementation to be done.
///////////////////////////////////////
// linux/syscalls.h
//Here add the function def like asmlinkage long sys_cryptocopy(void *ptr);
///////////////////////////////////////
```
- Code files added in folder **CSE-506**
```
README            //User manual
Makefile          //Build commands for lodable module and user interface.
install_module.sh //Script to update kernel module after a change.
sys_cryptocopy.h  //Header file shared by syscall and user interface
sys_cryptocopy.c  //Actual syscall function implementation
test_cryptocopy.c //User interface for syscall.
test_cases        //Folder with test scripts.
```


## Implementation Overview:
### 1. test_cryptocopy
- Handeles all the supported arguments.
- do an initial sanity check and report if any error.
- produces help page with -h option.
- Initialize and fill the data structure to be passed to sys call -- **struct cryptocopy_args**.
- Make syscall to cryptocopy with valid arguments.
- Provides user readable error if received any from the sys call.
- Extra arguments like -u and -l can be used to give cipher size and key length respectively for encryption/decryption.
- Decryption should have same value of cipher size(-l) as encryption. I only check for key which is stored as encrypted key in encrypted file. If it matches but cipher size mismatch, the output of decryption can be wrong.
- In future, plan is to create a metadata containing such information and stored in encrypted file.
- SHA256_CTX is used to generate hash of the password given by user. It generates 256 bits hash but user can use 128 key size as well. In that case I use the first 128 bit of the hash.

### 2. cryptocopy
- Main implementation of copy/encryption/decryption is in this file.
- Checks like valid parameter, SOURCE/DEST file permission, availabilty of source files are done here.
- If DEST file is present but have no permission then sys call will return with error.
- If DEST file is present and have permission, the sys call overwrites the file. In future plan is to move such file DEST.temp* as taught in the class.
- If DEST file is not present it is created with same priority as SOURCE file and the output is written.
- AES CTR is being currently used to encrypt and decrypt. But I have created a flow to pass option "-C [aes, des, des3_ede]". In future, the plan is to add other cipher support.
- I was trying to update IV in while loop of encryption/decryption. But I noticed that it is already getting incremented by AES library.

### 3. test_cases
- Scripts need to be run from folder CSE-506
- e.g. "./test_cases/test10.sh"

### 4. Code cleaning
- Used checkpatch.pl to remove all the errors.
- Some warnings were not fixed, motly related to line length.


## Steps to run the code
1. Starting with fresh CS506 linux machine.
2. git pull changes from hw1.
3. Use kernel.config to setup the kernel config.
4. make
5. make modules
6. make modules_install install
7. reboot.
8. goto folder **CSE-506**
9. make
10. sh install_module.sh
11. Use **./test_cryptocopy** to make syscall cryptocopy.

## References
- [CryptoAPI Example](https://docs.kernel.org/crypto/api-samples.html).
- [SHA256 Hashing](https://stackoverflow.com/questions/25544604/is-there-a-header-file-for-sha256).
- [File permissions.](https://stackoverflow.com/questions/19195560/whats-wrong-with-vfs-stat-call)
- https://stackoverflow.com/questions/29248585/c-checking-command-line-argument-is-integer-or-not
