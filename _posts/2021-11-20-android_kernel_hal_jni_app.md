---
layout: post
title: Android_From_Kernel_To_Application
tags: [Tutorial,Jekyll]
author: laoYao
---

* content
{:toc}


# fix for download lineageos source file

```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>

  <remote  name="github"
           fetch="https://hub.fastgit.org/" />

  <remote  name="lineage"
           fetch="https://mirrors.tuna.tsinghua.edu.cn/git/lineageOS/"
           review="review.lineageos.org" />

  <remote  name="private"
           fetch="ssh://git@github.com" />

  <remote  name="aosp"
           fetch="https://mirrors.tuna.tsinghua.edu.cn/git/AOSP"
           review="android-review.googlesource.com"
           revision="refs/tags/android-11.0.0_r37" />

  <default revision="refs/heads/lineage-18.1"
           remote="github"
           sync-c="true"
           sync-j="4" />
```

# sample

    dts
        /home/forlinx/android_project/kernel_imx/arch/arm/boot/dts/imx6qdl.dtsi
        /home/forlinx/android_project/kernel_imx/arch/arm/boot/dts/imx6qdl-sabresd.dtsi

    kernel
        /home/forlinx/android_project/kernel_imx/drivers/video/backlight/pwm_bl.c

    hal
        /home/forlinx/android_project/hardware/imx/lights/fsl_lights.c

    jni
        /home/forlinx/android_project/frameworks/base/services/core/java/com/android/server/lights/Light.java
        /home/forlinx/android_project/frameworks/base/services/core/java/com/android/server/lights/LightsManager.java
        /home/forlinx/android_project/frameworks/base/services/core/java/com/android/server/lights/LightsService.java

    "application"
        /home/forlinx/android_project/frameworks/base/services/core/java/com/android/server/power/PowerManagerService.java

# kernel

