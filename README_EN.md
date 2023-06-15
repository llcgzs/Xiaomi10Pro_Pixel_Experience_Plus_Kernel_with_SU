# Mi 10 and Mi 10 Pro pixel experience plus kernel architecture kernelSU tutorial
# (Other models and systems can also be referred to, mainly the part that modifies the kernel code later)
## Kernel 4.19 reference tutorial, directly use kprobe integration, unable to boot, so the kernel source code needs to be modified
# If you don't want to read the tutorial, just pull it to the end, see article 5
# Compile one yourself, or download the compiled one
# Help me release a Release
## 1. fork https://github.com/xiaoleGun/KernelSU_Action

## 2. modify file [config.env](https://github.com/kissunyeason/Xiaomi10Pro_Pixel_Experience_Plus_Kernel_with_SU/blob/main/config.env)

### Kernel Source

Change to your kernel URL

Fill in https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity
The kernel needs to be modified here, so fork one by yourself, and you can modify it conveniently. Please check the previous fork for the original kernel

### Kernel Source Branch

Modify to your kernel branch

we modify to[thirteen](https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/tree/thirteen) （安卓13）

### Kernel Config

Change to your kernel configuration file name

we modify to[cmi_stock-defconfig](https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/blob/thirteen/arch/arm64/configs/cmi_stock-defconfig)  

Be sure to find the corresponding file name

### Arch

For example: arm64

not modified here

### Kernel Image Name

Modify it to the kernel binary that needs to be flashed, which is generally consistent with the BOARD_KERNEL_IMAGE_NAME in your aosp-device treeFor example

For example: Image.gz-dtb

Common file formats are Image、Image.gz

This is actually the format of the kernel file

