---
layout: post
title: Linux Dev Tips
tags: [Tutorial,Jekyll]
author: laoYao
---

# /etc/apt/sources.list

```
# ubuntu 20.04

deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse

# deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse



# 18.04

deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

# deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse



# 14.04

deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse

# deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
```

# modify cpu frequency

```
查看可用频率
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors

当前频率
cat /sys/devices/system/cpu/cpu0/cpufreq/cpuinfo_cur_freq

当前负载
top -d 1

设置频率
echo userspace > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
echo 198000 > /sys/devices/system/cpu/cpu0/cpufreq/scaling_setspeed

```

## add sys sign to app

    # shell in ubuntu
    openssl pkcs8 -in platform.pk8 -inform DER -outform PEM -out shared.priv.pem -nocrypt

    openssl pkcs12 -export -in platform.x509.pem -inkey shared.priv.pem -out shared.pk12 -name androiddebugkey

    keytool -importkeystore -deststorepass android -destkeypass android -destkeystore debug.keystore -srckeystore shared.pk12 -srcstoretype PKCS12 -srcstorepass android -alias androiddebugkey


## how to debug serial via AVD device

    emulator.exe @Nexus5 -qemu -serial COM2



screenshot
```
code . --user-data-dir /root/.vscode

adb shell screencap -p /sdcard/01.png

adb shell cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_frequencies
echo 480000  > /sys/devices/system/cpu/cpu0/cpufreq/scaling_max_freq

```


```
// editor
sudo gedit

// file explorer
sudo nautilus

// sudo vscode
// You are trying to start Visual Studio Code as a super user which isn't recommended. If this was intended, please specify an alternate user data directory using the `--user-data-dir` argument.
code --user-data-dir="~/.vscode" .
```

---

todo: disable terminal line wap

---

allwinner build error @docker env:
	USER: unbound variable
	
solve:
```
	export USER=$(whoami)
```

---

```
sudo vim /etc/systemd/logind.conf
```

close not sleep need modify to
```
#HandleLidSwitch=suspend  --->  HandleLidSwitch=ignore
```


> 2021-05-11

problem:
```
zhsh@zhsh-XiaomiBook:~$ adb devices
List of devices attached
c4aede4a0504	no permissions (missing udev rules? user is in the plugdev group); see [http://developer.android.com/tools/device.html]
```

solve:
```
touch /etc/udev/rules.d/51-android.rules
vi /etc/udev/rules.d/51-android.rules
```

add:
```
SUBSYSTEM=="usb",ENV{DEVTYPE}=="usb_device",MODE="0666"
```

@terminal
```
sudo usermod -aG plugdev $LOGNAME
```

if still unavailable
```
sudo apt-get install android-sdk-platform-tools-common
```

after all, must reconnect usb device!!!

---

https://hub.FastGit.org <---> https://github.com

---

solve zip file by windows show wrong
```
unzip -O CP936 desktop.zip
```

# forever hide navigation bar

@frameworks/base/services/core/java/com/android/server/policy/PhoneWindowManager.java,
before:
```
        String navBarOverride = SystemProperties.get("qemu.hw.mainkeys");
        if ("1".equals(navBarOverride)) {
            mHasNavigationBar = false;
        } else if ("0".equals(navBarOverride)) {
            mHasNavigationBar = true;
        }
```

after
```
        String navBarOverride = SystemProperties.get("qemu.hw.mainkeys");
        if ("1".equals(navBarOverride)) {
            mHasNavigationBar = false;
        } else if ("0".equals(navBarOverride)) {
            mHasNavigationBar = false;
        } else {
            mHasNavigationBar = false;
        }
```

---

forever hide status bar

@frameworks/base/packages/SystemUI/src/com/android/systemui/statusbar/phone/PhoneStatusBarView.java

before:
```
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        boolean barConsumedEvent = mBar.interceptTouchEvent(event);

        if (DEBUG_GESTURES) {
            if (event.getActionMasked() != MotionEvent.ACTION_MOVE) {
                EventLog.writeEvent(EventLogTags.SYSUI_PANELBAR_TOUCH,
                       event.getActionMasked(), (int) event.getX(), (int) event.getY(),
                        barConsumedEvent ? 1 : 0);
            }
        }

        return barConsumedEvent || super.onTouchEvent(event);
    }
    
    @Override
    public boolean onInterceptTouchEvent(MotionEvent event) {
        return mBar.interceptTouchEvent(event) || super.onInterceptTouchEvent(event);
    }
```

after:
```
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        //boolean barConsumedEvent = mBar.interceptTouchEvent(event);

        //if (DEBUG_GESTURES) {
        //    if (event.getActionMasked() != MotionEvent.ACTION_MOVE) {
        //        EventLog.writeEvent(EventLogTags.SYSUI_PANELBAR_TOUCH,
        //               event.getActionMasked(), (int) event.getX(), (int) event.getY(),
        //                barConsumedEvent ? 1 : 0);
        //    }
        //}

        //return barConsumedEvent || super.onTouchEvent(event);
        
        return false;
    }
    
    @Override
    public boolean onInterceptTouchEvent(MotionEvent event) {
        //return mBar.interceptTouchEvent(event) || super.onInterceptTouchEvent(event);
        return false;
    }
```