`freg.c`
```c
//kernel\goldfinsh\drivers\freg\freg.c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/types.h>
#include <linux/fs.h>
#include <linux/proc_fs.h>
#include <linux/device.h>
#include <linux/slab.h>
#include <linux/seq_file.h>
#include <asm/uaccess.h>

#include "freg.h"

static int freg_major = 0;//主设备号
static int freg_minor = 0;//从设备号

static struct class* freg_class = NULL;
static struct fake_reg_dev* freg_dev = NULL;

//传统文件操作方法表
static int freg_open(struct inode* inode, struct file* filp);
static int freg_release(struct inode* inode, struct file* filp);
static ssize_t freg_read(struct file* filp, char __user *buf, size_t count, loff_t* f_pos);
static ssize_t freg_write(struct file* filp, const char __user *buf, size_t count, loff_t* f_pos);

static struct file_operations freg_fops = {
    .owner = THIS_MODULE,
    .open = freg_open,
    .release = freg_release,
    .read = freg_read,
    .write = freg_write,
};

#define SEQ_printf(m, x...)     \
    do {                \
        if (m)          \
            seq_printf(m, x);   \
        else            \
            pr_err(x);      \
    } while (0)

//读取设备寄存器val的值
static int freg_proc_show(struct seq_file *m, void *v)
{
    SEQ_printf(m, "%d\n", freg_dev->val);
    return 0;
}

//打开设备寄存器
static int freg_proc_open(struct inode *inode, struct file *file)
{
    return single_open(file, freg_proc_show, inode->i_private);
}

//把缓存区buf的值写到设备寄存器val中，内部使用， freg_val_store freg_proc_write调用
static ssize_t __freg_set_val(struct fake_reg_dev* dev, const char* buf, size_t count){
    int val = 0;

    val = simple_strtol(buf, NULL, 10);

    if(down_interruptible(&(dev->sem))){
        return -ERESTARTSYS;
    }

    dev->val = val;
    up(&(dev->sem));

    return count;
}

/*
static ssize_t freg_proc_read(char* page, char** start, off_t off, int count, int* eof, void* data){
    if(off >0 ){
        *eof = 1;
        return 0;
    }

    return __freg_get_val(freg_dev, page);
}*/

//将缓存器的值buff保存到设备寄存器val中
static ssize_t freg_proc_write(struct file *filp, const char *ubuf, size_t cnt, loff_t *data){
    int err = 0;
    char* page = NULL;

    if(cnt > PAGE_SIZE){
        printk(KERN_ALERT"The buff is too large: %lu.\n", cnt);
        return -EFAULT;
    }

    page = (char*) __get_free_page(GFP_KERNEL);
    if(!page){
        printk(KERN_ALERT"Failed to alloc page.\n");
        return -ENOMEM;
    }

	//先把用户提供的缓冲区的值复制到内核缓冲区中
    if(copy_from_user(page, ubuf, cnt)){
        printk(KERN_ALERT"Failed to copy buff from user.\n");
        err = -EFAULT;
        goto out;
    }

    err = __freg_set_val(freg_dev, page, cnt);

out:
    free_page((unsigned long)page);
    return err;
}

static const struct file_operations freg_proc_fops = {
    .open = freg_proc_open,
    .write = freg_proc_write,
    .read = seq_read,
    .llseek = seq_lseek,
    .release = single_release,
};


//devfs文件系统设备属性操作方法
static ssize_t freg_val_show(struct device* dev, struct device_attribute* attr, char* buf);
static ssize_t freg_val_store(struct device* dev, struct device_attribute* attr, const char* buf, size_t count);

//devfs文件系统设备属性
static DEVICE_ATTR(val, S_IRUGO | S_IWUSR, freg_val_show, freg_val_store);

//打开设备方法
static int freg_open(struct inode * inode, struct file * filp){
    struct fake_reg_dev * dev;

	//将自定义设备结构体保存在文件指针的私有数据域中，以便访问设备时直接拿来用
    dev = container_of(inode->i_cdev, struct fake_reg_dev, dev);
    filp->private_data = dev;

    return 0;
}

//设备文件释放时调用，空实现
static int freg_release(struct inode* inode, struct file* filp){
    return 0;
}
//读取设备寄存器val的值
static ssize_t freg_read(struct file* filp, char __user *buf, size_t count, loff_t* f_pos){
    ssize_t err = 0;
    struct fake_reg_dev* dev = filp->private_data;

	//同步访问
    if(down_interruptible(&(dev->sem))){
        return -ERESTARTSYS;
    }

    if(count < sizeof(dev->val)){
        goto out;
    }

	//将寄存器val的值复制到用户提供的缓冲区中
    if(copy_to_user(buf, &(dev->val), sizeof(dev->val))){
        err = -EFAULT;
        goto out;
    }

    err = sizeof(dev->val);

out:
	//撤销同步
    up(&(dev->sem));
    return err;
}

//写设备寄存器val的值
static ssize_t freg_write(struct file * filp, const char __user * buf, size_t count, loff_t * f_pos){
    struct fake_reg_dev* dev = filp->private_data;
    ssize_t err = 0;

    if(down_interruptible(&(dev->sem))){
        return -ERESTARTSYS;
    }

    if(count != sizeof(dev->val)){
        goto out;
    }

	//将寄存器val的值读取到缓冲区buf中，内部使用，freg_val_show调用
    if(copy_from_user(&(dev->val), buf, count)){
        err = -EFAULT;
        goto out;
    }

    err = sizeof(dev->val);

out:
    up(&(dev->sem));
    return err;
}

static ssize_t __freg_get_val(struct fake_reg_dev* dev, char* buf){
    int val = 0;

    if(down_interruptible(&dev->sem)){
        return -ERESTARTSYS;
    }

    val = dev->val;
    up(&(dev->sem));

    return snprintf(buf, PAGE_SIZE, "%d\n", val);
}


//都设备属性val值
static ssize_t freg_val_show(struct device* dev, struct device_attribute* attr, char* buf){
    struct fake_reg_dev* hdev = (struct fake_reg_dev*)dev_get_drvdata(dev);

    return __freg_get_val(hdev, buf);
}

//写设备属性val
static ssize_t freg_val_store(struct device*dev, struct device_attribute* attr, const char* buf, size_t count){
    struct fake_reg_dev* hdev = (struct fake_reg_dev*)dev_get_drvdata(dev);

    return __freg_set_val(hdev, buf, count);
}


//创建 /proc/freg文件
static void freg_create_proc(void){
    proc_create(FREG_DEVICE_PROC_NAME, 0644, 0,  &freg_proc_fops);
}

static void freg_remove_proc(void){
    remove_proc_entry(FREG_DEVICE_PROC_NAME, NULL);
}

//初始化设备
static int __freg_setup_dev(struct fake_reg_dev* dev){
    int err;
    dev_t devno = MKDEV(freg_major, freg_minor);

    memset(dev, 0, sizeof(struct fake_reg_dev));

	//初始化字符设备
    cdev_init(&(dev->dev), &freg_fops);
    dev->dev.owner = THIS_MODULE;
    dev->dev.ops = &freg_fops;

	//注册字符设备
    err = cdev_add(&(dev->dev), devno, 1);
    if(err){
        return err;
    }

	//初始化信号量
    //init_MUTEX(&(dev->sem));
    sema_init(&(dev->sem), 1);
    //初始化寄存器val值
    dev->val = 0;

    return 0;
}

//模块加载方法
static int __init freg_init(void){
    int err = -1;
    dev_t dev = 0;
    struct device* temp = NULL;

    printk(KERN_ALERT"Initializing freg device.\n");

	//动态分配主设备号和从设备号
    err = alloc_chrdev_region(&dev, 0, 1, FREG_DEVICE_NODE_NAME);
    if(err < 0){
        printk(KERN_ALERT"Failed to alloc char dev region.\n");
        goto fail;
    }

    freg_major = MAJOR(dev);
    freg_minor = MINOR(dev);

	//分配freg设备结构体
    freg_dev = kmalloc(sizeof(struct fake_reg_dev), GFP_KERNEL);
    if(!freg_dev){
        err = -ENOMEM;
        printk(KERN_ALERT"Failed to alloc freg device.\n");
        goto unregister;
    }

	//初始化设备
    err = __freg_setup_dev(freg_dev);
    if(err){
        printk(KERN_ALERT"Failed to setup freg device: %d.\n", err);
        goto cleanup;
    }

	//在sys/class目录下创建设备类别目录freg
    freg_class = class_create(THIS_MODULE, FREG_DEVICE_CLASS_NAME);
    if(IS_ERR(freg_class)){
        err = PTR_ERR(freg_class);
        printk(KERN_ALERT"Failed to create freg device class.\n");
        goto destroy_cdev;
    }

	//dev目录和/syst/class/freg目录下分别创建设备文件freg
    temp = device_create(freg_class, NULL, dev, NULL, "%s", FREG_DEVICE_FILE_NAME);
    if(IS_ERR(temp)){
        err = PTR_ERR(temp);
        printk(KERN_ALERT"Failed to create freg device.\n");
        goto destroy_class;
    }

	//在/sys/class/freg/freg目录下创建属性文件val
    err = device_create_file(temp, &dev_attr_val);
    if(err < 0){
        printk(KERN_ALERT"Failed to create attribute val of freg device.\n");
        goto destroy_device;
    }

    dev_set_drvdata(temp, freg_dev);

    freg_create_proc();

    printk(KERN_ALERT"Succedded to initialize freg device.\n");

    return 0;

destroy_device:
    device_destroy(freg_class, dev);
destroy_class:
    class_destroy(freg_class);
destroy_cdev:
    cdev_del(&(freg_dev->dev));
cleanup:
    kfree(freg_dev);
unregister:
    unregister_chrdev_region(MKDEV(freg_major, freg_minor), 1);
fail:
    return err;
}

//模块卸载
static void __exit freg_exit(void){
    dev_t devno = MKDEV(freg_major, freg_minor);

    printk(KERN_ALERT"Destory freg device.\n");

	//删除 /proc/freg文件
    freg_remove_proc();

	//注销设备类别和设备
    if(freg_class){
        device_destroy(freg_class, MKDEV(freg_major, freg_minor));
        class_destroy(freg_class);
    }

	//删除字符设备和释放设备内存
    if(freg_dev){
        cdev_del(&(freg_dev->dev));
        kfree(freg_dev);
    }

	//释放设备号资源
    unregister_chrdev_region(devno, 1);
}

MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("Fake Register Driver");

module_init(freg_init);
module_exit(freg_exit);

```

`freg.h`
```c
// kernel\goldfish\drivers\freg\freg.h

#ifndef _FAKE_REG_H_
#define _FAKE_REG_H_

#include <linux/cdev.h>
#include <linux/semaphore.h>

//定义了四个字符串常量，分别用来描述虚拟硬件设备 freg 在设备文件系统中的名称
#define FREG_DEVICE_NODE_NAME "freg"
#define FREG_DEVICE_FILE_NAME "freg"
#define FREG_DEVICE_PROC_NAME "freg"
#define FREG_DEVICE_CLASS_NAME "freg"

// 描述虚拟硬件设备freg
struct fake_reg_dev 
{
    // 描述一个虚拟寄存器
    int val;
    // 一个信号量, 用来同步访问虚拟寄存器 val
    struct semaphore sem;
    // 一个标准的Linux字符设备结构体变量， 用来标志该虚拟硬件设备 freg 的类型为字符设备
    struct cdev dev;
};

#endif
```

`makefile`
```makefile
# kernel\goldfish\drivers\freg\Makefile
obj-$(CONFIG_FREG) += freg.o
```

`Kconfig`
```
# kernel\goldfish\drivers\freg\Kconfig
config FREG
	tristate "Fake Register Driver"
	default y
	help
	This is the freg driver for android system
```

`README`
```diff
    /drivers/Makefile
+    obj-y   += freg/

    /drivers/Kconfig
+    source "drivers/freg/Kconfig"


    > maybe need delete `kernel/.config`

    > [https://www.jianshu.com/p/4ea84d1ce4e5]

    > [https://blog.csdn.net/gangjindianzi/article/details/116505264]
```

# hal

`README`
```
hal硬件抽象层

[https://blog.csdn.net/gangjindianzi/article/details/116505964]
```

