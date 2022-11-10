---
layout: post
title: Interesting Tips
tags: [Tutorial]
author: laoYao
---

# 图片查看器 @LTSC

```
@echo off&cd\&color 0a&cls

echo 恢复Win10照片查看器

reg add "HKLM\SOFTWARE\Microsoft\Windows Photo Viewer\Capabilities\FileAssociations" /v ".jpg" /t REG_SZ /d PhotoViewer.FileAssoc.Tiff /f

reg add "HKLM\SOFTWARE\Microsoft\Windows Photo Viewer\Capabilities\FileAssociations" /v ".jpeg" /t REG_SZ /d PhotoViewer.FileAssoc.Tiff /f

reg add "HKLM\SOFTWARE\Microsoft\Windows Photo Viewer\Capabilities\FileAssociations" /v ".bmp" /t REG_SZ /d PhotoViewer.FileAssoc.Tiff /f

reg add "HKLM\SOFTWARE\Microsoft\Windows Photo Viewer\Capabilities\FileAssociations" /v ".png" /t REG_SZ /d PhotoViewer.FileAssoc.Tiff /f

echo 请双击或右击图片，选择“照片查看器”即可

pause
```

# hide

```
Windows Registry Editor Version 5.00

; 删除我的电脑/此电脑里面的 3D对象
[-HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{0DB7E03F-FC29-4DC6-9020-FF41B59E513A}]
[-HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{0DB7E03F-FC29-4DC6-9020-FF41B59E513A}]

; 隐藏另存为里面的 3D对象
[-HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Explorer\FolderDescriptions\{31C0DD25-9439-4F12-BF41-7FF4EDA38722}\PropertyBag]
"ThisPCPolicy"="Hide"

;隐藏我的电脑/此电脑里面的 图片
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\FolderDescriptions\{0ddd015d-b06c-45d5-8c4c-f59713854639}\PropertyBag]
"ThisPCPolicy"="Hide"

; 隐藏另存为里面的 图片
[HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Explorer\FolderDescriptions\{0ddd015d-b06c-45d5-8c4c-f59713854639}\PropertyBag]
"ThisPCPolicy"="Hide"

; 视频
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\FolderDescriptions\{35286a68-3c57-41a1-bbb1-0eae73d76c95}\PropertyBag]
"ThisPCPolicy"="Hide"

[HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Explorer\FolderDescriptions\{35286a68-3c57-41a1-bbb1-0eae73d76c95}\PropertyBag]
"ThisPCPolicy"="Hide"

; 下载
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\FolderDescriptions\{7d83ee9b-2244-4e70-b1f5-5393042af1e4}\PropertyBag]
"ThisPCPolicy"="Hide"

[HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Explorer\FolderDescriptions\{7d83ee9b-2244-4e70-b1f5-5393042af1e4}\PropertyBag]
"ThisPCPolicy"="Hide"

; 音乐
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\FolderDescriptions\{a0c69a99-21c8-4671-8703-7934162fcf1d}\PropertyBag]
"ThisPCPolicy"="Hide"

[HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Explorer\FolderDescriptions\{a0c69a99-21c8-4671-8703-7934162fcf1d}\PropertyBag]
"ThisPCPolicy"="Hide"

; 桌面
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\FolderDescriptions\{B4BFCC3A-DB2C-424C-B029-7FE99A87C641}\PropertyBag]
"ThisPCPolicy"="Hide"

[HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Explorer\FolderDescriptions\{B4BFCC3A-DB2C-424C-B029-7FE99A87C641}\PropertyBag]
"ThisPCPolicy"="Hide"

; 文档
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\FolderDescriptions\{f42ee2d3-909f-4907-8871-4c22fc0bf756}\PropertyBag]
"ThisPCPolicy"="Hide"

[HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Explorer\FolderDescriptions\{f42ee2d3-909f-4907-8871-4c22fc0bf756}\PropertyBag]
"ThisPCPolicy"="Hide"
```

# activate 

```
slmgr /skms ip
slmgr /ato
```

# shell loop do cmd
```shell
while true ;do dmesg; sleep 2;done;
```

# 打开 Linux的dev_dbg

```
#define DEBUG
```
add `#define DEBUG` to top line of driver file 

```
CONFIG_DEBUG_FS=y
CONFIG_DYNAMIC_DEBUG=y
CONFIG_CONSOLE_LOGLEVEL_DEFAULT=10
```

```shell
cat /proc/sys/kernel/printk
```


# Windows11 classic Context Menu

写批处理, 还原就把add 改成delete

```bat
reg add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /f /ve
taskkill /f /im explorer.exe & start explorer.exe
```

---

# G9008W 破解联通4G

