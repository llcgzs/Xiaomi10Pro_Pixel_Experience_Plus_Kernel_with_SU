**Englis** | [中文](README.md)
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

## 3. Start Building
### 3.1 [action](https://github.com/kissunyeason/Xiaomi10Pro_Pixel_Experience_Plus_Kernel_with_SU/actions) 
### 3.2 [build-kernel](https://github.com/kissunyeason/Xiaomi10Pro_Pixel_Experience_Plus_Kernel_with_SU/actions/workflows/build-kernel.yml)
### 3.3 run workflow(right)
### 3.4 Wait for the building to complete
### 3.5 DownloadAnyKernel3-KernelSU-cmi-xxxxxxx.zip 
### 3.6 Flash in with TWRP
### 3.7 If you are lucky, you should NOT be able to boot (remember to backup boot.img in advance)
### 3.8 Throw your phone in the trash

## 4. Modify the kernel
### 4.1 Modify [fs/exec.c](https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/blob/thirteen/fs/exec.c)（Change the kernel source code in your fork! )）

 
Between(line 1916)
```C
static int __do_execve_file(int fd, struct filename *filename,
 	return retval;
 }
```
  and 
```C
 static int do_execveat_common(int fd, struct filename *filename,
 			      struct user_arg_ptr argv,
 			      struct user_arg_ptr envp,
 			      int flags)
 {
```
add
```C
extern int ksu_handle_execveat(int *fd, struct filename **filename_ptr, void *argv,
void *envp, int *flags);
```

Look[here](https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/commit/bd6276dd5249b85ada5b6caf479e5c74dd269639)


Between(lin1923)
```C
 static int do_execveat_common(int fd, struct filename *filename,
 			      struct user_arg_ptr argv,
 			      struct user_arg_ptr envp,
 			      int flags)
 {
```
and
```C
	return __do_execve_file(fd, filename, argv, envp, flags, NULL);
 }
```
add
```C
ksu_handle_execveat(&fd, &filename, &argv, &envp, &flags);
```
Look[here](https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/commit/a0dfa44cbe79a2a532aadcfd33919e38ad753f26)

### 4.2 修改[fs/open.c](https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/blob/thirteen/fs/open.c)（在你fork的内核源码改！）

在（大概349行）
```C
SYSCALL_DEFINE4(fallocate, int, fd, int, mode, loff_t, offset, loff_t, len)
 	return ksys_fallocate(fd, mode, offset, len);
 }
```
和
```C
/*
  * access() needs to use the real uid/gid, not the effective uid/gid.
  * We do this by temporarily clearing all FS-related capabilities and
```
add
```C
extern int ksu_handle_faccessat(int *dfd, const char __user **filename_user, int *mode,
int *flags);
```

在（大概357行）
```C
SYSCALL_DEFINE4(fallocate, int, fd, int, mode, loff_t, offset, loff_t, len)
  */
 long do_faccessat(int dfd, const char __user *filename, int mode)
 {
```
and
```C
 	const struct cred *old_cred;
 	struct cred *override_cred;
 	struct path path;
diff --git a/fs/read_write.c b/fs/read_write.c
index 650fc7e0f3a6..55be193913b6 100644
```
add
```C
u_handle_faccessat(&dfd, &filename, &mode, NULL);
```
Look[here](https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/commit/c2e8afafdd7ef3c5b706b6433c82ee00e7154996?diff=split)

### 4.3 修改[fs/read_write.c](https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/blob/thirteen/fs/read_write.c)（在你fork的内核源码改！）
在（大概436行）
```C
EXPORT_SYMBOL(kernel_read);
 
```
和
```C
ssize_t vfs_read(struct file *file, char __user *buf, size_t count, loff_t *pos)
 {
 	ssize_t ret;
 
```
在之间插入
```C
extern int ksu_handle_vfs_read(struct file **file_ptr, char __user **buf_ptr,
size_t *count_ptr, loff_t **pos);
```
在
```C
ssize_t vfs_read(struct file *file, char __user *buf, size_t count, loff_t *pos)
 {
 	ssize_t ret;
 
```
和
```C
 	if (!(file->f_mode & FMODE_READ))
 		return -EBADF;
 	if (!(file->f_mode & FMODE_CAN_READ))
diff --git a/fs/stat.c b/fs/stat.c
index 376543199b5a..82adcef03ecc 100644
```
之间插入
```C
ksu_handle_vfs_read(&file, &buf, &count, &pos);
```
参照[这里](https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/commit/0af0751989211c9fbcd6480e1a10b91a9b600477?diff=split)