and @frameworks/base/core/res/res/values/dimens.xml

before:
```
    <!-- Height of the status bar -->
    <dimen name="status_bar_height">24dp</dimen>
```

```
    <!-- Height of the status bar -->
    <dimen name="status_bar_height">0dp</dimen>
```

and @rameworks/base/packages/SystemUI/src/com/android/systemui/statusbar/phone/PhoneStatusBar.java

before:
```
    private void addStatusBarWindow() {
        makeStatusBarView();
        mStatusBarWindowManager = new StatusBarWindowManager(mContext);
        mRemoteInputController = new RemoteInputController(mStatusBarWindowManager,
                mHeadsUpManager);
        mStatusBarWindowManager.add(mStatusBarWindow, getStatusBarHeight());
    }
```

after:
```
    private void addStatusBarWindow() {
        makeStatusBarView();
        mStatusBarWindowManager = new StatusBarWindowManager(mContext);
        mRemoteInputController = new RemoteInputController(mStatusBarWindowManager,
                mHeadsUpManager);
        mStatusBarWindowManager.add(mStatusBarWindow, getStatusBarHeight());
        mStatusBarView.setVisibility(View.GONE);
    }
```

---

launch as Launcher

before:
```
        <activity
            android:name="com.licheedev.serialtool.activity.MainActivity"
            android:screenOrientation="landscape"
            android:theme="@style/AppThemeNoActionBar"
            android:windowSoftInputMode="stateHidden|stateUnchanged">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
```

after:
```
        <activity
            android:name="com.licheedev.serialtool.activity.MainActivity"
            android:launchMode="singleTask"
            android:screenOrientation="landscape"
            android:theme="@style/AppThemeNoActionBar"
            android:windowSoftInputMode="stateHidden|stateUnchanged">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.HOME" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
```   

---

> 2021-05-13

vertical seekbar
```

import android.content.Context;
import android.graphics.Canvas;
import android.util.AttributeSet;
import android.view.MotionEvent;

public class VerticalSeekBar extends androidx.appcompat.widget.AppCompatSeekBar {

    public VerticalSeekBar(Context context) {
        super(context);
    }

    public VerticalSeekBar(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }

    public VerticalSeekBar(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(h, w, oldh, oldw);
    }

    @Override
    protected synchronized void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(heightMeasureSpec, widthMeasureSpec);
        setMeasuredDimension(getMeasuredHeight(), getMeasuredWidth());
    }

    protected void onDraw(Canvas c) {
        c.rotate(-90);
        c.translate(-getHeight(), 0);

        super.onDraw(c);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        if (!isEnabled()) {
            return false;
        }

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
            case MotionEvent.ACTION_MOVE:
            case MotionEvent.ACTION_UP:
                setProgress(getMax() - (int) (getMax() * event.getY() / getHeight()));
                onSizeChanged(getWidth(), getHeight(), 0, 0);
                break;

            case MotionEvent.ACTION_CANCEL:
                break;
        }
        return true;
    }
}
```

---

change brightness in app
```
	// set value
        Settings.System.putInt(
                getContentResolver(),
                Settings.System.SCREEN_BRIGHTNESS,
                255 * progress / 100
        );
        
        	
        try {
      	    // set mode
            Settings.System.putInt(
                    getContentResolver(),
                    Settings.System.SCREEN_BRIGHTNESS_MODE,
                    Settings.System.SCREEN_BRIGHTNESS_MODE_MANUAL
            );

	    // get value
            int brightness = Settings.System.getInt(
                    getContentResolver(),
                    Settings.System.SCREEN_BRIGHTNESS
            );
            mPbScreenLight.setProgress(brightness * 100 / 255);
        } catch (Exception exception) {
        }
```        

---

todo: android sleep state??

todo: how android reboot??

todo: bootlogo.bmp and bootanimation.zip

---

---
* 编写 Linux 设备驱动要求工程师有非常好的硬件基础, 懂得 SRAM、Flash、SDRAM、磁盘的读写方式, UART、I2C、USB 等设备的接口以及轮询、中断、DMA 的原理, PCI 总线的工作方式以及 CPU 的内存管理单元(MMU)等。
* 编写 Linux 设备驱动要求工程师有非常好的 C 语言基础,能灵活地运用 C 语言的结构体、指针、函数指针及内存动态申请和释放等。
* 编写 Linux 设备驱动要求工程师有一定的 Linux 内核基础,虽然并不要求工程师对内核各个部分有深入的研究,但至少要明白驱动与内核的接口。尤其是对于块设备、网络设备、Flash 设备、串口设备等复杂设备,内核定义的驱动体系结构本身就非常复杂。
* 编写 Linux 设备驱动要求工程师有非常好的多任务并发控制和同步的基础,因为在驱动中会大量使用自旋锁、互斥、信号量、等待队列等并发与同步机制。

---

第〇周：4.21-4.27车间/测试实习

第一周：4.27电脑环境搭建，4.28-4.30/5.6-5.8熟悉开发板，Android串口通信

第二周：源代码编译，全屏，启动图，启动动画修改 5.10-5.12

第三周：Linux驱动开发基本了解，Android开发分层基本了解 5.13-5.15

第四周：C语言基本了解，电路图基本原理

---