```
5.0
https://www.sammobile.com/samsung/galaxy-s5/firmware/SM-G9008W/CHM/download/G9008WZMU1BPD2/1071222/

6.0
https://www.sammobile.com/samsung/galaxy-s5/firmware/SM-G9008W/CHM/download/G9008WZMS1CRH1/240477/


    Power off your phone

    Reboot Phone in Download Mode
        Connect USB-cable to your computer. Hold down Volume up
        and Volume down at same time. While holding down, connect 
        the USB-cable to your phone

    Wait until you get a blue sign in Odin

    Add MD5-file in AP-field in Odin. Do not tick any extra boxes. 
        The only options to be ticked are F. Reset Time and Auto-Reboot.

    Click the start button to begin flashing the firmware update

    android 7.0
        adb shell settings put global captive_portal_detection_enabled 0


    1.在拨号键盘输入*#0808#，选择DM+MODEM+ADB模式。（破解完后记得改回去）
    2.在我的电脑-》右键-》属性-》设备管理-》端口下记住三星usb链接的com号
    3.打开高通QPST的连接配置工具QPST comfiguration，并手动添加端口,添加完成后，会在QPST Configuration中看到端口处于连接状态添加完成后，会在QPST Configuration中看到端口处于连接状态
    4.打开EFS Explorer,选择MSM8974对应的端口，点击ok
    5.EFS Explorer与手机连接成功后，在左侧目录找到policyman文件夹，在carrier_policy.xml文件那右键->dalete删除，空白处右键->copy date file from pc
    6.再次在手机原生拨号软件拨号盘输入*#0808#，改回“MTP”模式重启手机

```

# kms server

```
curl
apt-get update -y && apt-get install curl -y
bash <(curl -s -L https://git.io/v2ray.sh)
v2ray url
wget https://raw.githubusercontent.com/dakkidaze/one-key-kms/master/one-key-kms-debian.sh && chmod +x one-key-kms-debian.sh && ./one-key-kms-debian.sh
wget https://raw.githubusercontent.com/dakkidaze/one-key-kms/master/kms.sh && chmod +x kms.sh
./kms.sh start
./kms.sh stop
./kms.sh status
```

```
Mac_Android_Usb网络共享
http://joshuawise.com/horndis
```

```
adb shell settings put global captive_portal_mode 0

定时关机
at 22:45 shutdown -s
```

```
大计算 强大算法 巨量数据
```

```
省人才交流中心档案转出

组织关系转出所需材料
    人才档案管理卡
    党费收据
    身份证原件
    党员手册原件
    提供将要转入的基层党组织名称及联系电话, 介绍信抬头名称, 问清楚接收方是否需要组织材料
    支部党员转出, 需告知支部书记, 党费交至转出当月

非本人 电话0371-65957359
    本人身份证复印件及亲笔委托书, 写在复印件空白处
    被委托人身份证原件及复印件
```

