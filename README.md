# 概述
franka分为desk界面和FCI

本教程主要把几个官方和非官方的教程资料整合了一下，重点在顺利下载安装对应库、实现对应例程的流程。


## 资料

基本功能和desk界面学习参照[pdf文档](https://github.com/yejianheng57/Franka_tutorial/blob/main/110010_Product%20Manual%20Franka%20Production%203_1.6_ZH.pdf)和[网课《Franka机器人入门培训》](https://meeting.tencent.com/crm/2Omw5Rk91e)

FCI学习参照[指南](https://www.franka.cn/FCI/index.html#)

- 指南中的《综述》《最低系统和网络要求》《版本兼容》《入门指南》可以参考并且自查。
- 操作系统建议使用linux双系统，虚拟机不满足实时性要求，windows不能用ros/ros2
- 然后[设置实时内核](https://www.franka.cn/FCI/installation_linux.html#setting-up-the-real-time-kernel)
- libfranka是FCI 客户端的官方 C++ 实现，安装和操作推荐直接看[官方github](https://www.franka.cn/FCI/getting_started.html)
- franka_ros2提供libfranka 的 ROS 2 集成，安装参考[官方github](https://github.com/frankarobotics/franka_ros2)
,使用参考[官方github](https://github.com/frankarobotics/franka_ros2)和[指南](https://www.franka.cn/FCI/franka_ros2.html#)都可以
- franky是非官方的支持 Python 和 C++的库，参考[github](https://github.com/TimSchneider42/franky)，[frankx](https://github.com/pantor/frankx)也是一个著名的非官方库


# 操作机械臂
## 开机

打开黑色控制器后面的开关，等待一段时间。

进入desk界面
- 网线直接连接机械臂的话，网址输入robot.franka.de（不能开梯子），输入帐号密码，帐号`franka`，密码`franka123` 
- 网线连接黑色控制器的话，网址输入172.16.0.2（在[已经设置好网络](https://www.franka.cn/FCI/getting_started.html#setting-up-the-network)之后）

desk上解锁关节（要解锁关节后，hand才会连接）
## 操作杂谈

开FCI运行程序的时候，比如运行python的时候，要用execution模式

当机械臂进入singular奇异位姿亮红灯的时候，改成programming模式，按住腕部两个按钮移动回正常位姿

y轴,z轴好像反了？

https://frankaemika.github.io/ 已经被废弃，相关库被转移到Franka Robotics，因此[指南](https://www.franka.cn/FCI/index.html#)里的部分跳转界面会跳转到404



# libfranka

安装使用从源代码构建（方法二）

下载libfranka照着[官方github](https://www.franka.cn/FCI/getting_started.html)做就行

实时内核建议用打补丁的方式，不要用ubuntu pro方式，否则就可能像我一样。

我没有使用debian

libfranka版本下的是0.15.0

跑通了示例代码


# franka_ros2
安装和操作参考[指南](https://www.franka.cn/FCI/franka_ros2.html)即可（或者[github](https://github.com/frankarobotics/franka_ros2)）

记得开启FCI

github中` Run a ROS 2 example controller` 章节可实现的功能个人感觉相比libfranka要更加多样和灵活（指南中的“[示例控制器](https://www.franka.cn/FCI/franka_ros2.html#example-controllers)”感觉实现的是同一个功能，但是不知道为什么运行会显示找不到文件）

github中 `Run a ROS 2 example controller` 章节，先打开 `/franka_ros2_ws/src/franka_bringup/config` ，在 `franka.config.yaml` 中把 `robot_ip` 改成 `172.16.0.2` ， `load_gripper` 改成 `true` ，然后参考 `controllers.yaml` 中的控制器名称，输入到
``` 
ros2 launch franka_bringup example.launch.py controller_name:=your_desired_controller
```
中的 `your_desired_controller` 进行示例演示。


可以用[moveit](https://www.franka.cn/FCI/franka_ros2.html#moveit)实现到指定位姿的路径规划plan和真实实现exeute
也可以用gazebo和rviz实现模拟可视化

指南中[` franka_gripper` ](https://www.franka.cn/FCI/franka_ros2.html#franka-gripper)章节无法操作，找不到 `fr3_gripper` 文件夹在哪里


# franky

## 操作步骤
- 确保实时内核安装完毕
- 创建conda环境，操作README里的 `Installing franky` 
- desk页面激活FCI，直接从 `Tutorial` 开始vscode复制例程运行

## 我的操作细节
我创建了conda环境“franky”，python=3.10

虽然我装了libfranka0.15.0，但是没有sudo make install,因此没有添加到系统路径，只在自己的库里，所以franky的时候仍然按照教程编译了轮子文件0.15.0



# 实时内核

只要使用FCI,就需要实时内核。在装实时内核的时候推荐使用打补丁的方式。
## 显卡驱动问题

以下是我的个人个别问题

我用ubuntu pro,去ubuntu官网订阅，wifi图标没了，网卡驱动没了，问AI稀里糊涂的搞好了（详见realtime_iwlwifi.markdown）

然后显卡驱动也没了，在装franky的“Can I use CUDA jointly with franky?”这一章节的脚本的时候，直接重启的时候黑屏  
```
[    0.229044] ACPI Error: AE_NOT_FOUND, While resolving a named reference package element - _SB_.PC00.LPCB.SEN2 (20210730/dspkginit-438)
/dev/nvme1n1p6: clean, 350721/2191168 files, 5828982/8749824 blocks
[    3.357152] iwlwifi 0000:00:14.3: WRT: Invalid time point 28 for host command TLV
[    3.916597]
```
原因：实时内核可能不完整，启动的时候混乱
解决方法：先进grub启动回标准内核，然后把实时内核删除，换打补丁的方式：

进grub：
开机的时候长按esc
grub里输入
```
ls 
ls (hd0,msdos1)/  #逐个进入，找一下boot在哪
ls (hd1,msdos16/boot/  #看vmlinuz和initrd的版本
set root=(hd1,msdos6)
linux /boot/vmlinuz-6.8.0-65-generic root=/dev/nvme1n1p6 rw #注意替换成自己的版本呢
initrd /boot/initrd.img-6.8.0-65-generic nomodeset  #注意替换成自己的版本呢
boot #启动后就可以回到原来的内核。但是此时再重启仍然会黑屏混乱，需要把realtime内核删了
```

启动后删realtime：
```
dpkg -l | grep linux-image
ls /boot
sudo apt-get purge linux-image-5.15.0-1089-realtime linux-headers-5.15.0-1089-realtime
sudo apt-get autoremove
sudo update-grub
sudo reboot
```

然后就只剩下干净的内核了。

我是选择把显卡驱动给删除了，等到后面franky正常后再按照脚本装。