### 4.4 修改[fs/stat.c](https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/blob/thirteen/fs/stat.c)（在你fork的内核源码改！）

在（大概150行）
```C
int vfs_statx_fd(unsigned int fd, struct kstat *stat,
 }
 EXPORT_SYMBOL(vfs_statx_fd);
 
```
和
```C
 /**
  * vfs_statx - Get basic and extra attributes by filename
  * @dfd: A file descriptor representing the base dir for a relative filename
```
之间插入
```C
extern int ksu_handle_stat(int *dfd, const char __user **filename_user, int *flags);
```

在（大概170行）
```C
int vfs_statx(int dfd, const char __user *filename, int flags,
 	int error = -EINVAL;
 	unsigned int lookup_flags = LOOKUP_FOLLOW | LOOKUP_AUTOMOUNT;
```
和
```C
if ((flags & ~(AT_SYMLINK_NOFOLLOW | AT_NO_AUTOMOUNT |
AT_EMPTY_PATH | KSTAT_QUERY_FLAGS)) != 0)
return -EINVAL;
```
之间插入
```C
	ksu_handle_stat(&dfd, &filename, &flags);
```
参照[这里](https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/commit/03271214854e33efe56142ddfa12c830addcb32b?diff=split)

## 到这里应该就可以了，直接跳到第5步

### 4.5 如果你的内核没有 vfs_statx, 使用 vfs_fstatat 来代替它：
#### 找到[fs/stat.c](https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/blob/thirteen/fs/stat.c)（在你fork的内核源码改！）
 在
   ```C
   int vfs_fstat(unsigned int fd, struct kstat *stat)
 }
 EXPORT_SYMBOL(vfs_fstat);
 
 ```
 与
 ```C
  int vfs_fstatat(int dfd, const char __user *filename, struct kstat *stat,
 		int flag)
 {
 ```
 之间插入
 ```C
 extern int ksu_handle_stat(int *dfd, const char __user **filename_user, int *flags);

 ```
### 4.6 对于早于 4.17 的内核，如果没有 do_faccessat，可以直接找到 faccessat 系统调用的定义然后修改：
找到[fs/open.c](https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/blob/thirteen/fs/open.c)（在你fork的内核源码改！）
在
```C
SYSCALL_DEFINE4(fallocate, int, fd, int, mode, loff_t, offset, loff_t, len)
 	return error;
 }
 
```
和
```C
 /*
  * access() needs to use the real uid/gid, not the effective uid/gid.
  * We do this by temporarily clearing all FS-related capabilities and
```
之间插入
```C
extern int ksu_handle_faccessat(int *dfd, const char __user **filename_user, int *mode,
			        int *flags);
```
在
```C
SYSCALL_DEFINE3(faccessat, int, dfd, const char __user *, filename, int, mode)
 	int res;
 	unsigned int lookup_flags = LOOKUP_FOLLOW;
 
```
和
```C
 	if (mode & ~S_IRWXO)	/* where's F_OK, X_OK, W_OK, R_OK? */
 		return -EINVAL;
```
之间插入
```C
ksu_handle_faccessat(&dfd, &filename, &mode, NULL);

```
### 4.6 要使用 KernelSU 内置的安全模式，你还需要修改 drivers/input/input.c 中的 input_handle_event 方法：
找到[/drivers/input/input.c](https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/blob/thirteen/drivers/input/input.c)
在
```C
static int input_get_disposition(struct input_dev *dev,
        return disposition;
 }


```
和
```C
 static void input_handle_event(struct input_dev *dev,
```
之间插入
```C
extern int ksu_handle_input_handle_event(unsigned int *type, unsigned int *code, int *value);

```
在
```C
static void input_handle_event(struct input_dev *dev,
                               unsigned int type, unsigned int code, int value)
 {
        int disposition = input_get_disposition(dev, type, code, &value);
```
和
```C

        if (disposition != INPUT_IGNORE_EVENT && type != EV_SYN)
                add_input_randomness(type, code, value);
```
之间插入
```C
       ksu_handle_input_handle_event(&type, &code, &value);

```

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