`Android.mk`
```mk
# Android/hardware/libhardware/Modules/freg/Android.mk

LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE_TAGS := optional
LOCAL_PRELINK_MODULE := false

# 保存在 out/target/product/generic/system/lib/hw
LOCAL_MODULE_PATH := $(TARGET_OUT_SHARED_LIBRARIES)/hw
LOCAL_SHARED_LIBRARIES := liblog
LOCAL_SRC_FILES := freg.cpp
LOCAL_MODULE := freg.default

# 将该硬件抽象层模块编译成一个动态链接库文件，名称为 freg.default.so
include $(BUILD_SHARED_LIBRARY)
# 根据硬件抽象层模块文件的命名规范，当我们要加载硬件抽象层模块freg时，只需要指定它的ID值 == freg
# 系统就会根据一定的规则成功地找到要加载的 freg.default.so 文件

```

`freg.cpp`
```cpp
// Android/hardware/libhardware/Modules/freg/freg.cpp

#define LOG_TAG "FregHALStub"

#include <hardware/hardware.h>
#include <hardware/freg.h>

#include <fcntl.h>
#include <errno.h>

#include <cutils/log.h>
#include <cutils/atomic.h>

#define DEVICE_NAME "/dev/freg"
#define MODULE_NAME "Freg"
#define MODULE_AUTHOR "cpucode"

/* 设备打开接口 */
static int freg_device_open(const struct hw_module_t* module, const char* id, struct hw_device_t** device);
/* 设备关闭接口 */
static int freg_device_close(struct hw_device_t* device);
/* 设备寄存器写接口 */
static int freg_set_val(struct freg_device_t* dev, int val);
/* 设备寄存器读接口 */
static int freg_get_val(struct freg_device_t* dev, int* val);

/* 定义模块操作方法结构体变量 */
static struct hw_module_methods_t freg_module_methods = {
    open: freg_device_open
};

/* 定义模块结构体变量 */
// 每一个硬件抽象层模块必须导出一个名称为 HAL_MODULE_INFO_SYM 的符号
struct freg_module_t HAL_MODULE_INFO_SYM = {
    common: {
        // tag 必须 == HARDWARE_MODULE_TAG
        tag: HARDWARE_MODULE_TAG,    
        version_major: 1,
        version_minor: 0,
        id: FREG_HARDWARE_MODULE_ID,
        name: MODULE_NAME,
        author: MODULE_AUTHOR,
        methods: &freg_module_methods,
    }
};

// 打开操作
static int freg_device_open(const struct hw_module_t* module, 
                            const char* id, 
                            struct hw_device_t** device) 
{
    // 判断 id 与虚拟硬件设备freg的ID值是否 匹配
    if(!strcmp(id, FREG_HARDWARE_DEVICE_ID))
    {
        struct freg_device_t* dev;

        dev = (struct freg_device_t*)malloc(sizeof(struct freg_device_t));
        if(!dev) 
        {
            LOGE("Failed to alloc space for freg_device_t.");
            return -EFAULT;    
        }

        memset(dev, 0, sizeof(struct freg_device_t));

        // 硬件设备标签（dev->common.tag） 必须 == HARDWARE_DEVICE_TAG
        dev->common.tag = HARDWARE_DEVICE_TAG;
        dev->common.version = 0;
        dev->common.module = (hw_module_t*)module;
        // 关闭函数设置为 freg_device_close
        dev->common.close = freg_device_close;
        // 写函数
        dev->set_val = freg_set_val;
        // 读函数
        dev->get_val = freg_get_val;

        // 打开虚拟硬件设备文件/dev/freg ， 且将得到的文件描述符保存在结构体 freg_device_t 的成员变量 fd 中
        // 不修改设备文件/dev/freg的访问权限 , 打不开
        if((dev->fd = open(DEVICE_NAME, O_RDWR)) == -1)
        {
            LOGE("Failed to open device file /dev/freg -- %s.", strerror(errno));
            free(dev);

            return -EFAULT;
        }

        *device = &(dev->common);

        LOGI("Open device file /dev/freg successfully.");

        return 0;
    }

    return -EFAULT;
}

// 虚拟硬件设备freg的关闭函数
static int freg_device_close(struct hw_device_t* device) 
{
    struct freg_device_t* freg_device = (struct freg_device_t*)device;

    if(freg_device) 
    {
        // 关闭设备文件/dev/freg
        close(freg_device->fd);
        // 释放设备
        free(freg_device);
    }

    return 0;
}

static int freg_set_val(struct freg_device_t* dev, int val) 
{
    if(!dev) 
    {
        LOGE("Null dev pointer.");
        return -EFAULT;
    }

    LOGI("Set value %d to device file /dev/freg.", val);
    // 写虚拟硬件设备freg的寄存器val的内容
    write(dev->fd, &val, sizeof(val));

    return 0;
}

static int freg_get_val(struct freg_device_t* dev, int* val) 
{
    if(!dev) 
    {
        LOGE("Null dev pointer.");
        return -EFAULT;
    }

    if(!val) 
    {
        LOGE("Null val pointer.");
        return -EFAULT;
    }

    // 读虚拟硬件设备freg的寄存器val的内容
    read(dev->fd, val, sizeof(*val));

    LOGI("Get value %d from device file /dev/freg.", *val);

    return 0;
}

```

`freg.h`
```h
// Android/hardware/libhardware/include/hardware/freg.h

#ifndef ANDROID_FREG_INTERFACE_H
#define ANDROID_FREG_INTERFACE_H

__BEGIN_DECLS

// 常量和结构体都是按照硬件抽象层模块编写规范来定义的
/** * The id of this module */
// 定义模块ID
#define FREG_HARDWARE_MODULE_ID "freg"

/** * The id of this device */
// 定义设备ID
#define FREG_HARDWARE_DEVICE_ID "freg"

// 描述自定义模块结构体
struct freg_module_t 
{
    // 第一个成员变量的类型必须为 hw_module_t
    struct hw_module_t common;
};

// 描述虚拟硬件设备 freg
struct freg_device_t 
{
    // 第一个成员变量的类型必须为 hw_device_t
    struct hw_device_t common;
    
    // 一个文件描述符 , 用来描述打开的设备文件/dev/freg
    int fd;

    // 写虚拟硬件设备 freg 的寄存器 val 的内容
    int (*set_val)(struct freg_device_t* dev, int val);
    // 读虚拟硬件设备 freg 的寄存器 val 的内容
    int (*get_val)(struct freg_device_t* dev, int* val);
};

__END_DECLS

#endif

```

