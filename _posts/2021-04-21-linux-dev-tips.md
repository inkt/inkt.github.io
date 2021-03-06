---
layout: post
title: Linux Dev Tips
tags: [Tutorial,Jekyll]
author: laoYao
---


screenshot
```
code . --user-data-dir /root/.vscode

adb shell screencap -p /sdcard/01.png

adb shell cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_frequencies
echo 480000  > /sys/devices/system/cpu/cpu0/cpufreq/scaling_max_freq

docker start 7daa9ce68378 && docker exec -it 7daa9ce68378 bin/bash
```


> build cmd

```shell
cd /root/workspace/allwinner/android && source build/envsetup.sh && lunch 24 && extract-bsp && export USER=$(whoami) && make -j8 && pack
```

> 2021-5-10

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

before build space left 176.1G

after lichee built left 174.8G

---

before build android left 174.7G  @13:04
```
#### make completed successfully (01:27:58 (hh:mm:ss)) ####
```

build end @14:31 left 142.3G

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

build all commands:

root@linux
```
	apt-get update
	apt install docker.io
	docker
	mkdir -p /root/workspace/allwinner
	cd /root/workspace/allwinner
	docker load -i /home/zhsh/Desktop/img/ubuntu14_04lts_for_a40i_android7.tar
	docker images
	docker tag 21f1a026ea52 ubuntu14_04lts_for_a40i_android7:1.0.0
	docker images
	docker run -it --privileged=true --cap-add=SYS_ADMIN -v /root/workspace/allwinner:/root/workspace/allwinner -p 10022:22 21f1a026ea52 bin/bash
	apt install p7zip-full
	mv /home/zhsh/Desktop/img/android.tar.gz ./android.tar.gz
	mv /home/zhsh/Desktop/img/lichee.tar.gz ./lichee.tar.gz
	tar xvf lichee.tar.gz
	tar xvf android.tar.gz
	mv ./lichee.tar.gz /home/zhsh/Desktop/img/lichee.tar.gz
	mv ./android.tar.gz /home/zhsh/Desktop/img/android.tar.gz
	docker ps -a
	docker start dc415fb193ff
	docker exec -it dc415fb193ff bin/bash
```
	
root@docker
```
	cd /root/workspace/allwinner
	cd lichee
	./build.sh config
	./build.sh
	cd ../android
	source build/envsetup.sh
	lunch 24
	extract-bsp
	export USER=$(whoami)
	make -j8 && pack
```

---

```
sudo vim /etc/systemd/logind.conf
```

close not sleep need modify to
```
#HandleLidSwitch=suspend  --->  HandleLidSwitch=ignore
```


register firefox account for sync
user: 		inkt@qq.com 
password:	---

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
---

problem: libc6-dev : Depends: libc6 (= 2.31-0ubuntu9) but 2.31-0ubuntu9.2 is to be ...

solve: use aliyun or tringhua mirror, modify: /etc/apt/sources.list
```
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
# deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse

```

---

> 2021-05-12

bootlogo@
lichee/tools/pack/chips/sun8iw11p1/configs/OKA40i_C/bootlogo.bmp

bootanimation@
/android/device/softwinner/a40-OKA40i_C/media/bootanimation.zip

see@ https://blog.csdn.net/linziyo/article/details/109534771

---

forever hide navigation bar

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

> 2021-05-14

TP-WDN5200H @ Ubuntu
```
$ sudo apt update
$ sudo apt install build-essential git dkms
$ git clone https://github.com/brektrou/rtl8821CU.git
$ cd rtl8821CU
$ chmod +x dkms-install.sh
$ sudo ./dkms-install.sh
$ sudo modprobe 8821cu
```

---
* ?????? Linux ??????????????????????????????????????????????????????, ?????? SRAM???Flash???SDRAM????????????????????????, UART???I2C???USB ??????????????????????????????????????????DMA ?????????, PCI ??????????????????????????? CPU ?????????????????????(MMU)??????
* ?????? Linux ?????????????????????????????????????????? C ????????????,?????????????????? C ??????????????????????????????????????????????????????????????????????????????
* ?????? Linux ??????????????????????????????????????? Linux ????????????,??????????????????????????????????????????????????????????????????,???????????????????????????????????????????????????????????????????????????????????????Flash ????????????????????????????????????,?????????????????????????????????????????????????????????
* ?????? Linux ?????????????????????????????????????????????????????????????????????????????????,?????????????????????????????????????????????????????????????????????????????????????????????????????????

---

????????????4.21-4.27??????/????????????

????????????4.27?????????????????????4.28-4.30/5.6-5.8??????????????????Android????????????

????????????????????????????????????????????????????????????????????? 5.10-5.12

????????????Linux???????????????????????????Android???????????????????????? 5.13-5.15

????????????C??????????????????????????????????????????

> 2021-05-23

---
docker-A40i
```
mkdir -p /root/workspace/allwinner
cd /root/workspace/allwinner
docker load -i /home/zhsh/Desktop/img/ubuntu14_04lts_for_a40i_android7.tar
docker images
docker tag 21f1a026ea52 ubuntu14_04lts_for_a40i_android7:1.0.0
docker images
docker run -it --privileged=true --cap-add=SYS_ADMIN -v /root/workspace/allwinner:/root/workspace/allwinner -p 10022:22 21f1a026ea52 bin/bash
apt install p7zip-full
mv /home/zhsh/Desktop/img/android.tar.gz ./android.tar.gz
mv /home/zhsh/Desktop/img/lichee.tar.gz ./lichee.tar.gz
tar xvf lichee.tar.gz
tar xvf android.tar.gz
mv ./lichee.tar.gz /home/zhsh/Desktop/img/lichee.tar.gz
mv ./android.tar.gz /home/zhsh/Desktop/img/android.tar.gz
docker ps -a
docker start dc415fb193ff
docker exec -it dc415fb193ff bin/bash
mv /root/workspace/allwinner/lichee/tools/pack/sun8iw11p1_androidm_a40-OKA40i_C_uart0-1280x800.img /home/zhsh/Desktop
cat ~/.ssh/id_rsa.pub
ssh -T git@gitee.com
7z e Desktp.zip
unzip -O CP936 Desktp.zip
adb -s 20080411 shell pm list packages
adb -s 20080411 uninstall com.licheedev.serialtool
sudo nautilus
```
---

lineageos download
```
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
---

# 2021-05-18

```
start learn from bilibili
```

---