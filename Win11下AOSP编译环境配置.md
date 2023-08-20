## Win11+wsl2+Ubuntu20.04配置AOSP编译环境

### 系统环境

1. Windows11  SSD500G  CPU6核12线程  内存32G
2. Ubuntu20.04

 ### 安装Ubuntu

* 安装流程

1. 查看可以安装的linux版本

   ```powershell
   wsl --list --online
   ```

   

2. 安装Ubuntu

   ````powershell
   wsl --install Ubuntu-20.04
   ````

3. 从C盘导出Ubuntu备份到其他盘

   ````powershell
   wsl --export Ubuntu D://ubuntu.tar
   ````

   

4. 注销C盘Ubuntu子系统

   ````powershell
   wsl --unregister Ubuntu-20.04
   ````

   

5. 导入Ubuntu

   1. 从备份中导入

      ```powershell
      wsl --import Ubuntu-20.04 D://Ubuntu20.04 D://ubuntu.tar
      ```

      
   
   2. 从本地vhdx导入(一般用于重装系统后导入)
   
      ```powershell
      wsl --import-in-place Ubuntu-20.04 D://Ubuntu20.04/ext4.vhdx
      ```
      
      

* 配置虚拟机参数

  在用户目录新建.wslconfig文件配置如下参数

  ```powershell
  [wsl2]
  # 自定义 Linux 内核的绝对路径
  #kernel=<path>
  # 给 WSL 2 虚拟机分配的内存大小
  memory=24G
  # 为 WSL 2 虚拟机分配的处理器核心数量
  processors=6
  # 为 WSL 2 虚拟机分配的交换空间，0 表示没有交换空间
  swap=8G
  # 自定义交换虚拟磁盘 vhd 的绝对路径
  #swapFile=<path>
  # 是否允许将 WSL 2 的端口转发到主机（默认为 true）
  localhostForwarding=true
  
  # `<path>` 必须是带反斜杠的绝对路径，例如 `C:\\Users\\kernel`
  # `<size>` 必须在后面加上单位，例如 8 GB 或 512 MB
  ```

  然后重新启动：`wsl --shutdown`   `wsl` 

* 其他命令

  ```powershell
  #查看已安装子系统状态
  wsl --list --verbose
  #设置默认wsl版本(wsl1,wsl2)
  wsl --set-default-version 2
  #设置默认子系统版本
  wsl --set-default Ubuntu-20.04
  #开启子系统
  wsl
  #关闭子系统
  wsl --shutdown
  ```

* 参考文档

  1. [WSL官方文档](https://learn.microsoft.com/zh-cn/windows/wsl/)

### 配置Ubuntu环境

* 更换系统软件源
  1. 打开系统源配置文件

     ```bash
     sudo vim /etc/apt/sources.list
     ```

     
  
  2. 替换软件源为清华源
  
      ```bash
      deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
      # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
      deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
      # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
      deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
      # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
      deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
      # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
      ```
  
  3. 更新升级软件源
  
     ```bash
     sudo apt-get update
     sudo apt-get upgrade
     ```
  
### 配置AOSP环境

* 安装必要软件
  
     ```bash
     sudo apt-get install git-core gnupg flex bison build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 libncurses5 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig python
     ```
  
* 配置git
  ```bash
  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"
  ```
  
* 配置repo
  
     1. 下载repo
        ```bash
        mkdir ~/bin
        curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o ~/bin/repo
        chmod +x ~/bin/repo
        ```
     
     2. 打开配置文件bashrc
        
        ````bash
        sudo vim ~/.bashrc
        ````
        
        
        
     3. 添加repo源与路径
        
        ```bash
        export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
        PATH=~/bin:$PATH
        ```
     
  4. 更新配置文件
     
     ````bash
     source ~/.bashrc
     ````
     
     
  
* 初始化AOSP仓库并且同步代码
  
  ```bash
  mkdir aosp 
  cd asop
  #初始化仓库,-b 指示分支，这里使用 android10
  repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-10.0.0_r41
  #同步远程代码
  repo sync
  ```
  
* 编译源码(内存最好32G太低可能会堆栈溢出)
  
  ```bash
  source build/envsetup.sh
  lunch aosp_x86_64-eng
  #数字最好根据自己电脑核心数来
  make -j6
  ```
  
* 启动模拟器
  
  ````bash
  emulator -verbose -cores 4 -show-kernel
  ````
  
  