`hardware.c`
```c
// hardware\libhardware\hardware.c


#include <hardware/hardware.h>

#include <cutils/properties.h>

#include <dlfcn.h>
#include <string.h>
#include <pthread.h>
#include <errno.h>
#include <limits.h>

#define LOG_TAG "HAL"
#include <utils/Log.h>

/** Base path of the hal modules */
#if defined(__LP64__)

// 定义要加载的硬件抽象层模块文件所在的目录
// 编译好的模块文件位于 out/target/product/generic/system/lib/hw 目录中， 
// 而这个目录经过打包后， 就对应于设备上的 /system/lib/hw 目录
#define HAL_LIBRARY_PATH1 "/system/lib64/hw"

// 定义的目录为 /vendor/lib/hw ，用来保存设备厂商所提供的硬件抽象层模块接口文件
#define HAL_LIBRARY_PATH2 "/vendor/lib64/hw"

#else
#define HAL_LIBRARY_PATH1 "/system/lib/hw"
#define HAL_LIBRARY_PATH2 "/vendor/lib/hw"
#endif

/**
 * There are a set of variant filename for modules. The form of the filename
 * is "<MODULE_ID>.variant.so" so for the led module the Dream variants 
 * of base "ro.product.board", "ro.board.platform" and "ro.arch" would be:
 *
 * led.trout.so
 * led.msm7k.so
 * led.ARMV6.so
 * led.default.so
 */
// variant 表示四个系统属性 ro.hardware、 ro.product.board、 ro.board.platform 和 ro.arch 之一
// 依次按照上到下的顺序来取它们的属性值, 其中有一个系统属性存在，就把它的值作为 variant 的值，
// 然后再检查对应的文件是否存在，如果存在，就找到要加载的硬件抽象层模块文件了； 
// 否则， 就继续查找下一个系统属性。 
// 如 所有的系统属性都不存在， 或 所有的硬件抽象层模块文件都不存在， 
// 就使用“ ＜MODULE_ID＞.default.so ”来作为要加载的硬件抽象层模块文件的名称
/**
    系统属性ro.hardware是在系统启动时， 由init进程负责设置的。
    它会首先读取/proc/cmdline文件， 然后检查里面是否有androidboot.hardware的属性， 如果有，就把它的值作为属性ro.hardware的值；
    否则， 就将/proc/cpuinfo文件里面的硬件信息解析出来， 也就是 将Hardware字段的内容作为属性ro.hardware的值。
    系统属性ro.product.board、ro.board.platform和 ro.arch是从/system/build.prop文件读取出来的。
    文件/system/build.prop是由编译系统中的编译脚本 build/core/Makefile 和 Shell脚本 build/tools/buildinfo.sh 生成的
*/

// 组装要加载的硬件抽象层模块的文件名称
static const char *variant_keys[] = {
    "ro.hardware",  /* This goes first so that it can pick up a different
                       file on the emulator. */
    "ro.product.board",
    "ro.board.platform",
    "ro.arch"
};

// 数组 variant_keys的大小
static const int HAL_VARIANT_KEYS_COUNT =
    (sizeof(variant_keys)/sizeof(variant_keys[0]));

/**
 * Load the file defined by the variant and if successful
 * return the dlopen handle and the hmi.
 * @return 0 = success, !0 = failure.
 */
static int load(const char *id,
        const char *path,
        const struct hw_module_t **pHmi)
{
    int status = -EINVAL;
    void *handle = NULL;
    struct hw_module_t *hmi = NULL;

    /*
     * load the symbols resolving undefined symbols before
     * dlopen returns. Since RTLD_GLOBAL is not or'd in with
     * RTLD_NOW the external symbols will not be global
     */
    /* 将动态链接库文件加载到内存中 */
    handle = dlopen(path, RTLD_NOW);
    if (handle == NULL) {
        char const *err_str = dlerror();
        ALOGE("load: module=%s\n%s", path, err_str?err_str:"unknown");
        status = -EINVAL;
        goto done;
    }

    /* Get the address of the struct hal_module_info. */
    const char *sym = HAL_MODULE_INFO_SYM_AS_STR;

    // 获得里面名称为 HAL_MODULE_INFO_SYM_AS_STR 的符号
    // 符号指向的是一个自定义的硬件抽象层模块结构体，
    // 它包含了对应的硬件抽象层模块的所有信息
    // 将模块中的 HMI 符号转换为一个 hw_module_t 结构体指针
    hmi = (struct hw_module_t *)dlsym(handle, sym);
    if (hmi == NULL) {
        ALOGE("load: couldn't find symbol %s", sym);
        status = -EINVAL;
        goto done;
    }

    /* Check that the id matches */
    /* 验证加载得到的硬件抽象层模块ID 是否 与所要求加载的硬件抽象层模块ID一致 */
    if (strcmp(id, hmi->id) != 0) {
        ALOGE("load: id=%s != hmi->id=%s", id, hmi->id);
        status = -EINVAL;
        goto done;
    }

    //将成功加载后得到的模块句柄值 handle 保存在 hw_module_t 结构体指针 hmi 的成员变量 dso 中
    hmi->dso = handle;

    /* success */
    status = 0;

    done:
    if (status != 0) {
        hmi = NULL;
        if (handle != NULL) {
            dlclose(handle);
            handle = NULL;
        }
    } else {
        ALOGV("loaded HAL id=%s path=%s hmi=%p handle=%p",
                id, path, *pHmi, handle);
    }

    *pHmi = hmi;

    return status;
}

/*
 * Check if a HAL with given name and subname exists, if so return 0, otherwise
 * otherwise return negative.  On success path will contain the path to the HAL.
 */
static int hw_module_exists(char *path, size_t path_len, const char *name,
                            const char *subname)
{
    snprintf(path, path_len, "%s/%s.%s.so",
             HAL_LIBRARY_PATH2, name, subname);
    if (access(path, R_OK) == 0)
        return 0;

    snprintf(path, path_len, "%s/%s.%s.so",
             HAL_LIBRARY_PATH1, name, subname);
    if (access(path, R_OK) == 0)
        return 0;

    return -ENOENT;
}

int hw_get_module_by_class(const char *class_id, const char *inst,
                           const struct hw_module_t **module)
{
    int i = 0;
    char prop[PATH_MAX] = {0};
    char path[PATH_MAX] = {0};
    char name[PATH_MAX] = {0};
    char prop_name[PATH_MAX] = {0};


    if (inst)
        snprintf(name, PATH_MAX, "%s.%s", class_id, inst);
    else
        strlcpy(name, class_id, PATH_MAX);

    /*
     * Here we rely on the fact that calling dlopen multiple times on
     * the same .so will simply increment a refcount (and not load
     * a new copy of the library).
     * We also assume that dlopen() is thread-safe.
     */

    /* First try a property specific to the class and possibly instance */
    snprintf(prop_name, sizeof(prop_name), "ro.hardware.%s", name);
    if (property_get(prop_name, prop, NULL) > 0) {
        if (hw_module_exists(path, sizeof(path), name, prop) == 0) {
            goto found;
        }
    }

    /* Loop through the configuration variants looking for a module */
    for (i=0 ; i<HAL_VARIANT_KEYS_COUNT; i++) {
        // 获得的系统属性的值
        if (property_get(variant_keys[i], prop, NULL) == 0) {
            continue;
        }
        if (hw_module_exists(path, sizeof(path), name, prop) == 0) {
            goto found;
        }
    }

    /* Nothing found, try the default */
    if (hw_module_exists(path, sizeof(path), name, "default") == 0) {
        goto found;
    }

    return -ENOENT;

found:
    /* load the module, if this fails, we're doomed, and we should not try
     * to load a different variant. */
    return load(class_id, path, module);
}


// id: 输入参数， 表示要加载的硬件抽象层模块ID；
// module : 输出参数， 如果加载成功， 那么它指向一个自定义的硬件抽象层模块结构体
// return: 0 == success, <0 == error and *pHmi == NULL
// 加载硬件抽象层模块
int hw_get_module(const char *id, const struct hw_module_t **module)
{
    return hw_get_module_by_class(id, NULL, module);
}

```