There is no suffix in the format of the kernel source code I use, fill in Image (the first letter is a capital I, the I of I'm Li Lei)

### Clang

Use custom clang

You can use a non-official clang such as [proton-clang](https://github.com/kdrag0n/proton-clang).

It is suggested here that if you don’t understand, don’t fill in randomly.

### Custom Clang Source

> Fill in a link that includes `.git` if it is a git repository.

Git repository or direct chain of compressed zip files is supported.

It is suggested here that if you don’t understand, don’t fill in randomly.

### Custom cmds

If you're using custom clang, you should be able to modify these settings on your own. :)

### Clang Branch

Due to [#23](https://github.com/xiaoleGun/KernelSU_Action/issues/23), we provide an option to customize the Google main branch. The main ones include:
| Clang Branch |
| ------------ |
| master |
| master-kernel-build-2021 |
| master-kernel-build-2022 |

Or other branches, please search for them according to your own needs at https://android.googlesource.com/platform/prebuilts/clang/host/linux-x86.

#### Clang Version

Enter the Clang version to use.

| Clang Version | Corresponding Android Version | AOSP-Clang Version |
| ------------- | ----------------------------- | ------------------ |
| 12.0.5 | Android S | r416183b |
| 14.0.6 | Android T | r450784d |
| 14.0.7 | | r450784e |
| 15.0.1 | | r458507 |

Generally, Clang12 can compile most of the 4.14 and above kernels.

```C
CLANG_BRANCH=master-kernel-build-2022
CLANG_VERSION=r450784d
```

Copy and paste if you don't understand what I'm talking about

### GCC

#### Enable GCC 64

Enable GCC 64C cross-compiler.

#### Enable GCC 32

Enable GCC 32C cross-compiler.

### Extra cmds

Some kernels require additional compilation commands to compile correctly. Generally, no other commands are needed, so please search for information about your kernel. Please separate the command and the command with a space.

For example: `LLVM=1 LLVM_IAS=1`

### Disable LTO

LTO is used to optimize the kernel but sometimes causes errors.

### Enable KernelSU

Enable KernelSU for troubleshooting kernel failures or compiling the kernel separately.

It must be written as true here, otherwise what do you think you are doing?

#### KernelSU Branch or Tag

Select the branch or tag of KernelSU:

- main branch (development version): `KERNELSU_TAG=main`
- Latest TAG (stable version): `KERNELSU_TAG=`
- Specify the TAG (such as `v0.5.2`): `KERNELSU_TAG=v0.5.2`
default is fine


### Disable CC_WERROR

It is used to repair some kernels that do not support or disable Kprobes, and repair KernelSU that does not detect variables with Kprobes enabled, throwing warnings and causing errors

Close here, fill in false

### Add Kprobes Config

Inject parameters into the defconfig automatically.

Don't modify it here, anyway, you need to modify the kernel

If you don't believe in evil, you can change it to true

### Add overlayfs Config

This parameter provides support for the KernelSU module and system partition read and write. Inject parameters into Defconfig automatically.

Don't modify it here, anyway, you need to modify the kernel

If you don't believe in evil, you can change it to true

### Enable ccache

Enable the cache to make the second kernel compile faster. It can reduce the time by at least 2/5.

### Need DTBO

> Added from previous workflows, view historical commits

Build boot.img, and you need to provide a `Source boot image`.

Fill in false here

### Build Boot IMG

> Added from previous workflows, view historical commits

Build boot.img, and you need to provide a `Source boot image`.

### Source Boot Image

As the name suggests, it provides a boot image source system that can boot normally and requires a direct chain, preferably from the same kernel source and AOSP device tree as your current system. Ramdisk contains the partition table and init, without which the compiled image will not boot up properly.


The suggestion here is to extract the boot.img corresponding to the ROM, publish it as a release, and then copy the url.
For example：https://github.com/kissunyeason/Xiaomi10Pro_Pixel_Experience_Plus_Kernel_with_SU/releases/download/offical-boot/boot.img

## 3. 开始编译
### 3.1 [action](https://github.com/kissunyeason/Xiaomi10Pro_Pixel_Experience_Plus_Kernel_with_SU/actions) 
### 3.2 [build-kernel](https://github.com/kissunyeason/Xiaomi10Pro_Pixel_Experience_Plus_Kernel_with_SU/actions/workflows/build-kernel.yml)
### 3.3 run workflow(right)
### 3.4 Wait for the building to complete
### 3.5 DownloadAnyKernel3-KernelSU-cmi-xxxxxxx.zip 
### 3.6 Flash in with TWRP
### 3.7 If you are lucky, you should NOT be able to boot (remember to backup boot.img in advance)
### 3.8 Throw your phone in the trash

## 4. Modify the kernel
### 4.1 Modify fs/exec.c (change in your fork's kernel source code!)

Find the following paragraph (about 1916 lines)
```C
putname(filename);
return retval;
```
  and
  
```C
static int do_execveat_common
```
Add these two lines between
```C
extern int ksu_handle_execveat(int *fd, struct filename **filename_ptr, void *argv,
void *envp, int *flags);
```

Refer to [here](https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/commit/bd6276dd5249b85ada5b6caf479e5c74dd269639)

Continue to this file

Found (about line 1923)
```C
struct user_arg_ptr envp,
int flags)
{
```
and
```C
return __do_execve_file(fd, filename, argv, envp, flags, NULL);
```
Add these two lines between
```C
ksu_handle_execveat(&fd, &filename, &argv, &envp, &flags);
```
Refer to [here](https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/commit/a0dfa44cbe79a2a532aadcfd33919e38ad753f26)
### 4.2 Modify fs/open.c (in the kernel source code of your fork!)

Find this paragraph (about 349 lines)
```C
return ksys_fallocate(fd, mode, offset, len);
}
```
and
```C
/*
 * access() needs to use the real uid/gid, not the effective uid/gid.
```
Add these two lines between
```C
extern int ksu_handle_faccessat(int *dfd, const char __user **filename_user, int *mode,
int *flags);
```

Found (about 357 lines)
```C
long do_faccessat(int dfd, const char __user *filename, int mode)
{
```
and 
```C
const struct cred *old_cred;
struct cred *override_cred;
struct path path;
```
Add these two lines between
```C
u_handle_faccessat(&dfd, &filename, &mode, NULL);
```
Refer [here](https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/commit/c2e8afafdd7ef3c5b706b6433c82ee00e7154996?diff=split)

### 4.3 Modify fs/read_write.c (change in your fork kernel source code!)
Find this line (about line 436)
```C
EXPORT_SYMBOL(kernel_read);
 
```
and
```C
ssize_t vfs_read(struct file *file, char __user *buf, size_t count, loff_t *pos)
 {
ssize_t ret;
```
Add these two lines between
```C
extern int ksu_handle_vfs_read(struct file **file_ptr, char __user **buf_ptr,
size_t *count_ptr, loff_t **pos);
```
the following paragraph
```C
ssize_t ret;
```
and
```C

if (!(file->f_mode & FMODE_READ))
return -EBADF;
```
Add these two lines between
```C
ksu_handle_vfs_read(&file, &buf, &count, &pos);
```
Refer [here](https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/commit/0af0751989211c9fbcd6480e1a10b91a9b600477?diff=split)

### 4.4 Modify fs/fs/stat.c (in the kernel source code of your fork!)

Find this paragraph (about 150 lines)
```C
EXPORT_SYMBOL(vfs_statx_fd);
```
and
```C

/**
 * vfs_statx - Get basic and extra attributes by filename
```
Add these two lines between
```C
extern int ksu_handle_stat(int *dfd, const char __user **filename_user, int *flags);
```

here (about line 170)
```C
struct path path;
int error = -EINVAL;
unsigned int lookup_flags = LOOKUP_FOLLOW | LOOKUP_AUTOMOUNT;
ksu_handle_stat(&dfd, &filename, &flags);
```
and 
```C
if ((flags & ~(AT_SYMLINK_NOFOLLOW | AT_NO_AUTOMOUNT |
AT_EMPTY_PATH | KSTAT_QUERY_FLAGS)) != 0)
return -EINVAL;
```
Add these two lines between
```C
ksu_handle_stat(&dfd, &filename, &flags);
```
Refer [here](https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/commit/03271214854e33efe56142ddfa12c830addcb32b?diff=split)

## 5. start Building
### 5.1 [action](https://github.com/kissunyeason/Xiaomi10Pro_Pixel_Experience_Plus_Kernel_with_SU/actions) 
### 5.2 [build-kernel](https://github.com/kissunyeason/Xiaomi10Pro_Pixel_Experience_Plus_Kernel_with_SU/actions/workflows/build-kernel.yml)
### 5.3 run workflow
### 5.4 Wait for the Building to complete
### 5.5 DownloadAnyKernel3-KernelSU-cmi-xxxxxxx.zip 
### 5.6 Flash in with TWRP
### 5.7 If you are lucky, it should boot up this time (remember to backup boot.img in advance)
### 5.8 Can not boot
### 5.9 Directly download my Build or [Release](https://github.com/kissunyeason/Xiaomi10Pro_Pixel_Experience_Plus_Kernel_with_SU/releases)
### 5.10 Can not boot
### 5.11 Throw your phone in the trash

## grateful

- [AnyKernel3](https://github.com/osm0sis/AnyKernel3)
- [AOSP](https://android.googlesource.com)
- [KernelSU](https://github.com/tiann/KernelSU)
- [xiaoxindada](https://github.com/xiaoxindada)
- [xiaoleGun](https://github.com/xiaoleGun/KernelSU_Action)
- [JayanthKandula](https://github.com/JayanthKandula/kernel_xiaomi_sm8250-immensity)
- @summ919
- @魔法师咖波
- @dadaadsda