```
modbuspoll      5A5742575C5D391A17627B6C010350
slave           5455415451475662
slave7.5.1      54554154514756666A655A187D545E505C454F
vm              ZF3R0-FHED2-M80TY-8QYGC-NPKYF
xp              MRX3F-47B9T-2487J-KWKMF-RPWBY

macOS istat
982092332@qq.com
GAWAE-FCWQ3-P8NYB-C7GF7-NEDRT-Q5DTB-MFZG6-6NEQC-CRMUD-8MZ2K-66SRB-SU8EW-EDLZ9-TGH3S-8SGA

ed2k://|file|%E6%8C%87%E7%8E%AF%E7%8E%8B.%E6%8A%A4%E6%88%92%E4%BD%BF%E8%80%85.%E5%8A%A0%E9%95%BF%E7%89%88.The.Lord.of.the.Rings.The.Fellowship.of.the.Ring.Extended.Editions.2001.BD.MiniSD-TLF.mkv|1840264910|6A7DC5CC1B8BC8704B63D00534F01402|h=4GASXT5GNDTE35HNLJE6B4ZLKAHMFTHP|/
ed2k://|file|%E6%8C%87%E7%8E%AF%E7%8E%8B.%E5%8F%8C%E5%A1%94%E5%A5%87%E5%85%B5.%E5%8A%A0%E9%95%BF%E7%89%88.The.Lord.of.the.Rings.The.Two.Towers.Extended.Editions.2002.BD.MiniSD-TLF.mkv|1986103938|52065CE62D95CC543055995D3D73CF5C|h=VIEKUKX4WEIC3HFDJPNLO2VJGGLUGTFI|/
ed2k://|file|%E6%8C%87%E7%8E%AF%E7%8E%8B.%E7%8E%8B%E8%80%85%E5%BD%92%E6%9D%A5.%E5%8A%A0%E9%95%BF%E7%89%88.The.Lord.of.the.Rings.The.Return.of.the.King.Extended.Editions.2003.BD.MiniSD-TLF.mkv|2318192280|3DD4382BCCB7A32DE43E608F8DCC261F|h=PB7YUOK4W5Z7ETU6UX6GQ3ARGDCWAUOB|/

ed2k://|file|sc_office_2003_pro.iso|616847360|AB7DEC602B533F9DF8A04AAB1B27C213|/
ed2k://|file|zh-hans_windows_xp_professional_with_service_pack_3_x86_cd_vl_x14-74070.iso|630237184|EC51916C9D9B8B931195EE0D6EE9B40E|/
ed2k://|file|%5B.%E6%AC%A7%E7%89%B9%E5%85%8B.ACAD.2014.%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%E7%89%8864%E4%BD%8D%5D.AutoCAD_2014_Simplified_Chinese_Win_64bit_dlm.sfx.exe|1583623158|28bab81fb144cad13328dd444f4dd1a7|/
ed2k://|file|Autodesk.2014%E6%B3%A8%E5%86%8C%E6%9C%BA.rar|601914|e1f3eb3d8094ecbf23728840fb318cb7|/
ed2k://|file|%5B.%E6%AC%A7%E7%89%B9%E5%85%8B.ACAD.2014.%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%E7%89%8864%E4%BD%8D%5D.autocad_2014_offline_help_simplified_chinese_win_32_64bit_dlm.sfx.exe|203127652|91db04df70749560da30976176834ce9|/
ed2k://|file|%5B.%E6%AC%A7%E7%89%B9%E5%85%8B.ACAD.2014.%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%E7%89%8864%E4%BD%8D%5D.AutoCAD_2014_VBA_Enabler_English_Win_64bit_dlm.sfx.exe|111419053|e89ca50d9cf222d062d0e9949740afb7|/
http://trial2.autodesk.com/SWDLDNET3/2014/3DSMAX/DLM/Autodesk_3ds_Max_2014_EFGJKS_Win_64bit_dlm.sfx.exe
http://trial2.autodesk.com/SWDLDNET3/2014/MAYA/DLM/Autodesk_Maya_2014_dlm.sfx.exe
ed2k://|file|cn_windows_7_ultimate_with_sp1_x64_dvd_u_677408.iso|3420557312|B58548681854236C7939003B583A8078|/|
http://down.sandai.net/Thunder5.9.28.1564.exe


ed2k://|file|魔兽争霸III.混乱之治.iso|643596288|f3bdbfda64dc6ffe3c4e35c5b24441cb|h=cbdvzdr6vbjyebz75uivkzyyhrds7h2n|/
Reign of Chaos 6HZAPZ-CIA2-0DRFXF-KRS1-JTBLMY
ed2k://|file|魔兽争霸III.冰封王座.iso|557991936|5b5f62046072dc2542addc33f62b71a0|h=k5i7y3bc4pqkeq32xc3a3cb6xmsk3umj|/
Frozen Throne W79YYX-E7N4-Y9N24D-WEZR-FVNGTJ
ed2k://|file|[红色警戒2].RA2.rar|1486635488|7f04b65c75365165c811fc50476f02cb|h=7epm72w3k3anb5lwnewfmidor7632qca|/
ed2k://|file|祖玛复仇记Zuma's.Revenge英文版.rar|98634496|af81d6c63fc3018d81047ce69c49b329|/
ed2k://|file|祖玛传奇Zuma.Deluxe中文版.rar|11228680|eb257144b14ff490a1404d6b9fdebbfd|h=522n2vk57ifoju5a4heytdl2ej4t5ul2|/
ed2k://|file|植物大战僵尸年度版Plants.Vs.Zombies.Game.Of.The.Year.Edition无广告官方中文版.rar|90301766|b283ad67a90bda2ef2e0fc1f6deddfb7|h=e6qdw5o72j2atzydmw6qudipfc7drbtd|/
ed2k://|file|毛毛Chuzzle.Deluxe中文版.rar|12410895|df6b872944736a8e0efb98efc69412ba|h=btvhvx4466unm7snlutlq6g22qxvhx5j|/
ed2k://|file|粘粘世界World.of.Goo中文版.rar|73082687|ef799e5c768c0882b5777591d01f41db|h=c2whraxdy5i4e7o7k2q5mx4d7fdkguu7|/
ed2k://|file|[平衡-完美硬盘版].Ballance.EXE|16162781|b98efdb42b979a32cefcde38b252c623|/
ed2k://|file|[平衡Cheat].Database.zip|599|5bb33464dadcea6d4409a3896dd102eb|/
ed2k://|file|[极品飞车9：最高通缉].Need.For.Speed.Most.Wanted-RELOADED-ENG.iso|2303506432|7c031d819e9f9dc2130559f2ae1dc6b6|h=77uuwactltpo5qmwwp4vasea6cwwgiem|/
ed2k://|file|%E5%90%88%E9%87%91%E5%BC%B9%E5%A4%B4+%E5%90%88%E9%9B%86%281%2C2%2C3%2C4%2C5%2CX%29.rar|193142139|abe013c148d8e6de8d4943ae14dd93b7|h=A5LFQXTETIXLS7J3H6P5NGJAFBQWQ54T|/
ed2k://|file|[2006年德国世界杯进球集锦].FIFA.World.Cup.All.The.Goals.Of.Germany.2006.blu-ray.x264.720P.AC3-CHD.mkv|4687565656|a5f644a2a057b7420789f9c07e7c2f6c|h=hhjmlkyahlmiqkpcqtzuinsfcfcjuogo|/
ed2k://|file|大话西游之仙履奇缘.A.Chinese.Odyssey.Part.II.Cinderella.1994.BDRip.x264.AAC.miniSD-TLF.mkv|649520873|0F44CF4513B0898D6D0E04F1B4192BFC|h=CHUK6EY2SZWDGPGEA5LDXA5VO3ULAJWK|/|
ed2k://|file|大话西游之月光宝盒.A.Chinese.Odyssey.Part.I.Pandoras.Box.1994.BDRip.x264.2Audio.AAC.miniSD-TLF.mkv|574496832|5EF625B94E77211BE2FE4B9CF1E87477|h=354O5YDYVLAA4VMGFWQ33QRCTSUXCRBQ|/|
ed2k://|file|%E7%96%AF%E7%8B%82%E7%9A%84%E7%9F%B3%E5%A4%B4.Crazy.Stone.2006.HDTV.2Audio.MiniSD-TLF.mkv|622781376|5F013B2E551BCB37F220934DFCA4E19D|h=KIBMZGVS4DHQMODTHXSIH2HKQJRAGUXD|/
ed2k://|file|%E4%BA%BA%E5%9C%A8%E5%9B%A7%E9%80%94.Lost.On.Journey.2010.HDTV.MiniSD-TLF.mkv|702178924|8ECCDB88358D90E934A4886D6DAB886B|/
ed2k://|file|%E6%80%A6%E7%84%B6%E5%BF%83%E5%8A%A8.Flipped.2010.BD.MiniSD-TLF.mkv|717539080|98F1CE8AE865CB92FF11F2DA53C7A7C9|/
ed2k://|file|%E5%BF%83%E8%8A%B1%E8%B7%AF%E6%94%BE.Breakup.Buddies.2014.WEB-DL.MiniSD-TLF.mkv|976275934|16BE3005529A91834784D1546FAA9A1E|h=FFIZ6VU3LF7VMKDHUULJ7TAJENKSOVVR|/
ed2k://|file|%E4%BA%BA%E5%86%8D%E5%9B%A7%E9%80%94%E4%B9%8B%E6%B3%B0%E5%9B%A7.Lost.In.Thailand.2012.HDTV.SUBBED.MiniSD-TLF.mkv|856591062|F1916F8303245BF130A0280EE7587E54|/
ed2k://|file|%E4%BA%BA%E5%86%8D%E5%9B%A7%E9%80%94%E4%B9%8B%E6%B3%B0%E5%9B%A7.Lost.in.Thailand.2012.BD.2Audio.MiniSD-TLF.mkv|895666789|23A23384C645BB41DFF80E807E1C00E9|/
ed2k://|file|侯宝林.-.[侯宝林大师相声精选].专辑.(mp3).rar|423587975|4b11fd521386ff50833967ae57327204|h=6fivr6r3aaqkc7qpjoqgbbim2i74hkkp|/
ed2k://|file|[新概念英语英音版课文mp3].New.Concept.English.mp3.book.1.rar|9330999|63d1a72cfae1b891d9bf0d5757964eef|h=wik645uuqrs2u526j2rtam3rqffcrnzk|/
ed2k://|file|[新概念英语英音版课文mp3].New.Concept.English.mp3.book.2.rar|16226549|d676c0e3b3cc22751c24d97113dfb0f9|h=3v56cjanemcqm5xw3mobd7ihmlnaqbug|p=B98AB1C7CEF265AB96BC0ACB30DC1948:3ADEF0D84838BAB7353C50885F10C2C7|/
ed2k://|file|[新概念英语英音版课文mp3].New.Concept.English.mp3.book.4.rar|16954538|7ed7503e41c4841ff19557d0861e0db1|h=njhoxzmq54bqa5oxz6ki7dh5y53ijpjq|p=4566D2B8847C8FB647B2083F5519F1FA:5581637A4C2E180E1AA2C6006736BF86|/
ed2k://|file|[新概念英语英音版课文mp3].New.Concept.English.mp3.book.3.rar|21638856|32f368399ac218eb55e2b1200a52da96|h=njl2j5rbhstzxwl5k7pn7nr3bq3fgggl|p=697493F9FB640297EEAA340F34A43EC4:D1700F6969F102DF8D0CC0B7DD7F97C9:55C2198E159F631B1BB6BA11D720BE3F|/
ed2k://|file|豫剧-名家名段2.rar|70167828|7fb23c795ed144b7debe74cdcea7e411|h=c46evpiyuhsoevejznz7ioelg7b3oluh|/
ed2k://|file|豫剧-名家名段3.rar|81845936|240b8e6481bc0012ddb2f94dd4ebb6e7|h=k52d7evnrxy6wfs6663f4uo2jm5wvtcy|/
ed2k://|file|豫剧-名家名段1.rar|84241024|a0cf53f8ce5f0073be53e30088a5bb82|h=fzpd4qttjsejy23evwe7azbxq5gwohth|/
ed2k://|file|[宝石迷阵3].BJ3Setup.exe|108013877|3c11f81034d615a333899f78eff60969|h=2lq52pcmw5xjxapv3wemdcvnom46x3xc|/
ed2k://|file|《宝石迷阵3》特色版.exe|109325104|eebd4ddd570ab09b058444f74241734a|h=nfg25bzl7s5yhtpxf5dduqnufjupdewe|/
ed2k://|file|新版新概念英语第一册(英音)MP3.rar|136636891|cca3ce23c5e6f691d5e1f2a6414b72a6|h=gcsfmhgeu5c4a2irfeuzvhw3lys67tph|/
ed2k://|file|新版新概念英语第一册(美音)MP3.rar|145823463|52af1cce4f9217f856b2920bc82d1ba0|h=ww3oljoxvms2vpxxef7dndta56b55hfc|/
ed2k://|file|新概念第一册双数课.rar|159609873|1aa5d4d5937ed885ad9782f91082c755|/
ed2k://|file|新版新概念英语第四册(英音)MP3.rar|177848367|37c3e88b97a10fdaca97738e781e4ed2|h=k7bsgqxs5dnau4putemrwyigk6guaajn|/
ed2k://|file|新版新概念英语第三册(英音)MP3.rar|182452736|eaba7da3cde85fd367b31ce95f4541ec|h=j4jjzq4i7ajyn5hjhlsyrdczbwi27cor|/
ed2k://|file|新版新概念英语第四册(美音)MP3.rar|183285662|99ab91ca3b5dad3f422a395de77761af|h=6vlcevxh2cf7tcl7uizugagohpnjqbii|/
ed2k://|file|新版新概念英语第二册(美音)MP3.rar|205331061|abf39a2726d30e6a814829a206abb7a7|h=gi2xf5g7mvx24a2x5mjfv4zy27pc5dl6|/
ed2k://|file|新版新概念英语第二册(英音)MP3.rar|207878433|49598dd3132c3abd279af50d37bd4a3d|h=y6qx46clz55poq2zpwq6ehmh27xfub5w|/
ed2k://|file|新版新概念英语第三册(美音)MP3.rar|215128535|a6cbc7503566afebee7f13be0a775787|h=jxtpglhf2rjgjleaqgomrffbua7kps76|/
ed2k://|file|群星.-.[经典少儿歌曲180首].专辑.(MP3).rar|403011862|f149d3e09a47f3fea01059b82ce5a286|h=dknvvguwxx3ifkqefznit3f6hjjkaf72|/
ed2k://|file|[抗日血战上海滩].shanghai.iso|469548576|12f0f72700a20a9e8fb64864e1e0c323|h=xhknte3kwb3gtu6aoycbruohe74ttwzc|/
ed2k://|file|新概念英语（美音版）CD转mp3.rar|593543573|34549f9b0cb7eb5c41e02516bdab30b3|h=romunkge6jyk7ez3qeg7tpm7kaa3uhnh|/
ed2k://|file|评书：三国演义(单田芳)[112回-32kbps].rar|600569476|b6ea6ebbb75c73e23f3e6c631734a5a7|h=k2zup6vxbpd2drhmqxnl2zorsw6f3aki|/

magnet:?xt=urn:btih:102F25B9706989D587CBA9C919C6795575E67A85&dn=Warcraft%20III%20%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%E5%90%88%E9%9B%86
magnet:?xt=urn:btih:3A9DCD9E6871BEF046D78194A91BD67C84BBD110&dn=%E5%8D%83%E9%A6%96%E5%8D%95%E6%9B%B2
magnet:?xt=urn:btih:5E6391A8E48F990C6EA3E467B332B538B8738368&dn=%E7%8F%AD%E5%BE%97%E7%91%9E%E5%85%A8%E9%9B%8613%E5%BC%A0
magnet:?xt=urn:btih:B11BD3680FFA0DF1BD5A84D2EAC46210CBDF384F&dn=%E5%A5%A5%E6%96%AF%E5%8D%A1%E5%8E%86%E5%B1%8A%E6%9C%80%E4%BD%B3%E5%BD%B1%E7%89%87
magnet:?xt=urn:btih:E27D98B59FECB51B4987B33216E50252EFCCF274&dn=%E5%8E%86%E5%B1%8A%E5%A5%A5%E6%96%AF%E5%8D%A1
magnet:?xt=urn:btih:82DA265DCBE469457FFCB27ECAD89AA3F81D7653&dn=IMDB%20Top%20250%E5%90%88%E8%BE%91
magnet:?xt=urn:btih:59C9B5B9F863F7DA2308B723430AE38DD970B530&dn=IMDB%20Top%20250%E5%90%88%E8%BE%91%20TLF-MiniSD%E6%94%B6%E8%97%8F%E7%89%88
magnet:?xt=urn:btih:84D2A8D545C334D69BBF3B4D9CEA6ABACEF7342B&dn=%E3%80%90DOVS%E5%8E%9F%E5%88%9B%E3%80%91%E5%AE%9E%E5%86%B5%E8%B6%B3%E7%90%838%E5%9B%BD%E9%99%85%E7%89%88DOVS%E5%AE%8C%E7%BE%8E%E7%BA%AA%E5%BF%B5%E7%8F%8D%E8%97%8F%E7%89%88%5B%E7%A1%AC%E7%9B%98%E5%AE%89%E8%A3%85%E7%89%88%5D
magnet:?xt=urn:btih:C1F40E02B3EA06B44ACA74CD986E55AE4BE0180C&dn=Band%20Of%20Brothers%202001%20X264%201080p%20DTS
magnet:?xt=urn:btih:F8DDBE46D8B46085061680E7F2FBA9448CF3D990&dn=The.Pacific.2010.EP01-EP10.BluRay.1080p.DTS.x264-CHD
magnet:?xt=urn:btih:4223DD98CF31B47A79BBB3AD5102AE0F2400C628&dn=%E7%8C%AB%E5%92%8C%E8%80%81%E9%BC%A050%E5%91%A8%E5%B9%B410DVD%E7%8F%8D%E8%97%8F%E7%89%88
magnet:?xt=urn:btih:41AD245C06F051A6CC566E1F74B1AE94E088B668&dn=%E4%B8%AD%E5%8D%8E%E5%B0%8F%E5%BD%93%E5%AE%B6
magnet:?xt=urn:btih:92CF5CA56AE5A0558CC413200A90A37FA99D75E9&dn=%5BCMBT%5D%20Cooking%20Master%20Boy%20Complete
magnet:?xt=urn:btih:D89371566355C52EE1BADDAB751666188DB94D8E&dn=%E4%B8%AD%E5%8D%8E%E5%B0%8F%E5%BD%93%E5%AE%B6%EF%BC%88%E4%B8%AD%E5%8D%8E%E4%B8%80%E7%95%AA%EF%BC%89
magnet:?xt=urn:btih:B3C19B174807D29BCE5331682DE897ABC9713EFB&dn=%5B%E5%8F%B0%E7%81%A3%E6%9D%B1%E7%AB%8B%5D%5B%E5%B2%B8%E6%9C%AC%E9%BD%8A%E5%8F%B2%5D%5B%E7%81%AB%E5%BD%B1%E5%BF%8D%E8%80%85%5D%5BNARUTO%20-%E3%83%8A%E3%83%AB%E3%83%88-%20%5D%5B%E5%85%A8%E4%B8%83%E5%8D%81%E4%BA%8C%E9%9B%86%5D.rar
magnet:?xt=urn:btih:69274F4C7A4A3A1BAD6CBE4D94BBCD3DDDAF5416&dn=%E6%9C%88%E5%85%89%E6%81%8B%E6%9B%B2SOSG%E7%81%AB%E5%BD%B1%E5%BF%8D%E8%80%85
magnet:?xt=urn:btih:5BA10590CC244EE549586AE90C21994801B415B2&dn=%5BPKM%E5%AD%97%E5%B9%95%E7%BB%84%5D%E2%98%85%E7%88%B1%E8%BF%AA%E5%A5%A5%E7%89%B9%E6%9B%BC%E2%98%85%5B01-50fin%5D%5BDVDrip%5D%5Bmkv%5D
magnet:?xt=urn:btih:320A977390316B76F78653932FCBCD8B85214439&dn=1971.%E5%BD%92%E6%9D%A5%E7%9A%84%E5%A5%A5%E7%89%B9%E6%9B%BC
magnet:?xt=urn:btih:0E32A51A012F1AED7E5C606A10A84495DFD49F94&dn=%E3%82%A6%E3%83%AB%E3%83%88%E3%83%A9%E3%82%B7%E3%83%AA%E3%83%BC%E3%82%BA(%E6%98%AD%E5%92%8C)

magnet:?xt=urn:btih:E27D98B59FECB51B4987B33216E50252EFCCF274
magnet:?xt=urn:btih:82DA265DCBE469457FFCB27ECAD89AA3F81D7653
magnet:?xt=urn:btih:B11BD3680FFA0DF1BD5A84D2EAC46210CBDF384F
magnet:?xt=urn:btih:59C9B5B9F863F7DA2308B723430AE38DD970B530
magnet:?xt=urn:btih:C1F40E02B3EA06B44ACA74CD986E55AE4BE0180C
magnet:?xt=urn:btih:F8DDBE46D8B46085061680E7F2FBA9448CF3D990
magnet:?xt=urn:btih:4223DD98CF31B47A79BBB3AD5102AE0F2400C628
magnet:?xt=urn:btih:0E32A51A012F1AED7E5C606A10A84495DFD49F94
magnet:?xt=urn:btih:92CF5CA56AE5A0558CC413200A90A37FA99D75E9
magnet:?xt=urn:btih:D89371566355C52EE1BADDAB751666188DB94D8E
magnet:?xt=urn:btih:B3C19B174807D29BCE5331682DE897ABC9713EFB
magnet:?xt=urn:btih:69274F4C7A4A3A1BAD6CBE4D94BBCD3DDDAF5416
magnet:?xt=urn:btih:5BA10590CC244EE549586AE90C21994801B415B2
magnet:?xt=urn:btih:320A977390316B76F78653932FCBCD8B85214439

magnet:?xt=urn:btih:5E6391A8E48F990C6EA3E467B332B538B8738368&dn=%E7%8F%AD%E5%BE%97%E7%91%9E%E5%85%A8%E9%9B%8613%E5%BC%A0
magnet:?xt=urn:btih:B11BD3680FFA0DF1BD5A84D2EAC46210CBDF384F&dn=%E5%A5%A5%E6%96%AF%E5%8D%A1%E5%8E%86%E5%B1%8A%E6%9C%80%E4%BD%B3%E5%BD%B1%E7%89%87
magnet:?xt=urn:btih:E27D98B59FECB51B4987B33216E50252EFCCF274&dn=%E5%8E%86%E5%B1%8A%E5%A5%A5%E6%96%AF%E5%8D%A1
magnet:?xt=urn:btih:82DA265DCBE469457FFCB27ECAD89AA3F81D7653&dn=IMDB%20Top%20250%E5%90%88%E8%BE%91
magnet:?xt=urn:btih:59C9B5B9F863F7DA2308B723430AE38DD970B530&dn=IMDB%20Top%20250%E5%90%88%E8%BE%91%20TLF-MiniSD%E6%94%B6%E8%97%8F%E7%89%88
magnet:?xt=urn:btih:84D2A8D545C334D69BBF3B4D9CEA6ABACEF7342B&dn=%E3%80%90DOVS%E5%8E%9F%E5%88%9B%E3%80%91%E5%AE%9E%E5%86%B5%E8%B6%B3%E7%90%838%E5%9B%BD%E9%99%85%E7%89%88DOVS%E5%AE%8C%E7%BE%8E%E7%BA%AA%E5%BF%B5%E7%8F%8D%E8%97%8F%E7%89%88%5B%E7%A1%AC%E7%9B%98%E5%AE%89%E8%A3%85%E7%89%88%5D
magnet:?xt=urn:btih:4223DD98CF31B47A79BBB3AD5102AE0F2400C628&dn=%E7%8C%AB%E5%92%8C%E8%80%81%E9%BC%A050%E5%91%A8%E5%B9%B410DVD%E7%8F%8D%E8%97%8F%E7%89%88
magnet:?xt=urn:btih:41AD245C06F051A6CC566E1F74B1AE94E088B668&dn=%E4%B8%AD%E5%8D%8E%E5%B0%8F%E5%BD%93%E5%AE%B6
magnet:?xt=urn:btih:92CF5CA56AE5A0558CC413200A90A37FA99D75E9&dn=%5BCMBT%5D%20Cooking%20Master%20Boy%20Complete
magnet:?xt=urn:btih:D89371566355C52EE1BADDAB751666188DB94D8E&dn=%E4%B8%AD%E5%8D%8E%E5%B0%8F%E5%BD%93%E5%AE%B6%EF%BC%88%E4%B8%AD%E5%8D%8E%E4%B8%80%E7%95%AA%EF%BC%89
magnet:?xt=urn:btih:B3C19B174807D29BCE5331682DE897ABC9713EFB&dn=%5B%E5%8F%B0%E7%81%A3%E6%9D%B1%E7%AB%8B%5D%5B%E5%B2%B8%E6%9C%AC%E9%BD%8A%E5%8F%B2%5D%5B%E7%81%AB%E5%BD%B1%E5%BF%8D%E8%80%85%5D%5BNARUTO%20-%E3%83%8A%E3%83%AB%E3%83%88-%20%5D%5B%E5%85%A8%E4%B8%83%E5%8D%81%E4%BA%8C%E9%9B%86%5D.rar
magnet:?xt=urn:btih:5BA10590CC244EE549586AE90C21994801B415B2&dn=%5BPKM%E5%AD%97%E5%B9%95%E7%BB%84%5D%E2%98%85%E7%88%B1%E8%BF%AA%E5%A5%A5%E7%89%B9%E6%9B%BC%E2%98%85%5B01-50fin%5D%5BDVDrip%5D%5Bmkv%5D
magnet:?xt=urn:btih:320A977390316B76F78653932FCBCD8B85214439&dn=1971.%E5%BD%92%E6%9D%A5%E7%9A%84%E5%A5%A5%E7%89%B9%E6%9B%BC
magnet:?xt=urn:btih:0E32A51A012F1AED7E5C606A10A84495DFD49F94&dn=%E3%82%A6%E3%83%AB%E3%83%88%E3%83%A9%E3%82%B7%E3%83%AA%E3%83%BC%E3%82%BA(%E6%98%AD%E5%92%8C)

ed2k://|file|%E7%BA%A6%E4%BC%9A%EF%BD%9E%E6%81%8B%E7%88%B1%E7%A9%B6%E7%AB%9F%E6%98%AF%E4%BB%80%E4%B9%88%E5%91%A2%EF%BD%9E.Date.Koitowa.Donnamonokashira.Ep01.Chi_Jap.HDTVrip.1024X576.mkv|627379256|fea74204994c771d09180aa03a70798d|h=pr4gg4z6l2646ees3cxfab7vxyyapnk3|/
ed2k://|file|%E7%BA%A6%E4%BC%9A%EF%BD%9E%E6%81%8B%E7%88%B1%E7%A9%B6%E7%AB%9F%E6%98%AF%E4%BB%80%E4%B9%88%E5%91%A2%EF%BD%9E.Date.Koitowa.Donnamonokashira.Ep02.Chi_Jap.HDTVrip.1024X576.mkv|522854790|1b8b998553ba9109ea0e17f5123c0159|h=lygam4pccxxnrdqpgdeug4xfvx6xrilc|/
ed2k://|file|%E7%BA%A6%E4%BC%9A%EF%BD%9E%E6%81%8B%E7%88%B1%E7%A9%B6%E7%AB%9F%E6%98%AF%E4%BB%80%E4%B9%88%E5%91%A2%EF%BD%9E.Date.Koitowa.Donnamonokashira.Ep03.Chi_Jap.HDTVrip.1024X576.mkv|522638648|2de40496ef76a36a426bd6107dbb4dc0|h=vwhrhh5ht27l35lgluufnpojn25fn2e4|/
ed2k://|file|%E7%BA%A6%E4%BC%9A%EF%BD%9E%E6%81%8B%E7%88%B1%E7%A9%B6%E7%AB%9F%E6%98%AF%E4%BB%80%E4%B9%88%E5%91%A2%EF%BD%9E.Date.Koitowa.Donnamonokashira.Ep04.Chi_Jap.HDTVrip.1024X576.mkv|522681786|0ad63c0e824ed83d5c821b2556662a97|h=y277gxdhdtcaeqhjy74jgq5saavgvons|/
ed2k://|file|%E7%BA%A6%E4%BC%9A%EF%BD%9E%E6%81%8B%E7%88%B1%E7%A9%B6%E7%AB%9F%E6%98%AF%E4%BB%80%E4%B9%88%E5%91%A2%EF%BD%9E.Date.Koitowa.Donnamonokashira.Ep05.Chi_Jap.HDTVrip.1024X576.mkv|522891130|e0121b6cfca984ca294e34d81dbc7bea|h=x6emptt353esyzeqaa7dgok2kh6oxqdj|/
ed2k://|file|%E7%BA%A6%E4%BC%9A%EF%BD%9E%E6%81%8B%E7%88%B1%E7%A9%B6%E7%AB%9F%E6%98%AF%E4%BB%80%E4%B9%88%E5%91%A2%EF%BD%9E.Date.Koitowa.Donnamonokashira.Ep06.Chi_Jap.HDTVrip.1024X576.mkv|522922717|6cd95143bf32e1c00c39263c8335f845|h=e6yf5irebmwn2zsuy75zndne4zmy2x57|/
ed2k://|file|%E7%BA%A6%E4%BC%9A%EF%BD%9E%E6%81%8B%E7%88%B1%E7%A9%B6%E7%AB%9F%E6%98%AF%E4%BB%80%E4%B9%88%E5%91%A2%EF%BD%9E.Date.Koitowa.Donnamonokashira.Ep07.Chi_Jap.HDTVrip.1024X576.mkv|522741768|e20fe35353f5f5d9142dd6b9725cf06d|h=q7gvowptl3qtl477jzofewsfig4ozhas|/
ed2k://|file|%E7%BA%A6%E4%BC%9A%EF%BD%9E%E6%81%8B%E7%88%B1%E7%A9%B6%E7%AB%9F%E6%98%AF%E4%BB%80%E4%B9%88%E5%91%A2%EF%BD%9E.Date.Koitowa.Donnamonokashira.Ep08.Chi_Jap.HDTVrip.1024X576.mkv|522786461|6af67b7e8c8246cb3e5a23b2c432b6f4|h=wzm6quceaahpgt4ovqbnebwkuwn4ci45|/
ed2k://|file|%E7%BA%A6%E4%BC%9A%EF%BD%9E%E6%81%8B%E7%88%B1%E7%A9%B6%E7%AB%9F%E6%98%AF%E4%BB%80%E4%B9%88%E5%91%A2%EF%BD%9E.Date.Koitowa.Donnamonokashira.Ep09.Chi_Jap.HDTVrip.1024X576.mkv|627198588|5c4f7710c273ef98d81f116d03dd5ea3|h=wevqbibwtheorv3faivzs4m6aartgf44|/
ed2k://|file|%E7%BA%A6%E4%BC%9A%EF%BD%9E%E6%81%8B%E7%88%B1%E7%A9%B6%E7%AB%9F%E6%98%AF%E4%BB%80%E4%B9%88%E5%91%A2%EF%BD%9E.Date.Koitowa.Donnamonokashira.Ep10.Final.Chi_Jap.HDTVrip.1024X576.mkv|627423905|7a2f7da5cd9224a514b1a448f3d58d75|h=rdkunagbx6t27jnyo7fuycg664khbxh6|/
ed2k://|file|%E7%BA%A6%E4%BC%9A%EF%BD%9E%E6%81%8B%E7%88%B1%E7%A9%B6%E7%AB%9F%E6%98%AF%E4%BB%80%E4%B9%88%E5%91%A2%EF%BD%9ESP.Date.Koitowa.Donnamonokashira.SP.Chi_Jap.HDTVrip.1024X576-ZhuixinFan.mkv|1392254768|b2c3be049b58657431f5ebf66f08ceb3|h=fnaywcsqycwmmtfkzfrnnro4sxkpcwgo|/
```

```
http://mp.weixin.qq.com/s?__biz=MjM5ODM2MzY4MQ==&mid=2651771253&idx=1&sn=e50c21dbf3526be0554555946633c655&chksm=bd31a63b8a462f2d2ad44827651f24ce627e028f7e34e12582b5bf0259824fd2193862b6cb4a&mpshare=1&scene=24&srcid=1117sxB6XOdhLr72lVIqiqQM#rd
```