`hardware.h`
```h
// hardware\libhardware\include\hardware\hardware.h

#ifndef ANDROID_INCLUDE_HARDWARE_HARDWARE_H
#define ANDROID_INCLUDE_HARDWARE_HARDWARE_H

#include <stdint.h>
#include <sys/cdefs.h>

#include <cutils/native_handle.h>
#include <system/graphics.h>

__BEGIN_DECLS

/*
 * Value for the hw_module_t.tag field
 */

#define MAKE_TAG_CONSTANT(A,B,C,D) (((A) << 24) | ((B) << 16) | ((C) << 8) | (D))

#define HARDWARE_MODULE_TAG MAKE_TAG_CONSTANT('H', 'W', 'M', 'T')
#define HARDWARE_DEVICE_TAG MAKE_TAG_CONSTANT('H', 'W', 'D', 'T')

#define HARDWARE_MAKE_API_VERSION(maj,min) \
            ((((maj) & 0xff) << 8) | ((min) & 0xff))

#define HARDWARE_MAKE_API_VERSION_2(maj,min,hdr) \
            ((((maj) & 0xff) << 24) | (((min) & 0xff) << 16) | ((hdr) & 0xffff))
#define HARDWARE_API_VERSION_2_MAJ_MIN_MASK 0xffff0000
#define HARDWARE_API_VERSION_2_HEADER_MASK  0x0000ffff


/*
 * The current HAL API version.
 *
 * All module implementations must set the hw_module_t.hal_api_version field
 * to this value when declaring the module with HAL_MODULE_INFO_SYM.
 *
 * Note that previous implementations have always set this field to 0.
 * Therefore, libhardware HAL API will always consider versions 0.0 and 1.0
 * to be 100% binary compatible.
 *
 */
#define HARDWARE_HAL_API_VERSION HARDWARE_MAKE_API_VERSION(1, 0)

/*
 * Helper macros for module implementors.
 *
 * The derived modules should provide convenience macros for supported
 * versions so that implementations can explicitly specify module/device
 * versions at definition time.
 *
 * Use this macro to set the hw_module_t.module_api_version field.
 */
#define HARDWARE_MODULE_API_VERSION(maj,min) HARDWARE_MAKE_API_VERSION(maj,min)
#define HARDWARE_MODULE_API_VERSION_2(maj,min,hdr) HARDWARE_MAKE_API_VERSION_2(maj,min,hdr)

/*
 * Use this macro to set the hw_device_t.version field
 */
#define HARDWARE_DEVICE_API_VERSION(maj,min) HARDWARE_MAKE_API_VERSION(maj,min)
#define HARDWARE_DEVICE_API_VERSION_2(maj,min,hdr) HARDWARE_MAKE_API_VERSION_2(maj,min,hdr)

struct hw_module_t;
struct hw_module_methods_t;
struct hw_device_t;

/**
 * Every hardware module must have a data structure named HAL_MODULE_INFO_SYM
 * and the fields of this data structure must begin with hw_module_t
 * followed by module specific information.
 */
/* 硬件抽象层中的每一个模块都必须自定义一个硬件抽象层模块结构体， 而且它的第一个成员变量的类型必须为 hw_module_t */
typedef struct hw_module_t {

    /* 成员变量 tag 的值必须设置为 HARDWARE_MODULE_TAG， 即设置为一个常量值（'H'＜＜24|'W'＜＜16|'M'＜＜8|'T'）, 用来标志这是一个硬件抽象层模块结构体 */
    /** tag must be initialized to HARDWARE_MODULE_TAG */
    uint32_t tag;

    /**
     * The API version of the implemented module. The module owner is
     * responsible for updating the version when a module interface has
     * changed.
     *
     * The derived modules such as gralloc and audio own and manage this field.
     * The module user must interpret the version field to decide whether or
     * not to inter-operate with the supplied module implementation.
     * For example, SurfaceFlinger is responsible for making sure that
     * it knows how to manage different versions of the gralloc-module API,
     * and AudioFlinger must know how to do the same for audio-module API.
     *
     * The module API version should include a major and a minor component.
     * For example, version 1.0 could be represented as 0x0100. This format
     * implies that versions 0x0100-0x01ff are all API-compatible.
     *
     * In the future, libhardware will expose a hw_get_module_version()
     * (or equivalent) function that will take minimum/maximum supported
     * versions as arguments and would be able to reject modules with
     * versions outside of the supplied range.
     */
    uint16_t module_api_version;
#define version_major module_api_version
    /**
     * version_major/version_minor defines are supplied here for temporary
     * source code compatibility. They will be removed in the next version.
     * ALL clients must convert to the new version format.
     */

    /**
     * The API version of the HAL module interface. This is meant to
     * version the hw_module_t, hw_module_methods_t, and hw_device_t
     * structures and definitions.
     *
     * The HAL interface owns this field. Module users/implementations
     * must NOT rely on this value for version information.
     *
     * Presently, 0 is the only valid value.
     */
    uint16_t hal_api_version;
#define version_minor hal_api_version

    /** Identifier of module */
    const char *id;

    /** Name of this module */
    const char *name;

    /** Author/owner/implementor of the module */
    const char *author;

    /* 定义了一个硬件抽象层模块的操作方法列表 */
    /** Modules methods */
    struct hw_module_methods_t* methods;

    /*  保存加载硬件抽象层模块后得到的句柄值 */
    /*  加载硬件抽象层模块的过程实际上就是调用 dlopen 函数来加载与其对应的动态链接库文件的过程。 
        在调用 dlclose 函数来卸载这个硬件抽象层模块时， 要用到这个句柄值 
    */
    /** module's dso */
    void* dso;

#ifdef __LP64__
    uint64_t reserved[32-7];
#else
    /** padding to 128 bytes, reserved for future use */
    uint32_t reserved[32-7];
#endif

} hw_module_t;

typedef struct hw_module_methods_t {
    /** Open a specific device */
    /** 
    * @function: 打开硬件抽象层模块中的硬件设备 
    * @parameter:   * module : 要打开的硬件设备所在的模块 
                    * id : 要打开的硬件设备的ID 
                    * device : 一个输出参数，描述一个已经打开的硬件设备 
    */
    int (*open)(const struct hw_module_t* module, const char* id,
            struct hw_device_t** device);

} hw_module_methods_t;

/**
 * Every device data structure must begin with hw_device_t
 * followed by module specific public methods and attributes.
 */
/**
 * 硬件抽象层模块中的每一个硬件设备都必须自定义一个硬件设备结构体， 
 * 而且它的第一个成员变量的类型必须为hw_device_t 
 */
typedef struct hw_device_t {
    /** tag must be initialized to HARDWARE_DEVICE_TAG */
    /* * tag 必须 == HARDWARE_DEVICE_TAG，即设置为一个常量值（'H'＜＜24|'W'＜＜16|'D'＜＜8|'T'）, * 用来标志这是一个硬件抽象层中的硬件设备结构体 */
    uint32_t tag;

    /**
     * Version of the module-specific device API. This value is used by
     * the derived-module user to manage different device implementations.
     *
     * The module user is responsible for checking the module_api_version
     * and device version fields to ensure that the user is capable of
     * communicating with the specific module implementation.
     *
     * One module can support multiple devices with different versions. This
     * can be useful when a device interface changes in an incompatible way
     * but it is still necessary to support older implementations at the same
     * time. One such example is the Camera 2.0 API.
     *
     * This field is interpreted by the module user and is ignored by the
     * HAL interface itself.
     */
    uint32_t version;

    /** reference to the module this device belongs to */
    struct hw_module_t* module;

    /** padding reserved for future use */
#ifdef __LP64__
    uint64_t reserved[12];
#else
    uint32_t reserved[12];
#endif

    /** Close this device */
    /* 关闭一个硬件设备 */
    int (*close)(struct hw_device_t* device);

} hw_device_t;

/**
 * Name of the hal_module_info
 */
/* 硬件抽象层中的每一个模块都必须存在一个导出符号 HAL_MODULE_IFNO_SYM， 即“HMI”， 它指向一个自定义的硬件抽象层模块结构体 */
#define HAL_MODULE_INFO_SYM         HMI

/**
 * Name of the hal_module_info as a string
 */
#define HAL_MODULE_INFO_SYM_AS_STR  "HMI"

/**
 * Get the module info associated with a module by id.
 *
 * @return: 0 == success, <0 == error and *module == NULL
 */
int hw_get_module(const char *id, const struct hw_module_t **module);

/**
 * Get the module info associated with a module instance by class 'class_id'
 * and instance 'inst'.
 *
 * Some modules types necessitate multiple instances. For example audio supports
 * multiple concurrent interfaces and thus 'audio' is the module class
 * and 'primary' or 'a2dp' are module interfaces. This implies that the files
 * providing these modules would be named audio.primary.<variant>.so and
 * audio.a2dp.<variant>.so
 *
 * @return: 0 == success, <0 == error and *module == NULL
 */
int hw_get_module_by_class(const char *class_id, const char *inst,
                           const struct hw_module_t **module);

__END_DECLS

#endif  /* ANDROID_INCLUDE_HARDWARE_HARDWARE_H */

```

`ueventd.rc`
```
# system\core\rootdir\ueventd.rc

subsystem adf
	devname uevent_devname

/dev/null                 0666   root       root
/dev/zero                 0666   root       root
/dev/full                 0666   root       root
/dev/ptmx                 0666   root       root
/dev/tty                  0666   root       root
/dev/random               0666   root       root
/dev/urandom              0666   root       root
# Make HW RNG readable by group system to let EntropyMixer read it.
/dev/hw_random            0440   root       system
/dev/ashmem               0666   root       root
/dev/binder               0666   root       root

# 所有的用户均可以访问设备文件/dev/freg， 就可以 打开设备文件/dev/freg 和 读写它的内容
/dev/freg                 0666   root       root

...
```

# jni

`Android.mk`
```mk
# frameworks/base/Android.mk

## READ ME: ########################################################
##
## When updating this list of aidl files, consider if that aidl is
## part of the SDK API.  If it is, also add it to the list below that
## is preprocessed and distributed with the SDK.  This list should
## not contain any aidl files for parcelables, but the one below should
## if you intend for 3rd parties to be able to send those objects
## across process boundaries.
##
## READ ME: ########################################################
LOCAL_SRC_FILES += \
    core/java/android/accessibilityservice/IAccessibilityServiceConnection.aidl \
    #...
    core/java/android/net/IThrottleManager.aidl \
    core/java/android/nfc/IP2pTarget.aidl \
    core/java/android/os/IVibratorService.aidl \
    # 将需要的添加到编译脚本文件中
    core/java/android/os/IFregService.aidl \
    core/java/android/service/urlrenderer/IUrlRendererService.aidl \
    #...
    voip/java/android/net/sip/ISipService.aidl
#
```

`FregService.java`
```java
// frameworks\base\services\java\com\android\server\FregService.java

package com.android.server;

import android.content.Context;
import android.os.IFregService;
import android.util.Slog;

// 硬件访问服务 FregService 继承了 IFregService.Stub 类
public class FregService extends IFregService.Stub 
{
    private static final String TAG = "FregService";

    private int mPtr = 0;

    FregService() 
    {
        // 调用 JNI 方法 init_native 来打开虚拟硬件设备 freg ，
        // 并且获得它的一个句柄值， 保存在成员变量 mPtr 中
        // 这个句柄值实际上是指向虚拟硬件设备freg在硬件抽象层中的一个设备对象
        mPtr = init_native();

        if(mPtr == 0) 
        {
            Slog.e(TAG, "Failed to initialize freg service.");
        }
    }

    public void setVal(int val) 
    {
        if(mPtr == 0) 
        {
            Slog.e(TAG, "Freg service is not initialized.");
            return;
        }
        // 调用 JNI 方法 setVal_native 来写虚拟硬件设备 freg 的寄存器 val
        setVal_native(mPtr, val);
    }    

    public int getVal() 
    {
        if(mPtr == 0) 
        {
            Slog.e(TAG, "Freg service is not initialized.");
            return 0;
        }

        //调用 JNI 方法 getVal_native 来读虚拟硬件设备 freg 的寄存器 val
        return getVal_native(mPtr);
    }

    private static native int init_native();
    private static native void setVal_native(int ptr, int val);
    private static native int getVal_native(int ptr);
};
```

`IFregService.aidl`
```aidl
// frameworks\base\core\java\android\os\IFregService.aidl

package android.os;

interface IFregService 
{
    // 往虚拟硬件设备freg的寄存器val中写入一个整数
    void setVal(int val);
    // 从虚拟硬件设备freg的寄存器val中读出一个整数
    int getVal();
}
```

`com_android_server_FregService.cpp`
```cpp
// frameworks\base\services\jni\com_android_server_FregService.cpp

#define LOG_TAG "FregServiceJNI"

#include "jni.h"
#include "JNIHelp.h"
#include "android_runtime/AndroidRuntime.h"

#include <utils/misc.h>
#include <utils/Log.h>
#include <hardware/hardware.h>
#include <hardware/freg.h>

#include <stdio.h>

namespace android
{
    //设置虚拟硬件设备 freg 的寄存器的值
    static void freg_setVal(JNIEnv* env, jobject clazz, jint ptr, jint value) 
    {
        int val = value;

        //将参数 ptr 转换为 freg_device_t 结构体变量
        freg_device_t* device = (freg_device_t*)ptr;
        if(!device)
        {
            LOGE("Device freg is not open.");
            return;
        }

        LOGI("Set value %d to device freg.", val);

        device->set_val(device, val);
    }

    //读取虚拟硬件设备freg的寄存器的值
    static jint freg_getVal(JNIEnv* env, jobject clazz, jint ptr)
    {
        int val = 0;

        //将传输ptr转换为 freg_device_t 结构体变量
        freg_device_t* device = (freg_device_t*)ptr;
        if(!device) 
        {
            LOGE("Device freg is not open.");
            return 0;
        }

        device->get_val(device, &val);

        LOGI("Get value %d from device freg.", val);

        return val;
    }

    //打开虚拟硬件设备freg
    static inline int freg_device_open(const hw_module_t* module, 
                                       struct freg_device_t** device) 
    {
        return module->methods->open(module, 
                                     FREG_HARDWARE_DEVICE_ID, 
                                     (struct hw_device_t**)device);
    }

    //初始化虚拟硬件设备freg
    static jint freg_init(JNIEnv* env, jclass clazz)
    {
        freg_module_t* module;
        freg_device_t* device;

        LOGI("Initializing HAL stub freg......");

        //加载硬件抽象层模块freg
        // 根据 FREG_HARDWARE_MODULE_ID 来加载 Android 硬件抽象层模块 freg
        if(hw_get_module(FREG_HARDWARE_MODULE_ID, 
                         (const struct hw_module_t**)&module) == 0) 
        {
            LOGI("Device freg found.");
            //打开虚拟硬件设freg , 打开设备ID为 FREG_HARDWARE_DEVICE_ID 的硬件设备
            if(freg_device_open(&(module->common), &device) == 0) 
            {
                LOGI("Device freg is open.");
                //将freg_device_t 接口转换为整型句柄值值返回
                return (jint)device;
            }

            LOGE("Failed to open device freg.");
            return 0;
        }

        LOGE("Failed to get HAL stub freg.");

        return 0;
    }

    // java本地接口方法表
    // 把JNI方法表method_table注册到Java虚拟机
    // 将函数freg_init、 freg_setVal和 freg_getVal的JNI方法注册
    // 为init_native、 setVal_native和 getVal_native
    static const JNINativeMethod method_table[] = {
        {"init_native", "()I", (void*)freg_init},
        {"setVal_native", "(II)V", (void*)freg_setVal},
        {"getVal_native", "(I)I", (void*)freg_getVal},
    };

    //注册java本地接口方法
    int register_android_server_FregService(JNIEnv *env) 
    {
        return jniRegisterNativeMethods(env, 
                                        "com/android/server/FregService", 
                                        method_table, 
                                        NELEM(method_table));
    }

};
```

## linux driver module introduction

`led_cdev.c`
```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/cdev.h>
#include <linux/fs.h>
#include <linux/uaccess.h>
#include <linux/io.h>

#define DEV_NAME            "led_chrdev"
#define DEV_CNT                 (3)

//在内核中，dev_t用来表示设备编号，dev_t是一个32位的数，其中，高12位表示主设备号，低20位表示次设备号
static dev_t devno;

//
struct class *led_chrdev_class;

/**
    内核用struct cdev结构体来描述一个字符设备

        struct cdev {
            struct kobject kobj;    //内嵌的内核对象，通过它将设备统一加入到“Linux设备驱动模型”中管理
            struct module *owner;   //字符设备驱动程序所在的内核模块对象的指针
            const struct file_operations *ops;
            struct list_head list;
            dev_t dev;              //字符设备的设备号，有主设备和次设备号构成
            unsigned int count;     //属于同一主设备好的次设备号的个数，用于表示设备驱动程序控制的实际同类设备的数量
        };
*/
struct led_chrdev {
    struct cdev dev;

    //数据寄存器虚拟地址指针
    unsigned int __iomem *va_dr;
    //输入输出方向寄存器虚拟地址指针
    unsigned int __iomem *va_gdir;
    //端口复用寄存器虚拟地址指针
    unsigned int __iomem *va_iomuxc_mux;
    //时钟寄存器虚拟地址指针
    unsigned int __iomem *va_ccm_ccgrx;
    //电气属性寄存器虚拟地址指针
    unsigned int __iomem *va_iomux_pad;

    //装载数据寄存器（物理地址）的变量
    unsigned long pa_dr;
    //装载输出方向寄存器（物理地址）的变量
    unsigned long pa_gdir;
    //装载端口复用寄存器（物理地址）的变量
    unsigned long pa_iomuxc_mux;
    //装载时钟寄存器（物理地址）的变量
    unsigned long pa_ccm_ccgrx;
    //装载电气属性寄存器（物理地址）的变量
    unsigned long pa_iomux_pad;

    //LED的引脚
    unsigned int led_pin;
    //时钟偏移地址（相对于CCM_CCGRx）
    unsigned int clock_offset;
};

/**
    //内核使用inode结构体在内核内部表示一个文件
    //我们可以使用多个file文件结构表示同一个文件的多个文件描述符
    //但此时， 所有的这些file文件结构全部都必须只能指向一个inode结构体
    struct inode {
        //表示设备文件的结点，这个域实际上包含了设备号
        dev_t i_rdev;

        //当inode结点指向一个字符设备文件时，此域为一个指向inode结构的指针
        struct cdev *i_cdev;
    }
*/

/**
    //表示每个打开的文件
    struct file {
        const struct file_operations *f_op;

        //在驱动程序中，通常用于指向描述设备的结构体
        void *private_data;
    };
*/

/* 打开RGB LED设备函数 */
static int led_chrdev_open(struct inode *inode, struct file *filp)
{

    unsigned int val = 0;

    //container_of(ptr, type, member):
    //  ptr:结构体变量中某个成员的地址
    //  type:结构体类型
    //  member:该结构体变量的具体名字
    //  return:结构体type的首地址

    //获取led_chrdev结构体的首地址
    struct led_chrdev *led_cdev =
        (struct led_chrdev *)container_of(inode->i_cdev, struct led_chrdev, dev);

    //一般很多的linux驱动都会将文件的私有数据private_data指向设备结构体，其保存了用户自定义设备结构体的地址
    //自定义结构体的地址被保存在private_data后，可以通过读、写等操作通过该私有数据去访问设备结构体中的成员
    filp->private_data =
        container_of(inode->i_cdev, struct led_chrdev, dev);

    printk("open\n");

    /* 实现地址映射 */
    led_cdev->va_dr = ioremap(led_cdev->pa_dr, 4);                  //数据寄存器映射，将led_cdev->va_dr指针指向映射后的虚拟地址起始处，这段地址大小为4个字节
    led_cdev->va_gdir = ioremap(led_cdev->pa_gdir, 4);              //方向寄存器映射
    led_cdev->va_iomuxc_mux = ioremap(led_cdev->pa_iomuxc_mux, 4);  //端口复用功能寄存器映射
    led_cdev->va_ccm_ccgrx = ioremap(led_cdev->pa_ccm_ccgrx, 4);    //时钟控制寄存器映射
    led_cdev->va_iomux_pad = ioremap(led_cdev->pa_iomux_pad, 4);    //电气属性配置寄存器映射


    //一般我们访问某个地址时都是先将该地址的数据读取到一个变量中然后修改该变量，最后再将该变量写入到原来的地址当中
    /* 配置寄存器 */
    val = ioread32(led_cdev->va_ccm_ccgrx); //间接读取寄存器中的数据
    val &= ~(3 << led_cdev->clock_offset);
    val |= (3 << led_cdev->clock_offset);   //置位对应的时钟位
    iowrite32(val, led_cdev->va_ccm_ccgrx); //重新将数据写入寄存器

    iowrite32(5, led_cdev->va_iomuxc_mux);  //复用位普通I/O口

    iowrite32(0x1F838, led_cdev->va_iomux_pad);

    val = ioread32(led_cdev->va_gdir);
    val &= ~(1 << led_cdev->led_pin);
    val |= (1 << led_cdev->led_pin);
    iowrite32(val, led_cdev->va_gdir);      //配置位输出模式

    val = ioread32(led_cdev->va_dr);
    val |= (0x01 << led_cdev->led_pin);
    iowrite32(val, led_cdev->va_dr);        //输出高电平

    return 0;
}


static int led_chrdev_release(struct inode *inode, struct file *filp)
{
    struct led_chrdev *led_cdev =
        (struct led_chrdev *)container_of(inode->i_cdev, struct led_chrdev, dev);

    /* 释放ioremap后的虚拟地址空间 */
    iounmap(led_cdev->va_dr);
    iounmap(led_cdev->va_gdir);
    iounmap(led_cdev->va_iomuxc_mux);
    iounmap(led_cdev->va_ccm_ccgrx);
    iounmap(led_cdev->va_iomux_pad);
    return 0
}

static ssize_t led_chrdev_write(struct file *filp, const char __user * buf,
                            size_t count, loff_t * ppos)
{
    unsigned long val = 0;
    unsigned long ret = 0;

    int tmp = count;

    //将用户空间缓存区复制到内核空间
    kstrtoul_from_user(buf, tmp, 10, &ret);

    //文件的私有数据地址赋给led_cdev结构体指针
    struct led_chrdev *led_cdev = (struct led_chrdev *)filp->private_data;

    //间接读取数据寄存器中的数据
    val = ioread32(led_cdev->va_dr);

    if (ret == 0)
            val &= ~(0x01 << led_cdev->led_pin);
    else
            val |= (0x01 << led_cdev->led_pin);

    //将数据重新写入寄存器中,控制LED亮灭
    iowrite32(val, led_cdev->va_dr);
    *ppos += tmp;
    return tmp;
}


/**
    struct file_operations {
        struct module *owner;
        loff_t (*llseek) (struct file *, loff_t, int);

        //__user用于修饰变量，表明该变量所在的地址空间是用户空间的
        ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);

        ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
        long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);

        //一般用于硬件的初始化
        int (*open) (struct inode *, struct file *)

        int (*release) (struct inode *, struct file *);
    };
*/
static struct file_operations led_chrdev_fops = {
    .owner = THIS_MODULE,
    .open = led_chrdev_open,
    .release = led_chrdev_release,
    .write = led_chrdev_write,
};

static struct led_chrdev led_cdev[DEV_CNT] = {
    {.pa_dr = 0x0209C000,.pa_gdir = 0x0209C004,.pa_iomuxc_mux =
     0x20E006C,.pa_ccm_ccgrx = 0x20C406C,.pa_iomux_pad =
     0x20E02F8,.led_pin = 4,.clock_offset = 26},
    {.pa_dr = 0x20A8000,.pa_gdir = 0x20A8004,.pa_iomuxc_mux =
     0x20E01E0,.pa_ccm_ccgrx = 0x20C4074,.pa_iomux_pad =
     0x20E046C,.led_pin = 20,.clock_offset = 12},
    {.pa_dr = 0x20A8000,.pa_gdir = 0x20A8004,.pa_iomuxc_mux =
     0x20E01DC,.pa_ccm_ccgrx = 0x20C4074,.pa_iomux_pad =
     0x20E0468,.led_pin = 19,.clock_offset = 12},
};

/**
    用户空间open函数
        在虚拟文件系统VFS中的查找对应与字符设备对应 struct inode 节点
        遍历散列表cdev_map，根据inod节点中的 cdev_t设备号找到cdev对象
        创建struct file对象，初始化struct file对象，将 struct file对象中的 file_operations成员指向 struct cdev对象中的 file_operations成员
*/
/**
    调用alloc_chrdev_region()函数向系统动态申请一个未被占用的设备号，
    使用alloc_chrdev_region()相比较于register_chrdev_region()的好处在于不必自己费时间去查看那些是未被占用的设备号，避免了设备号重复问题；
    调用class_create()函数创建一个RGB灯的设备类；
    分别给三个LED建立其对应的字符设备结构体cdev和led_chrdev_fops的关联
    初始化字符设备结构体，最后注册
    在驱动使用class_create创建设备类、在类的下面device_create创建设备节点
*/
static __init int led_chrdev_init(void)
{
    int i = 0;
    dev_t cur_dev;
    printk("led chrdev init\n");

    //向动态申请一个设备号
    //参数：
    //  指向dev_t类型数据的指针变量，用于存放分配到的设备编号的起始值
    //  次设备号的起始值，通常情况下，设置为0
    //  用于指定需要分配的设备编号的个数以及设备的名称
    alloc_chrdev_region(&devno, 0, DEV_CNT, DEV_NAME);

    //创建设备类
    led_chrdev_class = class_create(THIS_MODULE, "led_chrdev");

    for (; i < DEV_CNT; i++) {

            //初始化
            //绑定led_cdev与led_chrdev_fops
            cdev_init(&led_cdev[i].dev, &led_chrdev_fops);

            led_cdev[i].dev.owner = THIS_MODULE;
            cur_dev = MKDEV(MAJOR(devno), MINOR(devno) + i);

            //设备注册
            cdev_add(&led_cdev[i].dev, cur_dev, 1);

            //设备节点的创建
            //  led_chrdev_class：指向这个设备应该注册到的struct类的指针
            //  DEV_NAME "%d"：输入设备名称
            device_create(led_chrdev_class, NULL, cur_dev, NULL, DEV_NAME "%d", i);
    }
    return 0;
}

module_init(led_chrdev_init);

/**
    调用device_destroy()函数用于从linux内核系统设备驱动程序模型中移除一个设备
    并删除/sys/devices/virtual目录下对应的设备目录及/dev/目录下对应的设备文件；
    调用cdev_del()函数来释放散列表中的对象以及cdev结构本身；
    释放被占用的设备号以及删除设备类。
*/
static __exit void led_chrdev_exit(void)
{
    int i;
    dev_t cur_dev;
    printk("led chrdev exit\n");

    for (i = 0; i < DEV_CNT; i++) {

            cur_dev = MKDEV(MAJOR(devno), MINOR(devno) + i);

            device_destroy(led_chrdev_class, cur_dev);

            cdev_del(&led_cdev[i].dev);

    }
    unregister_chrdev_region(devno, DEV_CNT);
    class_destroy(led_chrdev_class);
}

module_exit(led_chrdev_exit);

MODULE_AUTHOR("embedfire");
MODULE_LICENSE("GPL");
```
