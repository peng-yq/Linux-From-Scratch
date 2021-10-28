# Linux From Scratch 11.0-systemed

LFS──Linux from Scratch，就是一种从网上直接下载源码，从头编译Linux的安装方式。它不是发行版，只是一个菜谱，告诉你到哪里去买菜（下载源码），怎么把这些生东西( raw code) 作成符合自己口味的菜肴──个性化的Linux，不单单是个性的桌面。

[官方中文文档](https://bf.mengyan1223.wang/lfs/zh_CN/11.0-systemd/LFS-SYSD-BOOK.html)     [官方英文文档](https://www.linuxfromscratch.org/lfs/view/stable/index.html)      [FAQ](https://www.linuxfromscratch.org/faq/)      [镜像](https://www.linuxfromscratch.org/mirrors.html)     [Linux操作教程](https://www.runoob.com/linux/linux-tutorial.html)     [LFS视频教程(English-version)](https://www.youtube.com/watch?v=9TYr1mCzMcg&t=909s)    [LFS视频教程(English-version & Ubuntu-as-Host-Machine)](https://www.youtube.com/watch?v=5tRJgDJC7kY)

## 第Ⅰ部分 引言

### 第1章 引言

#### 1.1 Linux

Linux is a **family** of **open-source Unix-like systems based on the Linux kernel**, an operating system kernel first released on September 17,1991,by Linus Torvalds.

Some Linux distributions: Unbuntu, Cent OS, Debian, Kali......

#### 1.2 基础命令

1. ls                          列出该目录下文件
2. cd                          切换目录
3. pwd                         显示当前目录位置
4. mkdir  *directory_name*       新建文件夹
5. rmkdir *directory_name*       删除文件夹
6. rm -r  *directory-name*       删除文件夹
7. touch  *file_name*            新建文件
8. rm     *file_name*            删除文件
9. cp *file_name* *file_name1*     在本目录下构造该文件的副本，并命名为*file_name1* 
10. cp *file_name* *directory_name* 将文件复制至另外一个目录 
11. cat                         查看文件内容
12. mv *file_name* *directory_name* 移动文件至另外一个目录 
13. mv *file_name* *file_name1*     重命名文件
14. ln *file_name* *link_name*      新建硬链接，即直接指向数据块，可通过ls -l发现两者一致
15. ln -s *file_name* *link_name*   新建软链接，即直接指向文件路径
16. gedit *file_name*             编辑文件，适用于GUI界面
17. vim   *file_name*             编辑文件，适用于Terminal，相较于vi，vim拥有更多的优势，[vim编辑命令](https://www.runoob.com/linux/linux-vim.html)
18. [Linux压缩命令](https://blog.csdn.net/weixin_44901564/article/details/99682926?ops_request_misc=&request_id=&biz_id=102&utm_term=Linux%E6%89%93%E5%8C%85&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-1-99682926.pc_search_result_hbase_insert&spm=1018.2226.3001.4187)

#### 1.3 学习LFS的意义

相较于安装传统的Linux发行版，学习以及构建属于自己的Linux，拥有以下优势：

1. 学习Linux系统内部是如何运行的，构建LFS系统的过程将展示Linux系统的工作原理，以及其他各组成部分的协作和依赖关系。
2. 允许用户更好的控制自己的系统，而不用依赖于其他人的Linux实现。
3. 允许用户创建独属于自己的非常紧凑的Linux系统，在安装传统的 Linux 发行版时，您往往不得不安装一大堆可能永远不会用到，甚至完全无法理解其必要性的程序。它们会浪费系统资源。
4. 自行定制的 Linux 系统在安全方面也具有优势。

## 第Ⅱ部分 准备工作

### 第2章 准备宿主系统

#### 2.1 宿主系统需求

以下搭建均在虚拟机中进行，发行版Linux为Ubuntu 20.04.2.0 LTS。

在搭建前需检查宿主机是否拥有必须的软件，并且版本达到最低要求（[软件及版本查看](https://bf.mengyan1223.wang/lfs/zh_CN/11.0-systemd/LFS-SYSD-BOOK.html#ch-partitioning-hostreqs)），切换至root用户并运行如下脚本。

```shell
cat > version-check.sh << "EOF"
#!/bin/bash
# Simple script to list version numbers of critical development tools
export LC_ALL=C
bash --version | head -n1 | cut -d" " -f2-4
MYSH=$(readlink -f /bin/sh)
echo "/bin/sh -> $MYSH"
echo $MYSH | grep -q bash || echo "ERROR: /bin/sh does not point to bash"
unset MYSH

echo -n "Binutils: "; ld --version | head -n1 | cut -d" " -f3-
bison --version | head -n1

if [ -h /usr/bin/yacc ]; then
  echo "/usr/bin/yacc -> `readlink -f /usr/bin/yacc`";
elif [ -x /usr/bin/yacc ]; then
  echo yacc is `/usr/bin/yacc --version | head -n1`
else
  echo "yacc not found" 
fi

bzip2 --version 2>&1 < /dev/null | head -n1 | cut -d" " -f1,6-
echo -n "Coreutils: "; chown --version | head -n1 | cut -d")" -f2
diff --version | head -n1
find --version | head -n1
gawk --version | head -n1

if [ -h /usr/bin/awk ]; then
  echo "/usr/bin/awk -> `readlink -f /usr/bin/awk`";
elif [ -x /usr/bin/awk ]; then
  echo awk is `/usr/bin/awk --version | head -n1`
else 
  echo "awk not found" 
fi

gcc --version | head -n1
g++ --version | head -n1
ldd --version | head -n1 | cut -d" " -f2-  # glibc version
grep --version | head -n1
gzip --version | head -n1
cat /proc/version
m4 --version | head -n1
make --version | head -n1
patch --version | head -n1
echo Perl `perl -V:version`
python3 --version
sed --version | head -n1
tar --version | head -n1
makeinfo --version | head -n1  # texinfo version
xz --version | head -n1

echo 'int main(){}' > dummy.c && g++ -o dummy dummy.c
if [ -x dummy ]
  then echo "g++ compilation OK";
  else echo "g++ compilation failed"; fi
rm -f dummy.c dummy
EOF
```

并输入`bash version-check.sh`运行脚本进行检查，本文使用的Ubuntu还有较多软件未下载，通过`sudo apt-get install package_name`下载即可。

<img src="Linux From Scratch.assets/image-20211010201124791.png"/>

本宿主机检查软件版本所发现的问题如下：

- Ubuntu默认的/bin/sh指向dash，此处需要改成bash，通过`dpkg-reconfigure dash`并选择否即可。
- Binutils缺乏，并且版本须在2.25~2.37。
- Bison缺乏，并且版本须高于2.7；缺乏/usr/bin/yacc。
- Gawk缺乏，并且版本须高于4.0.1；同时/usr/bin/awak->/usr/bin/gawk（本宿主机指向mawk）。
- GCC/G++缺乏，并且版本须在6.2~11.2.0。
- M4缺乏，并且版本须高于1.4.10
- Make缺乏，并且版本须高于4.0
- Texinfo缺乏，并且版本须高于4.7

> 官方文档中说明链接指向其他软件（如dash或mawk等）可能不会引发问题，但最好还是修改为相应的符号链接。

#### 2.2 创建新的分区

Ubuntu在默认配置下只拥有一个硬盘，此处我们添加一个新的硬盘来构建LFS系统，配置为20GB，SCSI，磁盘内容存储在单个文件中。

可通过Ubuntu自带的磁盘管理工具或通过命令`lsblk`查看本宿主机磁盘情况。

<img src="Linux From Scratch.assets/image-20211011200902437.png">

我们可以看到两块硬盘中sda被分成了3个分区，而sdb未被分区，因此我们选择sdb通过`sudo cfdisk /dev/sdb`命令进行操作，并选择GPT(GUID)分区表。

GUID分区表的优势：

- 支持2TB以上的大硬盘。
- 每个磁盘的分区个数几乎没有限制，唯一限制的是操作系统，比如Windows系统最多只允许划分128个分区。
- 分区大小几乎没有限制。
- 分区表自带备份。
- 每个分区可以有一个名称（不同于卷标）。

本文划分了3个分区，分别作为根分区（Root Partition），主分区（Main Partition），交换分区(Swap Partition)，具体容量分配如下。

<img src="Linux From Scratch.assets/image-20211011201016033.png">

#### 2.3 在分区上建立文件系统

现在我们建立好了空白分区，可以在分区上建立文件系统。LFS 可以使用 Linux 内核能够识别的任何文件系统，最常见的是 ext3 和 ext4。文件系统的选型是一个复杂的问题，要综合考虑分区的大小，以及其中所存储文件的特征。

本文中，根分区（sdb1）使用ext2；主分区（sdb2）采用ext4；此外由于新建了一个新的交换分区，我们需要初始化它。具体命令如下。

```shell
sudo mkfs -v -t ext2 /dev/sdb1
sudo mkfs -v -t ext4 /dev/sdb2
sudo mkswap /dev/sdb3
```

#### 2.4 设置$LFS环境变量

后续搭建中将经常使用环境变量LFS，因此我们需要将该变量定义且设置在所构建LFS使用的目录中。

```shell
sudo mkdir /mnt/lfs
export LFS=/mnt/lfs
echo $LFS
```

> 无论何时离开并重新进入了工作环境（例如切换至root或其他用户），需要执行`echo $LFS`确认环境变量的设置是否正确，若不正确则使用上述命令进行配置。

始终保持LFS变量正确，我们可以编辑`/root/.bash_profile`。

#### 2.5 挂载新的分区

我们已经在分区上建立了文件系统，为了访问分区，我们需要把分区挂载到选定的挂载点上。

**有关磁盘挂载问题的引入**

我们大多数人用惯了Windows系统，对linux系统中磁盘的管理就先入为主，不太好理解挂载这一动作。在linux系统中添加一块新磁盘后，要进行分区、格式化（分配文件系统）、挂载。当执行`ll /dev/sd*`时，可以看到相关的磁盘信息。大多数人会觉得硬盘添加，且分区、格式化了，可以用了。其实不然，还没有挂载好的硬盘就像新修的房子没有门一样，挂载就是将磁盘和某个文件夹捆绑在一起，做成一道磁盘的大门。

```shell
sudo mount -v -t ext4 /dev/sdb2 $LFS
sudo mkdir $LFS/boot
sudo mount -v -t ext2 /dev/sdb1 $LFS/boot
sudo /sbin/swapon -v /dev/sdb3
```

此时我们通过`cd $LFS`进入搭建文件夹，可以看到已完成分区已挂载完成。

**Lost+Found文件夹**

- 这个目录是使用标准的ext2/ext3档案系统格式才会产生的一个目录，目的在于当档案系统发生错误时， 将一些遗失的片段放置到这个目录下。这个目录通常会在分割槽的最顶层存在， 例如你加装一颗硬盘于/disk中，那在这个系统下就会自动产生一个这样的目录/disk/lost+found。
- lost+found这个目录一般情况下是空的，当系统非法关机后,如果你丢失了一些文件，在这里能找回来用来存放fsck过程中部分修复的文件的。
- lost+found：几乎每个被格式化过的Linux分区都会有，意外后找回的文件一般在这里面。
- 这个目录是储存发生意外后丢失的文件的。只有root用户才能打开。

### 第3章 软件包和补丁

#### 3.1 获取软件包和补丁

在搭建基本的Linux系统时需要下载必要的软件包，同时下载好的软件包和补丁需要保存在一个合适的位置，使得整个构建过程中都能容易地访问它们。另外，还需要一个工作目录，以便解压和编译软件包。我们可以将*$LFS/sources*既用于保存软件包和补丁，又作为工作目录。这样，我们需要的所有东西都在 LFS 分区中，因此在整个构建过程中都能够访问。

此外我们还需为该目录添加写入权限和 sticky 标志。“Sticky” 标志使得即使有多个用户对该目录有写入权限，也只有文件所有者能够删除其中的文件。

获取软件包以及补丁的方法：

- 在后续的两节中，单独下载这些文件。
- 从 https://www.linuxfromscratch.org/mirrors.html#files 中列出的某个镜像站下载包含所有所需文件的压缩包。
- 使用`wget`和下面描述的[wget-list](https://bf.mengyan1223.wang/lfs/zh_CN/11.0-systemd/wget-list)下载这些文件（将wget-list下载至$LFS/sources）。

```shell
sudo mkdir -v $LFS/sources
cd $LFS/sources
sudo chmod -v a+wt $LFS/sources
sudo wget --input-file=wget-list
```

在软件下载完毕后，我们通过[md5sums](https://bf.mengyan1223.wang/lfs/zh_CN/11.0-systemd/md5sums)用来检查所有软件包的正确性。将该文件复制到 *$LFS/sources*，运行以下命令即可得到检查结果（缺少的文件可在官方文档单个文件下载地址进行下载）：

```shell
pushd $LFS/sources
  md5sum -c md5sums
popd
```

在第6章中，会使用交叉编译器编译程序。为了将这个交叉编译器和其他程序分离，它会被安装在一个专门的目录。

```shell
sudo mkdir -pv $LFS/tools
cd $LFS
sudo ln -sv $LFS/tools /
```

### 第4章 最后准备工作

在LFS分区中需要进行的第一项任务是，创建一个有限的目录树，使得在第6章中编译的程序 (以及第5章中的glibc和libstdc++) 可以被安装到它们的最终位置。这样，在第8章中重新构建它们时，就能直接覆盖这些临时程序。

以root身份，执行以下命令创建所需的目录布局：

```shell
mkdir -pv $LFS/{etc,var} $LFS/usr/{bin,lib,sbin}

for i in bin lib sbin; do
  ln -sv usr/$i $LFS/$i
done

case $(uname -m) in
  x86_64) mkdir -pv $LFS/lib64 ;;
esac
```

在第6章中，会使用交叉编译器编译程序 (细节参见工具链技术说明一节)。为了将这个交叉编译器和其他程序分离，它会被安装在一个专门的目录。执行以下命令创建该目录：

```shell
mkdir -pv $LFS/tools
```

#### 4.1 添加LFS用户

在作为root用户登录时，一个微小的错误就可能损坏甚至摧毁整个系统。因此在后续的搭建中，我们将以非特权用户身份编译软件包。为了更容易地建立一个干净的工作环境，最好创建一个名为lfs的新用户，以及它从属于的一个新组 (组名也是lfs)，以便我们在安装过程中使用，此外我们还需将lfs设为$LFS中所有目录的所有者，使lfs对它们拥有完全访问权。

```shell
sudo groupadd lfs
sudo useradd -s /bin/bash -g lfs -m -k /dev/null lfs
sudo passwd lfs
sudo chown -v lfs $LFS/sources
chown -v lfs $LFS/{usr{,/*},lib,var,etc,bin,sbin,tools}
case $(uname -m) in
  x86_64) chown -v lfs $LFS/lib64 ;;
esac
```

上述命令的含义：

- -s /bin/bash

  设置bash为用户lfs的默认shell。

- -g lfs

  添加用户lfs到组lfs。

- -m

  为用户lfs创建一个主目录。

- -k /dev/null

  将模板目录设置为空设备文件，从而不从默认模板目录 (/etc/skel) 复制文件到新的主目录。

- lfs

  要创建的用户的名称。

在后续搭建中，我们通过`su - lfs`切换至用户lfs，-使得su启动一个登录shell，而不是非登录shell。

#### 4.2 配置环境

为了配置一个良好的工作环境，我们为bash创建两个新的启动脚本。以lfs的身份，执行以下命令，创建一个新的.bash_profile。

```shell
cat > ~/.bash_profile << "EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF
```

创建一个.bashrc文件。

```shell
cat > ~/.bashrc << "EOF"
set +h
umask 022
LFS=/mnt/lfs
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/usr/bin
if [ ! -L /bin ]; then PATH=/bin:$PATH; fi
PATH=$LFS/tools/bin:$PATH
CONFIG_SITE=$LFS/usr/share/config.site
export LFS LC_ALL LFS_TGT PATH CONFIG_SITE
EOF
```

最后，为了完全准备好编译临时工具的环境，指示shell读取刚才创建的配置文件。

```shell
source ~/.bash_profile
```

## 第Ⅲ部分 构建LFS交叉工具链和临时工具

### 前言 工具链技术说明

构建过程是基于*交叉编译*过程的。交叉编译通常被用于为一台与本机完全不同的计算机构建编译器及其工具链。这对于 LFS 并不严格必要，因为新系统运行的机器就是构建它时使用的。但是，交叉编译拥有一项重要优势，即任何交叉编译产生的程序都不可能依赖于宿主环境。

**本地编译**

本地编译可以理解为，在当前编译平台下，编译出来的程序只能放到当前平台下运行。平时我们常见的软件开发，都是属于本地编译：比如，我们在x86平台上，编写程序并编译成可执行程序。这种方式下，我们使用x86平台上的工具，开发针对x86平台本身的可执行程序，这个编译过程称为本地编译。

**交叉编译**

交叉编译可以理解为，在当前编译平台下，编译出来的程序能运行在体系结构不同的另一种目标平台上，但是编译平台本身却不能运行该程序：比如，我们在x86平台上，编写程序并编译成能运行在ARM平台的程序，编译得到的程序在x86平台上是不能运行的，必须放到ARM平台上才能运行。

**交叉编译的产生**

之所以要有交叉编译，主要原因是：

- Speed：目标平台的运行速度往往比主机慢得多，许多专用的嵌入式硬件被设计为低成本和低功耗，没有太高的性能
- Capability：整个编译过程是非常消耗资源的，嵌入式系统往往没有足够的内存或磁盘空间
- Availability：即使目标平台资源很充足，可以本地编译，但是第一个在目标平台上运行的本地编译器总需要通过交叉编译获得
- Flexibility：一个完整的Linux编译环境需要很多支持包，交叉编译使我们不需要花时间将各种支持包移植到目标板上

**交叉编译的常用术语**

- build：构建程序时使用的机器。
- host：将来会运行被构建的程序的机器。
- target：只有编译器使用这个术语，编译器为这台机器产生代码，它可能和buid和host都不同。

**SBU**

由于Linux From Scratch可以在许多不同系统上构建，我们无法直接给出估计时间，而是以标准构建单位（SBU）衡量时间。下面给出标准构建单位的测量方法。本文中构建的第一个软件包是Binutils，定义编译它需要的时间为标准构建单位，缩写为SBU。其他软件包的编译时间用SBU为单位表示。

### 第5章 编译交叉工具链

本章中编译的程序会被安装在$LFS/tools目录中，以将它们和后续章节中安装的文件分开。但是，本章中编译的库会被安装到它们的最终位置，因为这些库在我们最终要构建的系统中也存在。

> 若重新回到工作环境，请输入如下命令进行操作。

```shell
export LFS=/mnt/lfs
sudo mount -v -t ext4 /dev/sdb2 $LFS
sudo mount -v -t ext2 /dev/sdb1 $LFS/boot
sudo /sbin/swapon -v /dev/sdb3
su - lfs
```

#### 5.1 Binutils-2.37 - 第一遍

Binutils包含汇编器、链接器以及其他用于处理目标文件的工具。

估计构建时间：1 SBU

需要硬盘空间： 602 MB

```shell
cd $LFS/sources
tar -xf binutils-2.37.tar.xz
cd binutils-2.37

#建立专用目录用于构建Binutils
mkdir -v build
cd build

#准备编译Binutils
../configure --prefix=$LFS/tools \
             --with-sysroot=$LFS \
             --target=$LFS_TGT   \
             --disable-nls       \
             --disable-werror
             
#编译             
make

#安装该软件包
make install -j1
```

经过测试，本宿主机构建Binutils时间约为3min。

编译完成后将在*$LFS/tools*文件夹下生成*x86_64-lfs-linux-gnu*文件夹。

<img src="Linux From Scratch.assets/image-20211014113707822.png">

同时为了节约存储空间，我们可以通过`sudo rm -rf binutils-2.37 `删除解压的Binutils文件夹。

#### 5.2 GCC-11.2.0 - 第一遍

GCC软件包包含GNU编译器集合，其中有C和C++编译器。

估计构建时间： 12 SBU

需要硬盘空间： 3.4 GB

```shell
cd $LFS/sources
tar -xvf gcc-11.2.0.tar.xz
cd gcc-11.2.0

#解压相应依赖并重命名
tar -xf ../mpfr-4.1.0.tar.xz
mv -v mpfr-4.1.0 mpfr
tar -xf ../gmp-6.2.1.tar.xz
mv -v gmp-6.2.1 gmp
tar -xf ../mpc-1.2.1.tar.gz
mv -v mpc-1.2.1 mpc

#x86_64平台，设置存放64位库的默认目录为"lib"
case $(uname -m) in
  x86_64)
    sed -e '/m64=/s/lib64/lib/' \
        -i.orig gcc/config/i386/t-linux64
 ;;
esac

mkdir -v build
cd build

#准备编译GCC
../configure                                       \
    --target=$LFS_TGT                              \
    --prefix=$LFS/tools                            \
    --with-glibc-version=2.11                      \
    --with-sysroot=$LFS                            \
    --with-newlib                                  \
    --without-headers                              \
    --enable-initfini-array                        \
    --disable-nls                                  \
    --disable-shared                               \
    --disable-multilib                             \
    --disable-decimal-float                        \
    --disable-threads                              \
    --disable-libatomic                            \
    --disable-libgomp                              \
    --disable-libquadmath                          \
    --disable-libssp                               \
    --disable-libvtv                               \
    --disable-libstdcxx                            \
    --enable-languages=c,c++
    
#编译并进行安装
time make
make install

#创建一个完整版本的内部头文件
cd ..
cat gcc/limitx.h gcc/glimits.h gcc/limity.h > \
  `dirname $($LFS_TGT-gcc -print-libgcc-file-name)`/install-tools/include/limits.h
```

经过测试本宿主机编译完成GCC花费72min，orz。

#### 5.3 Linux-5.13.12 API 头文件

Linux API头文件（在linux-5.13.12.tar.xz中) 导出内核API供Glibc使用。

估计构建时间：0.1 SBU

需要硬盘空间：1.2GB

```shell
cd $LFS/sources
tar -xvf linux-5.13.12.tar.xz
cd linux-5.13.12

#确保软件包中没有遗留陈旧的文件
make mrproper

make headers
find usr/include -name '.*' -delete
rm usr/include/Makefile
cp -rv usr/include $LFS/usr

rm -rf linux-5.13.12
```

#### 5.4 Glibc-2.34

Glibc软件包包含主要的C语言库。它提供用于分配内存、检索目录、打开和关闭文件、读写文件、字符串处理、模式匹配、算术等用途的基本子程序。

估计构建时间：4.2 SBU

需要硬盘空间：744MB

```shell
cd $LFS/sources
tar -xvf glibc-2.34.tar.xz
cd glibc-2.34

#创建一个LSB兼容性符号链接。另外，对于x86_64，创建一个动态链接器正常工作所必须的符号链接
case $(uname -m) in
    i?86)   ln -sfv ld-linux.so.2 $LFS/lib/ld-lsb.so.3
    ;;
    x86_64) ln -sfv ../lib/ld-linux-x86-64.so.2 $LFS/lib64
            ln -sfv ../lib/ld-linux-x86-64.so.2 $LFS/lib64/ld-lsb-x86-64.so.3
    ;;
esac

#应用一个补丁，使得这些程序在FHS兼容的位置存放运行时数据
patch -Np1 -i ../glibc-2.34-fhs-1.patch

mkdir -v build
cd build

#确保将ldconfig和sln工具安装到/usr/sbin目录中
echo "rootsbindir=/usr/sbin" > configparms

#准备编译Glibc
../configure                             \
      --prefix=/usr                      \
      --host=$LFS_TGT                    \
      --build=$(../scripts/config.guess) \
      --enable-kernel=3.2                \
      --with-headers=$LFS/usr/include    \
      libc_cv_slibdir=/usr/lib

#编译同时检查变量
make
echo $LFS

#安装该软件包
make DESTDIR=$LFS install

#改正ldd脚本中硬编码的可执行文件加载器路径
sed '/RTLDLIST=/s@/usr@@g' -i $LFS/usr/bin/ldd

#最后的检查
echo 'int main(){}' > dummy.c
$LFS_TGT-gcc dummy.c
readelf -l a.out | grep '/ld-linux'

#若一切正常，最后输出为 [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]

#请理测试文件
rm -v dummy.c a.out

#安装limits.h头文件的安装
$LFS/tools/libexec/gcc/$LFS_TGT/11.2.0/install-tools/mkheaders

cd ../..
rm -rf glibc-2.34
```

#### 5.5 GCC-11.2.0中的Libstdc++-第一遍

Libstdc++是C++标准库。我们需要它才能编译C++代码（GCC的一部分用C++编写）。但在构建第一遍的GCC时我们不得不暂缓安装它，因为它依赖于当时还没有安装到目标目录的Glibc。

估计构建时间：0.4 SBU

需要硬盘空间：1.0 GB

```shell
cd $LFS/sources

#需要重新解压GCC源码包
tar -xvf gcc-11.2.0.tar.xz
cd gcc-11.2.0

mkdir -v build
cd build 

#准备编译Libstdc++
../libstdc++-v3/configure           \
    --host=$LFS_TGT                 \
    --build=$(../config.guess)      \
    --prefix=/usr                   \
    --disable-multilib              \
    --disable-nls                   \
    --disable-libstdcxx-pch         \
    --with-gxx-include-dir=/tools/$LFS_TGT/include/c++/11.2.0
    
time make
make DESTDIR=$LFS install

cd ../..
rm -rf gcc-11.2.0
```

### 第6章 交叉编译临时工具

#### 6.1 M4-1.4.19

M4软件包包含一个宏处理器。

估计构建时间：0.2 SBU
需要硬盘空间：32 MB

```shell
cd $LFS/sources
tar -xvf m4-1.4.19.tar.xz
cd m4-1.4.19

#准备编译M4
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)
            
 make
 make DESTDIR=$LFS install
 
 cd ..
 rm -rf m4-1.4.19
```

编译过程中出现如下错误。

<img src="Linux From Scratch.assets/image-20211017171122392.png">

检查发现缺少第5章Glibc编译完成后对limits.h头文件的安装（命令已补充在[5.4 Glibc-2.34](####5.4 Glibc-2.34)）。

#### 6.2 Ncurses-6.2

Ncurses软件包包含使用时不需考虑终端特性的字符屏幕处理函数库。

估计构建时间：0.7SBU

需要硬盘空间：48MB

```shell
cd $LFS/sources
tar -xvf ncurses-6.2.tar.gz
cd ncurses-6.2

#保证在配置是优先查找gawk命令
sed -i s/mawk// configure

#在宿主系统构建“tic”程序
mkdir build
pushd build
  ../configure
  make -C include
  make -C progs tic
popd

#准备编译ncurses
./configure --prefix=/usr                \
            --host=$LFS_TGT              \
            --build=$(./config.guess)    \
            --mandir=/usr/share/man      \
            --with-manpage-format=normal \
            --with-shared                \
            --without-debug              \
            --without-ada                \
            --without-normal             \
            --enable-widec
            
make
make DESTDIR=$LFS TIC_PATH=$(pwd)/build/progs/tic install
echo "INPUT(-lncursesw)" > $LFS/usr/lib/libncurses.so

cd ..
rm -rf ncurses-6.2
```

#### 6.3 Bash-5.1.8

Bash软件包包含Bourne-Again SHell。

估计构建时间：0.4 SBU

需要硬盘空间：64 MB

```shell
cd $LFS
tar -xvf bash-5.1.8.tar.gz
cd bash-5.1.8

#准备编译
./configure --prefix=/usr                   \
            --build=$(support/config.guess) \
            --host=$LFS_TGT                 \
            --without-bash-malloc
            
make
make DESTDIR=$LFS install

#为使用 sh 命令运行的shell的程序考虑，创建一个链接
ln -sv bash $LFS/bin/sh

cd ..
rm -rf bash-5.1.8
```

#### 6.4 Coreutils-8.32

Coreutils软件包包含用于显示和设定系统基本属性的工具。

估计构建时间：0.6 SBU

需要硬盘空间：151 MB

```shell
cd $LFS/sources
tar -xvf coreutils-8.32.tar.xz 
cd coreutils-8.32

#准备编译
./configure --prefix=/usr                     \
            --host=$LFS_TGT                   \
            --build=$(build-aux/config.guess) \
            --enable-install-program=hostname \
            --enable-no-install-program=kill,uptime
            
make
make DESTDIR=$LFS install

#将程序移动到它们最终安装时的正确位置。
mv -v $LFS/usr/bin/chroot                                     $LFS/usr/sbin
mkdir -pv $LFS/usr/share/man/man8
mv -v $LFS/usr/share/man/man1/chroot.1                        $LFS/usr/share/man/man8/chroot.8
sed -i 's/"1"/"8"/'                                           $LFS/usr/share/man/man8/chroot.8

cd ..
rm -rf coreutils-8.32
```

#### 6.5 Diffutils-3.8

Diffutils软件包包含显示文件或目录之间差异的程序。

估计构建时间：0.2 SBU

需要硬盘空间：28 MB

```shell
cd $LFS/sources
tar -xvf diffutils-3.8.tar.xz
cd diffutils-3.8

#准备编译
./configure --prefix=/usr --host=$LFS_TGT

make
make DESTDIR=$LFS install

cd ..
rm -rf diffutils-3.8
```

#### 6.6 File-5.40

File软件包包含用于确定给定文件类型的工具。

估计构建时间：0.2 SBU

需要硬盘空间：31 MB

```shell
cd $LFS/sources
tar -xvf file-5.40.tar.gz
cd file-5.40

#宿主系统file命令的版本必须和正在构建的软件包相同，才能在构建过程中创建必要的签名数据文件。运行以下命令，为宿主系统构建它
mkdir build
pushd build
  ../configure --disable-bzlib      \
               --disable-libseccomp \
               --disable-xzlib      \
               --disable-zlib
  make
popd

#准备编译
./configure --prefix=/usr --host=$LFS_TGT --build=$(./config.guess)

make FILE_COMPILE=$(pwd)/build/src/file
make DESTDIR=$LFS install

cd ..
rm -rf file-5.40
```

#### 6.7 Findutils-4.8.0

Findutils软件包包含用于查找文件的程序。这些程序能够递归地搜索目录树，以及创建、维护和搜索文件数据库 (一般比递归搜索快，但在数据库最近没有更新时不可靠)。

估计构建时间：0.2 SBU

需要硬盘空间：40 MB

```shell
cd $LFS/sources
tar -xvf findutils-4.8.0.tar.xz
cd findutils-4.8.0

#准备编译
./configure --prefix=/usr   \
            --localstatedir=/var/lib/locate \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)
            
make
make DESTDIR=$LFS install
cd ..
rm -rf findutils-4.8.0
```

#### 6.8 Gawk-5.1.0

Gawk软件包包含操作文本文件的程序。

估计构建时间：0.2 SBU

需要硬盘空间：43 MB

```shell
cd $LFS/sources
tar -xvf gawk-5.1.0.tar.xz
cd gawk-5.1.0

#确保不要安装一些没有必要的文件
sed -i 's/extras//' Makefile.in

#准备编译
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(./config.guess)
            
make
make DESTDIR=$LFS install
cd ..
rm -rf gawk-5.1.0
```

#### 6.9 Grep-3.7

Grep软件包包含在文件内容中进行搜索的程序。

估计构建时间：0.2 SBU

需要硬盘空间：25 MB

```shell
cd $LFS/sources
tar -xvf grep-3.7.tar.xz
cd grep-3.7

#准备编译
./configure --prefix=/usr   \
            --host=$LFS_TGT
            
make
make DESTDIR=$LFS install

cd ..
rm -rf grep-3.7
```

#### 6.10 Gzip-1.10

Gzip软件包包含压缩和解压缩文件的程序。

估计构建时间：0.1 SBU

需要硬盘空间：10 MB

```shell
cd $LFS/sources
tar -xvf gzip-1.10.tar.xz
cd gzip-1.10

#准备编译
./configure --prefix=/usr --host=$LFS_TGT

make
make DESTDIR=$LFS install

cd ..
rm -rf gzip-1.10
```

#### 6.11 Make-4.3

Make软件包包含一个程序，用于控制从软件包源代码生成可执行文件和其他非源代码文件的过程。

估计构建时间：0.1 SBU

需要硬盘空间：15 MB

```shell
cd $LFS/sources
tar -xvf make-4.3.tar.xz
cd make-4.3

#准备编译make
./configure --prefix=/usr   \
            --without-guile \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)
            
make
make DESTDIR=$LFS install

cd ..
rm -rf make-4.3
```

#### 6.12 Patch-2.7.6

Patch软件包包含通过应用 “补丁” 文件，修改或创建文件的程序，补丁文件通常是diff程序创建的。

估计构建时间：0.1 SBU

需要硬盘空间：12 MB

```shell
cd $LFS/sources
tar -xvf patch-2.7.6.tar.xz
cd patch-2.7.6

#准备编译
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)
            
make
make DESTDIR=$LFS install

cd ..
rm -rf patch-2.7.6
```

#### 6.13 Sed-4.8

Sed软件包包含一个流编辑器。

估计构建时间：0.1 SBU

需要硬盘空间：20 MB

```shell
cd $LFS/sources
tar -xvf sed-4.8.tar.xz
cd sed-4.8

#准备编译
./configure --prefix=/usr   \
            --host=$LFS_TGT
            
make
make DESTDIR=$LFS install

cd ..
rm -rf sed-4.8
```

#### 6.14 Tar-1.34

Tar软件包提供创建tar归档文件，以及对归档文件进行其他操作的功能。Tar可以对已经创建的归档文件进行提取文件，存储新文件，更新文件，或者列出文件等操作。

估计构建时间：0.2 SBU

需要硬盘空间：38 MB

```shell
cd $LFS/sources
tar -xvf tar-1.34.tar.xz
cd tar-1.34

#准备编译
./configure --prefix=/usr                     \
            --host=$LFS_TGT                   \
            --build=$(build-aux/config.guess)
            
make
make DESTDIR=$LFS install

cd ..
rm -rf tar-1.34
```

#### 6.15 Xz-5.2.5

Xz软件包包含文件压缩和解压缩工具，它能够处理lzma和新的xz压缩文件格式。使用xz压缩文本文件，可以得到比传统的gzip或bzip2更好的压缩比。

估计构建时间：0.1 SBU

需要硬盘空间：15 MB

```shell
cd $LFS/sources
tar -xvf xz-5.2.5.tar.xz
cd xz-5.2.5

#准备编译
./configure --prefix=/usr                     \
            --host=$LFS_TGT                   \
            --build=$(build-aux/config.guess) \
            --disable-static                  \
            --docdir=/usr/share/doc/xz-5.2.5
            
make
make DESTDIR=$LFS install

cd ..
rm -rf xz-5.2.5
```

#### 6.16 Binutils-2.37 - 第二遍

估计构建时间：1.3 SBU

需要硬盘空间：505 MB

```shell
cd $LFS/sources
tar -xvf binutils-2.37.tar.xz
cd binutils-2.37

mkdir -v build
cd build

#准备编译
../configure                   \
    --prefix=/usr              \
    --build=$(../config.guess) \
    --host=$LFS_TGT            \
    --disable-nls              \
    --enable-shared            \
    --disable-werror           \
    --enable-64-bit-bfd
    
make

#安装该软件包，并绕过导致libctf.so链接到宿主发行版zlib的问题
make DESTDIR=$LFS install -j1
install -vm755 libctf/.libs/libctf.so.0.0.0 $LFS/usr/lib

cd ../..
rm -rf binutils-2.37
```

#### 6.17 GCC-11.2.0 - 第二遍

估计构建时间：12 SBU

需要硬盘空间：3.3 GB

```shell
cd $LFS/sources
tar -xvf gcc-11.2.0.tar.xz
cd gcc-11.2.0

#解压GMP、MPFR、MPC，并重命名为GCC要求的目录名
tar -xf ../mpfr-4.1.0.tar.xz
mv -v mpfr-4.1.0 mpfr
tar -xf ../gmp-6.2.1.tar.xz
mv -v gmp-6.2.1 gmp
tar -xf ../mpc-1.2.1.tar.gz
mv -v mpc-1.2.1 mpc

如果是在x86_64上构建，修改64位库文件的默认目录名为 “lib”
case $(uname -m) in
  x86_64)
    sed -e '/m64=/s/lib64/lib/' -i.orig gcc/config/i386/t-linux64
  ;;
esac

mkdir -v build
cd build

#创建一个符号链接，以允许libgcc在构建时启用POSIX线程支持
mkdir -pv $LFS_TGT/libgcc
ln -s ../../../libgcc/gthr-posix.h $LFS_TGT/libgcc/gthr-default.h

#准备编译
../configure                                       \
    --build=$(../config.guess)                     \
    --host=$LFS_TGT                                \
    --prefix=/usr                                  \
    CC_FOR_TARGET=$LFS_TGT-gcc                     \
    --with-build-sysroot=$LFS                      \
    --enable-initfini-array                        \
    --disable-nls                                  \
    --disable-multilib                             \
    --disable-decimal-float                        \
    --disable-libatomic                            \
    --disable-libgomp                              \
    --disable-libquadmath                          \
    --disable-libssp                               \
    --disable-libvtv                               \
    --disable-libstdcxx                            \
    --enable-languages=c,c++
    
make
make DESTDIR=$LFS install

#创建符号链接
ln -sv gcc $LFS/usr/bin/cc

cd ../..
rm -rf gcc-11.2.0
```

### 第7章 进入Chroot并构建其他临时工具

本章将构建临时系统最后缺失的部分。为了隔离环境的正常工作，必须它与正在运行的内核之间建立一些通信机制。我们可以使用“chroot”环境进行构建，它与宿主系统除正在运行的内核外完全隔离。

> **本文后续所有命令都应该是在root用户登录的情况下完成，而不是lfs用户。此外，我们还要检查$LFS变量是否已经在root用户的环境下设置好。**

由于$LFS中整个目录树的所有者都是lfs，我们需要将其目录所有者改变为root。

```shell
chown -R root:root $LFS/{usr,lib,var,etc,bin,sbin,tools}
case $(uname -m) in
  x86_64) chown -R root:root $LFS/lib64 ;;
esac
```

#### 7.1 准备虚拟内核文件系统

内核对外提供了一些文件系统，以便自己和用户进行通信，它们是虚拟文件系统，并不占用磁盘空间，其内容保留在内存中。通过如下命令创建这些文件系统的挂载点。

```shell
mkdir -pv $LFS/{dev,proc,sys,run}
```

**挂载和填充/dev**

```shell
mount -v --bind /dev $LFS/dev
```

**挂载虚拟内核文件系统**

```shell
mount -v --bind /dev/pts $LFS/dev/pts
mount -vt proc proc $LFS/proc
mount -vt sysfs sysfs $LFS/sys
mount -vt tmpfs tmpfs $LFS/run

if [ -h $LFS/dev/shm ]; then
  mkdir -pv $LFS/$(readlink $LFS/dev/shm)
fi
```

#### 7.2 进入Chroot环境

现在已经准备好了所有继续构建其余工具时必要的软件包，可以进入chroot环境并完成剩余临时工具的安装。在安装最终的系统时，会继续使用这个chroot环境。以root用户身份，运行以下命令以进入当前只包含临时工具的chroot环境。

```shell
chroot "$LFS" /usr/bin/env -i   \
    HOME=/root                  \
    TERM="$TERM"                \
    PS1='(lfs chroot) \u:\w\$ ' \
    PATH=/usr/bin:/usr/sbin \
    /bin/bash --login +h
```

> bash提示符包含 I have no name!。这是正常的，因为现在还没有创建/etc/passwd文件。
>
> 本章剩余部分和后续各章中的命令都要在chroot环境中运行。

#### 7.3 创建目录

```shell
mkdir -pv /{boot,home,mnt,opt,srv}
mkdir -pv /etc/{opt,sysconfig}
mkdir -pv /lib/firmware
mkdir -pv /media/{floppy,cdrom}
mkdir -pv /usr/{,local/}{include,src}
mkdir -pv /usr/local/{bin,lib,sbin}
mkdir -pv /usr/{,local/}share/{color,dict,doc,info,locale,man}
mkdir -pv /usr/{,local/}share/{misc,terminfo,zoneinfo}
mkdir -pv /usr/{,local/}share/man/man{1..8}
mkdir -pv /var/{cache,local,log,mail,opt,spool}
mkdir -pv /var/lib/{color,misc,locate}

ln -sfv /run /var/run
ln -sfv /run/lock /var/lock

install -dv -m 0750 /root
install -dv -m 1777 /tmp /var/tmp
```

#### 7.4  创建必要的文件和符号链接

为了满足那些需要/etc/mtab的工具，执行以下命令，创建符号链接。

```shell
ln -sv /proc/self/mounts /etc/mtab
```

创建一个基本的/etc/hosts文件，一些测试套件，以及Perl的一个配置文件将会使用它。

```shell
cat > /etc/hosts << EOF
127.0.0.1  localhost $(hostname)
::1        localhost
EOF
```

为了使得root能正常登录，而且用户名“root”能被正常识别，必须在文件/etc/passwd和/etc/groups中写入相关的条目。

```shell
cat > /etc/passwd << "EOF"
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/dev/null:/bin/false
daemon:x:6:6:Daemon User:/dev/null:/bin/false
messagebus:x:18:18:D-Bus Message Daemon User:/run/dbus:/bin/false
systemd-bus-proxy:x:72:72:systemd Bus Proxy:/:/bin/false
systemd-journal-gateway:x:73:73:systemd Journal Gateway:/:/bin/false
systemd-journal-remote:x:74:74:systemd Journal Remote:/:/bin/false
systemd-journal-upload:x:75:75:systemd Journal Upload:/:/bin/false
systemd-network:x:76:76:systemd Network Management:/:/bin/false
systemd-resolve:x:77:77:systemd Resolver:/:/bin/false
systemd-timesync:x:78:78:systemd Time Synchronization:/:/bin/false
systemd-coredump:x:79:79:systemd Core Dumper:/:/bin/false
uuidd:x:80:80:UUID Generation Daemon User:/dev/null:/bin/false
systemd-oom:x:81:81:systemd Out Of Memory Daemon:/:/bin/false
nobody:x:99:99:Unprivileged User:/dev/null:/bin/false
EOF
```

创建/etc/group文件。

```shell
cat > /etc/group << "EOF"
root:x:0:
bin:x:1:daemon
sys:x:2:
kmem:x:3:
tape:x:4:
tty:x:5:
daemon:x:6:
floppy:x:7:
disk:x:8:
lp:x:9:
dialout:x:10:
audio:x:11:
video:x:12:
utmp:x:13:
usb:x:14:
cdrom:x:15:
adm:x:16:
messagebus:x:18:
systemd-journal:x:23:
input:x:24:
mail:x:34:
kvm:x:61:
systemd-bus-proxy:x:72:
systemd-journal-gateway:x:73:
systemd-journal-remote:x:74:
systemd-journal-upload:x:75:
systemd-network:x:76:
systemd-resolve:x:77:
systemd-timesync:x:78:
systemd-coredump:x:79:
uuidd:x:80:
systemd-oom:x:81:81:
wheel:x:97:
nogroup:x:99:
users:x:999:
EOF
```

为了移除 “I have no name!” 提示符，需要打开一个新shell。由于已经创建了文件/etc/passwd和/etc/group，用户名和组名现在就可以正常解析了。

```shell
exec /bin/bash --login +h
```

login、agetty和init等程序使用一些日志文件，以记录登录系统的用户和登录时间等信息。然而，这些程序不会创建不存在的日志文件。初始化日志文件，并为它们设置合适的访问权限。

```shell
touch /var/log/{btmp,lastlog,faillog,wtmp}
chgrp -v utmp /var/log/lastlog
chmod -v 664  /var/log/lastlog
chmod -v 600  /var/log/btmp
```

#### 7.5 GCC-11.2.0中的Libstdc++ - 第二遍

在构建第二遍的GCC时，我们不得不暂缓安装C++标准库，因为当时没有编译器能够编译它。我们不能使用那一节构建的编译器，因为它是一个本地编译器，不应在chroot外使用，否则可能导致编译产生的库被宿主系统组件污染。

估计构建时间：0.8 SBU

需要硬盘空间：1.1 GB

```shell
cd $LFS/sources
tar -xvf gcc-11.2.0.tar.xz
cd gcc-11.2.0

#创建一个符号链接，允许在GCC源码树中构建Libstdc++
ln -s gthr-posix.h libgcc/gthr-default.h

mkdir -v build
cd build

#准备编译
../libstdc++-v3/configure            \
    CXXFLAGS="-g -O2 -D_GNU_SOURCE"  \
    --prefix=/usr                    \
    --disable-multilib               \
    --disable-nls                    \
    --host=$(uname -m)-lfs-linux-gnu \
    --disable-libstdcxx-pch
    
make
make install

cd ../..
rm -rf gcc-11.2.0
```

#### 7.6 Gettext-0.21

Gettext软件包包含国际化和本地化工具，它们允许程序在编译时加入NLS(本地语言支持) 功能，使它们能够以用户的本地语言输出消息。

估计构建时间：1.8 SBU

需要硬盘空间：280 MB

```shell
cd $LFS/sources
tar -xvf gettext-0.21.tar.xz
cd gettext-0.21

#准备编译
./configure --disable-shared

make

#安装msgfmt，msgmerge，以及xgettext这三个程序
cp -v gettext-tools/src/{msgfmt,msgmerge,xgettext} /usr/bin

cd ..
rm -rf gettext-0.21
```

#### 7.7 Bison-3.7.6

Bison软件包包含语法分析器生成器。

估计构建时间：0.3 SBU

需要硬盘空间：50 MB

```shell
cd $LFS/sources
tar -xvf bison-3.7.6.tar.xz
cd bison-3.7.6

#准备编译
./configure --prefix=/usr \
            --docdir=/usr/share/doc/bison-3.7.6
            
make
make install

cd ..
rm -rf bison-3.7.6
```

#### 7.8 Perl-5.34.0

Perl软件包包含实用报表提取语言

估计构建时间：1.7 SBU

需要硬盘空间：272 MB

```shell
cd $LFS/sources
tar -xvf perl-5.34.tar.xz
cd perl-5.34

#准备编译
sh Configure -des                                        \
             -Dprefix=/usr                               \
             -Dvendorprefix=/usr                         \
             -Dprivlib=/usr/lib/perl5/5.34/core_perl     \
             -Darchlib=/usr/lib/perl5/5.34/core_perl     \
             -Dsitelib=/usr/lib/perl5/5.34/site_perl     \
             -Dsitearch=/usr/lib/perl5/5.34/site_perl    \
             -Dvendorlib=/usr/lib/perl5/5.34/vendor_perl \
             -Dvendorarch=/usr/lib/perl5/5.34/vendor_perl
             
make
make install

cd ..
rm -rf perl-5.34
```

#### 7.9 Python-3.9.6

Python3软件包包含Python开发环境。它被用于面向对象编程，编写脚本，为大型程序建立原型，或者开发完整的应用。

```shell
cd $LFS/sources
tar -xvf Python-3.9.6.tar.xz
cd Python-3.9.6

#准备编译 
./configure --prefix=/usr   \
            --enable-shared \
            --without-ensurepip
           
make
make install

cd ..
rm -rf Python-3.9.6
```

#### 7.10 Texinfo-6.8

Texinfo软件包包含阅读、编写和转换info页面的程序。

估计构建时间：0.3 SBU

需要硬盘空间：109 MB

```shell
cd $LFS/sources
tar -xvf texinfo-6.8.tar.xz
cd texinfo-6.8

#修复构建该软件包时出现的问题
sed -e 's/__attribute_nonnull__/__nonnull/' \
    -i gnulib/lib/malloc/dynarray-skeleton.c
    
#准备编译
./configure --prefix=/usr

make
make install

cd ..
rm -rf texinfo-6.8
```

#### 7.11 Util-linux-2.37.2

Util-linux软件包包含一些工具程序。

估计构建时间：0.7 SBU

需要硬盘空间：128 MB

```shell
cd $LFS/sources
tar -xvf util-linux-2.37.2
cd util-linux-2.37.2

mkdir -pv /var/lib/hwclock

#准备编译
./configure ADJTIME_PATH=/var/lib/hwclock/adjtime    \
            --libdir=/usr/lib    \
            --docdir=/usr/share/doc/util-linux-2.37.2 \
            --disable-chfn-chsh  \
            --disable-login      \
            --disable-nologin    \
            --disable-su         \
            --disable-setpriv    \
            --disable-runuser    \
            --disable-pylibmount \
            --disable-static     \
            --without-python     \
            runstatedir=/run
            
make
make install

cd ..
rm -rf util-linux-2.37.2
```

#### 7.12 请理和备份临时系统

**清理**

删除临时工具的文档，以防止它们进入最终构建的系统。其次，为了防止libtool.la文件对构建系统产生的问题，我们对其进行删除。最后我们还需要删除*/tools*目录，因为我们已不再需要它。

```shell
rm -rf /usr/share/{info,man,doc}/*
find /usr/{lib,libexec} -name \*.la -delete
rm -rf /tools
```

**备份**

```shell
#在chroot环境之外进行
exit

#进行备份之前，解除内核虚拟文件系统的挂载
umount $LFS/dev{/pts,}
umount $LFS/{sys,proc,run}

#备份
cd $LFS 
tar -cJpf $HOME/lfs-temp-tools-11.0-systemd.tar.xz .
```

***还原***

==请务必再次检查$LFS是否设置正确==

```shell
cd $LFS 
rm -rf ./* 
tar -xpf $HOME/lfs-temp-tools-11.0-systemd.tar.xz
```

若在备份或从备份进行恢复时退出了chroot环境，记得检查内核虚拟文件系统是否仍然处于挂载状态 (可以使用findmnt | grep $LFS进行检查)。若未进行挂载，请再次输入[7.1 准备虚拟内核文件系统](7.1 准备虚拟内核文件系统)、[7.2 进入Chroot环境](7.2 进入Chroot环境)的命令。

## 第Ⅳ部分 构建LFS系统

### 第8章 安装基本系统软件

由于在第7章中我们退出了chroot环境，因此我们首先需要将虚拟文件系统进行挂载并且再次进入chroot环境。

#### 8.1 Man-pages-5.13

Man-pages软件包包含2,200多个man页面。

估计构建时间：< 0.1 SBU

需要硬盘空间：4.7 MB

```shell
cd $LFS/sources
tar -xvf man-pages-5.13.tar.xz
cd man-pages-5.13

make prefix=/usr install

cd ..
rm -rf man-pages-5.13
```

#### 8.2 Iana-Etc-20210611

Iana-Etc软件包包含网络服务和协议的数据。

估计构建时间：< 0.1 SBU

需要硬盘空间：4.7 MB

```shell
cd $LFS/sources
tar -xvf iana-etc-20210611.tar.gz
cd iana-etc-20210611

cp services protocols /etc

cd ..
rm -rf iana-etc-20210611
```



#### 8.3 Glibc-2.34

**安装Glibc**

Glibc软件包包含主要的C语言库。它提供用于分配内存、检索目录、打开和关闭文件、读写文件、字符串处理、模式匹配、算术等用途的基本子程序。

估计构建时间：21 SBU

需要硬盘空间：2.4 GB

```shell
cd $LFS/sources
tar -xvf glibc-2.34.tar.xz
cd glibc-2.34

#修复上游开发者发现的一项安全问题
sed -e '/NOTIFY_REMOVED)/s/)/ \&\& data.attr != NULL)/' \
    -i sysdeps/unix/sysv/linux/mq_notify.c
    
#应用下列补丁，使得这些程序在 FHS 兼容的位置存储运行时数据
patch -Np1 -i ../glibc-2.34-fhs-1.patch

mkdir -v build
cd build

echo "rootsbindir=/usr/sbin" > configparms

#准备编译
../configure --prefix=/usr                            \
             --disable-werror                         \
             --enable-kernel=3.2                      \
             --enable-stack-protector=strong          \
             --with-headers=/usr/include              \
             libc_cv_slibdir=/usr/lib
             
make
make check

#防止警告
touch /etc/ld.so.conf

#修正Makefile
sed '/test-installation/s@$(PERL)@echo not running@' -i ../Makefile
make install

#改正 ldd 脚本中硬编码的可执行文件加载器路径
sed '/RTLDLIST=/s@/usr@@g' -i /usr/bin/ldd

#安装 nscd 的配置文件和运行时目录
cp -v ../nscd/nscd.conf /etc/nscd.conf
mkdir -pv /var/cache/nscd

#安装 nscd 的 systemd 支持文件
install -v -Dm644 ../nscd/nscd.tmpfiles /usr/lib/tmpfiles.d/nscd.conf
install -v -Dm644 ../nscd/nscd.service /usr/lib/systemd/system/nscd.service

#安装一些 locale，它们可以使得系统用不同语言响应用户请求
mkdir -pv /usr/lib/locale
localedef -i POSIX -f UTF-8 C.UTF-8 2> /dev/null || true
localedef -i cs_CZ -f UTF-8 cs_CZ.UTF-8
localedef -i de_DE -f ISO-8859-1 de_DE
localedef -i de_DE@euro -f ISO-8859-15 de_DE@euro
localedef -i de_DE -f UTF-8 de_DE.UTF-8
localedef -i el_GR -f ISO-8859-7 el_GR
localedef -i en_GB -f ISO-8859-1 en_GB
localedef -i en_GB -f UTF-8 en_GB.UTF-8
localedef -i en_HK -f ISO-8859-1 en_HK
localedef -i en_PH -f ISO-8859-1 en_PH
localedef -i en_US -f ISO-8859-1 en_US
localedef -i en_US -f UTF-8 en_US.UTF-8
localedef -i es_ES -f ISO-8859-15 es_ES@euro
localedef -i es_MX -f ISO-8859-1 es_MX
localedef -i fa_IR -f UTF-8 fa_IR
localedef -i fr_FR -f ISO-8859-1 fr_FR
localedef -i fr_FR@euro -f ISO-8859-15 fr_FR@euro
localedef -i fr_FR -f UTF-8 fr_FR.UTF-8
localedef -i is_IS -f ISO-8859-1 is_IS
localedef -i is_IS -f UTF-8 is_IS.UTF-8
localedef -i it_IT -f ISO-8859-1 it_IT
localedef -i it_IT -f ISO-8859-15 it_IT@euro
localedef -i it_IT -f UTF-8 it_IT.UTF-8
localedef -i ja_JP -f EUC-JP ja_JP
localedef -i ja_JP -f SHIFT_JIS ja_JP.SIJS 2> /dev/null || true
localedef -i ja_JP -f UTF-8 ja_JP.UTF-8
localedef -i nl_NL@euro -f ISO-8859-15 nl_NL@euro
localedef -i ru_RU -f KOI8-R ru_RU.KOI8-R
localedef -i ru_RU -f UTF-8 ru_RU.UTF-8
localedef -i se_NO -f UTF-8 se_NO.UTF-8
localedef -i ta_IN -f UTF-8 ta_IN.UTF-8
localedef -i tr_TR -f UTF-8 tr_TR.UTF-8
localedef -i zh_CN -f GB18030 zh_CN.GB18030
localedef -i zh_HK -f BIG5-HKSCS zh_HK.BIG5-HKSCS
localedef -i zh_TW -f UTF-8 zh_TW.UTF-8

#后续会进行安装的两个locale
localedef -i POSIX -f UTF-8 C.UTF-8 2> /dev/null || true
localedef -i ja_JP -f SHIFT_JIS ja_JP.SIJS 2> /dev/null || true
```

**配置Glibc**

由于 Glibc 的默认值在网络环境下不能很好地工作，需要创建配置文件/etc/nsswitch.conf。

```shell
cat > /etc/nsswitch.conf << "EOF"
# Begin /etc/nsswitch.conf

passwd: files
group: files
shadow: files

hosts: files dns
networks: files

protocols: files
services: files
ethers: files
rpc: files

# End /etc/nsswitch.conf
EOF
```

添加时区数据。

```shell
tar -xf ../../tzdata2021a.tar.gz

ZONEINFO=/usr/share/zoneinfo
mkdir -pv $ZONEINFO/{posix,right}

for tz in etcetera southamerica northamerica europe africa antarctica  \
          asia australasia backward; do
    zic -L /dev/null   -d $ZONEINFO       ${tz}
    zic -L /dev/null   -d $ZONEINFO/posix ${tz}
    zic -L leapseconds -d $ZONEINFO/right ${tz}
done

cp -v zone.tab zone1970.tab iso3166.tab $ZONEINFO
zic -d $ZONEINFO -p America/New_York
unset ZONEINFO

#运行脚本以确定时区
tzselect

#创建/etc/localtime
ln -sfv /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

配置动态加载器。

```shell
cat > /etc/ld.so.conf << "EOF"
# Begin /etc/ld.so.conf
/usr/local/lib
/opt/lib

EOF

cat >> /etc/ld.so.conf << "EOF"
# Add an include directory
include /etc/ld.so.conf.d/*.conf

EOF
mkdir -pv /etc/ld.so.conf.d

cd ../..
rm -rf glibc-2.34
```

#### 8.4 zlib-1.2.11

Zlib软件包包含一些程序使用的压缩和解压缩子程序。

估计构建时间：< 0.1 SBU

需要硬盘空间：5.0 MB

```shell
cd $LFS/sources
tar -xvf zlib-1.2.11.tar.xz
cd zlib-1.2.11

./configure --prefix=/usr
make
make check
make install
rm -fv /usr/lib/libz.a

cd ..
rm -rf zlib-1.2.11
```

#### 8.5 Bzip2-1.0.8

Bzip2软件包包含用于压缩和解压缩文件的程序。使用bzip2压缩文本文件可以获得比传统的 gzip优秀许多的压缩比。

估计构建时间：< 0.1 SBU

需要硬盘空间：7.2 MB

```shell
cd $LFS/sources
tar -xvf bzip2-1.0.8.tar.gz
cd bzip2-1.0.8

#应用补丁
patch -Np1 -i ../bzip2-1.0.8-install_docs-1.patch

#保证安装的符号链接是相对的
sed -i 's@\(ln -s -f \)$(PREFIX)/bin/@\1@' Makefile

#确保man页面被安装到正确位置
sed -i "s@(PREFIX)/man@(PREFIX)/share/man@g" Makefile

#准备编译
make -f Makefile-libbz2_so
make clean

make
make PREFIX=/usr install
cp -av libbz2.so.* /usr/lib
ln -sv libbz2.so.1.0.8 /usr/lib/libbz2.so

cp -v bzip2-shared /usr/bin/bzip2
for i in /usr/bin/{bzcat,bunzip2}; do
  ln -sfv bzip2 $i
done

rm -fv /usr/lib/libbz2.a

cd ..
rm -rf bzip2-1.0.8
```

#### 8.6 Xz-5.2.5

Xz软件包包含文件压缩和解压缩工具，它能够处理lzma和新的xz压缩文件格式。使用xz压缩文本文件，可以得到比传统的gzip或bzip2更好的压缩比。

估计构建时间：0.2 SBU

需要硬盘空间：15 MB

```shell
cd $LFS/sources
tar -xvf xz-5.2.5.tar.xz

cd xz-5.2.5

#准备编译
./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/xz-5.2.5
            
make
make check
make install

cd ..
rm -rf xz-5.2.5
```

#### 8.7 Zstd-1.5.0

Zstandard是一种实时压缩算法，提供了较高的压缩比。它具有很宽的压缩比/速度权衡范围，同时支持具有非常快速的解压缩。

估计构建时间：1.4 SBU

需要硬盘空间：60 MB

```shell
cd $LFS/sources
tar -xvf zstd-1.5.0.tar.gz
cd zstd-1.5.0

make
make check

make prefix=/usr install
rm -v /usr/lib/libzstd.a

cd ..
rm -rf zstd-1.5.0
```

#### 8.8 File-5.40

File软件包包含用于确定给定文件类型的工具。

估计构建时间：0.1 SBU

需要硬盘空间：15 MB

```shell
cd $LFS/sources
tar -xvf file-5.40.tar.gz
cd file-5.40

./configure --prefix=/usr
make
make check
make install

cd ..
rm -rf file-5.40
```

#### 8.9 Readline-8.1

Readline软件包包含一些提供命令行编辑和历史记录功能的库。

估计构建时间：0.1 SBU

需要硬盘空间：15 MB

```shell
cd $LFS/sources
tar -xvf readline-8.1.tar.gz 
cd readline-8.1

sed -i '/MV.*old/d' Makefile.in
sed -i '/{OLDSUFF}/c:' support/shlib-install

./configure --prefix=/usr    \
            --disable-static \
            --with-curses    \
            --docdir=/usr/share/doc/readline-8.1
            
make SHLIB_LIBS="-lncursesw"
make SHLIB_LIBS="-lncursesw" install
install -v -m644 doc/*.{ps,pdf,html,dvi} /usr/share/doc/readline-8.1

cd ..
rm -rf readline-8.1
```

#### 8.10 M4-1.4.19

M4软件包包含一个宏处理器。

估计构建时间：0.7 SBU

需要硬盘空间：48 MB

```shell
cd $LFS/sources
tar -xvf m4-1.4.19.tar.xz
cd m4-1.4.19

./configure --prefix=/usr
make
make check
make install

cd ..
rm -rf m4-1.4.19
```

#### 8.11 Bc-5.0.0

Bc软件包包含一个任意精度数值处理语言。

估计构建时间：< 0.1 SBU

需要硬盘空间：6.7 MB

```shell
cd $LFS/sources
tar -xvf bc-5.0.0.tar.xz
cd bc-5.0.0

CC=gcc ./configure --prefix=/usr -G -O3
make
make test
make install

cd ..
rm -rf bc-5.0.0
```

#### 8.12 Flex-2.6.4

Flex软件包包含一个工具，用于生成在文本中识别模式的程序。

估计构建时间：0.4 SBU

需要硬盘空间：32 MB

```shell
cd $LFS/sources
tar -xvf flex-2.6.4.tar.gz
cd flex-2.6.4

#准备编译
./configure --prefix=/usr \
            --docdir=/usr/share/doc/flex-2.6.4 \
            --disable-static
            
make 
make check
make install

ln -sv flex /usr/bin/lex
cd ..
rm -rf flex-2.6.4
```

#### 8.13 Tcl-8.6.11

Tcl软件包包含工具命令语言，它是一个可靠的通用脚本语言。Except软件包是用Tcl语言编写的。

```shell
cd $LFS/sources
tar -xvf tcl8.6.11-src.tar.gz 
cd tcl8.6.11

#解压文档
tar -xf ../tcl8.6.11-html.tar.gz --strip-components=1

SRCDIR=$(pwd)
cd unix
./configure --prefix=/usr           \
            --mandir=/usr/share/man \
            $([ "$(uname -m)" = x86_64 ] && echo --enable-64bit)
            
#构建该软件包
make

sed -e "s|$SRCDIR/unix|/usr/lib|" \
    -e "s|$SRCDIR|/usr/include|"  \
    -i tclConfig.sh

sed -e "s|$SRCDIR/unix/pkgs/tdbc1.1.2|/usr/lib/tdbc1.1.2|" \
    -e "s|$SRCDIR/pkgs/tdbc1.1.2/generic|/usr/include|"    \
    -e "s|$SRCDIR/pkgs/tdbc1.1.2/library|/usr/lib/tcl8.6|" \
    -e "s|$SRCDIR/pkgs/tdbc1.1.2|/usr/include|"            \
    -i pkgs/tdbc1.1.2/tdbcConfig.sh

sed -e "s|$SRCDIR/unix/pkgs/itcl4.2.1|/usr/lib/itcl4.2.1|" \
    -e "s|$SRCDIR/pkgs/itcl4.2.1/generic|/usr/include|"    \
    -e "s|$SRCDIR/pkgs/itcl4.2.1|/usr/include|"            \
    -i pkgs/itcl4.2.1/itclConfig.sh

unset SRCDIR

make test
make install
chmod -v u+w /usr/lib/libtcl8.6.so
make install-private-headers
ln -sfv tclsh8.6 /usr/bin/tclsh
mv /usr/share/man/man3/{Thread,Tcl_Thread}.3

cd ../..
rm -rf tcl8.6.11
```

#### 8.14 Expect-5.45.4

Expect软件包包含通过脚本控制的对话，自动化telnet，ftp，passwd，fsck，rlogin，以及tip等交互应用的工具。Expect对于测试这类程序也很有用，它简化了这类通过其他方式很难完成的工作。DejaGnu框架是使用Expect编写的。

估计构建时间：0.2 SBU

需要硬盘空间：3.9 MB

```shell
cd $LFS/sources
tar -xvf expect5.45.4.tar.gz 
cd expect5.45.4

#准备编译
./configure --prefix=/usr           \
            --with-tcl=/usr/lib     \
            --enable-shared         \
            --mandir=/usr/share/man \
            --with-tclinclude=/usr/include
            
make
make test
make install
ln -svf expect5.45.4/libexpect5.45.4.so /usr/lib

cd ..
rm -rf expect5.45.4
```

#### 8.15 DejaGNU-1.6.3

DejaGnu包含使用GNU工具运行测试套件的框架。它是用expect编写的，后者又使用Tcl(工具命令语言)。

估计构建时间：< 0.1 SBU

需要硬盘空间：6.9 MB

```shell
cd $LFS/sources
tar -xvf dejagnu-1.6.3.tar.gz 
cd dejagnu-1.6.3

mkdir -v build
cd build

../configure --prefix=/usr
makeinfo --html --no-split -o doc/dejagnu.html ../doc/dejagnu.texi
makeinfo --plaintext       -o doc/dejagnu.txt  ../doc/dejagnu.texi

make install
install -v -dm755  /usr/share/doc/dejagnu-1.6.3
install -v -m644   doc/dejagnu.{html,txt} /usr/share/doc/dejagnu-1.6.3

make check

cd ../..
rm -rf dejagnu-1.6.3
```

#### 8.16 Binutils-2.37

Binutils包含汇编器、链接器以及其他用于处理目标文件的工具。

估计构建时间：6.3 SBU

需要硬盘空间：4.5 GB

```shell
cd $LFS/sources
tar -xvf binutils-2.37.tar.xz 
cd binutils-2.37

#进行简单测试，确认伪终端在chroot环境中能正常工作（应输出spawn ls）
expect -c "spawn ls"

#应用补丁
patch -Np1 -i ../binutils-2.37-upstream_fix-1.patch

sed -i '63d' etc/texi2pod.pl
find -name \*.1 -delete

mkdir -v build
cd build

../configure --prefix=/usr       \
             --enable-gold       \
             --enable-ld=default \
             --enable-plugins    \
             --enable-shared     \
             --disable-werror    \
             --enable-64-bit-bfd \
             --with-system-zlib
             
make tooldir=/usr
make -k check
make tooldir=/usr install -j1
rm -fv /usr/lib/lib{bfd,ctf,ctf-nobfd,opcodes}.a

cd ../..
rm -rf binutils-2.37
```

#### 8.17 GMP-6.2.1

GMP软件包包含提供任意精度算术函数的数学库。

估计构建时间：1.0 SBU

需要硬盘空间：52 MB

```shell
cd $LFS/sources
tar -xvf gmp-6.2.1.tar.xz
cd gmp-6.2.1

./configure --prefix=/usr    \
            --enable-cxx     \
            --disable-static \
            --docdir=/usr/share/doc/gmp-6.2.1
            
make
make html

make check 2>&1 | tee gmp-check-log
awk '/# PASS:/{total+=$3} ; END{print total}' gmp-check-log

make install
make install-html

cd ..
rm -rf gmp-6.2.1
```

#### 8.18 MPFR-4.1.0

MPFR软件包包含多精度数学函数。

估计构建时间：0.8 SBU

需要硬盘空间：38 MB

```shell
cd $LFS/sources
tar -xvf mpfr-4.1.0.tar.xz
cd mpfr-4.1.0

#准备编译
./configure --prefix=/usr        \
            --disable-static     \
            --enable-thread-safe \
            --docdir=/usr/share/doc/mpfr-4.1.0
            
make
make html
make check
make install
make install-html

cd ..
rm -rf mpfr-4.1.0
```

#### 8.19 MPC-1.2.1

MPC软件包包含一个任意高精度，且舍入正确的复数算术库。

估计构建时间：0.3 SBU

需要硬盘空间：21 MB

```shell
cd $LFS/sources
tar -xvf mpc-1.2.1.tar.gz
cd mpc-1.2.1

#准备编译
./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/mpc-1.2.1
            
make
make html
make check
make install
make install-html

cd ..
rm -rf mpc-1.2.1
```

#### 8.20 Attr-2.5.1

Attr软件包包含管理文件系统对象扩展属性的工具。

估计构建时间：< 0.1 SBU

需要硬盘空间：4.1 MB

```shell
cd $LFS/sources
tar -xvf attr-2.5.1.tar.gz
cd attr-2.5.1

#准备编译
./configure --prefix=/usr     \
            --disable-static  \
            --sysconfdir=/etc \
            --docdir=/usr/share/doc/attr-2.5.1
            
make
make check
make install

cd ..
rm -rf attr-2.5.1
```

#### 8.21 Acl-2.3.1

Acl软件包包含管理访问控制列表的工具，访问控制列表能够更细致地自由定义文件和目录的访问权限。

估计构建时间：0.1 SBU

需要硬盘空间：6.1 MB

```shell
cd $LFS/sources
tar -xvf acl-2.3.1.tar.xz
cd acl-2.3.1

#准备编译
./configure --prefix=/usr         \
            --disable-static      \
            --docdir=/usr/share/doc/acl-2.3.1
            
make
make install

cd ..
rm -rf acl-2.3.1
```

#### 8.22 Libcap-2.53

Libcap软件包为Linux内核提供的POSIX 1003.1e权能字实现用户接口。这些权能字是 root用户的最高特权分割成的一组不同权限。

```shell
cd $LFS/sources
tar -xvf libcap-2.53.tar.xz
cd libcap-2.53

#防止静态库的安装
sed -i '/install -m.*STA/d' libcap/Makefile

make prefix=/usr lib=lib
make test
make prefix=/usr lib=lib install
chmod -v 755 /usr/lib/lib{cap,psx}.so.2.53

cd ..
rm -rf libcap-2.53
```

#### 8.23 Shadow-4.9

Shadow软件包包含安全地处理密码的程序。

估计构建时间：0.2 SBU

需要硬盘空间：45 MB

```shell
cd $LFS/sources
tar -xvf shadow-4.9.tar.xz
cd shadow-4.9

#避免安装已提供的man页面
sed -i 's/groups$(EXEEXT) //' src/Makefile.in
find man -name Makefile.in -exec sed -i 's/groups\.1 / /'   {} \;
find man -name Makefile.in -exec sed -i 's/getspnam\.3 / /' {} \;
find man -name Makefile.in -exec sed -i 's/passwd\.5 / /'   {} \;

#修复错误
sed -e "224s/rounds/min_rounds/" -i libmisc/salt.c

#准备编译
touch /usr/bin/passwd
./configure --sysconfdir=/etc \
            --with-group-name-max-length=32
            
make
make exec_prefix=/usr install
make -C man install-man
mkdir -p /etc/default
useradd -D --gid 999

#下面进行配置shadow

#对用户密码启用shadow加密
pwconv

#对组密码启用shadow加密
grpconv

#设定root密码
passwd root

cd ..
rm -rf shadow-4.9
```

#### 8.24 GCC-11.2.0

GCC软件包包含 GNU 编译器集合，其中有C和C++编译器。

估计构建时间：164 SBU

需要硬盘空间：4.3 GB

```shell
cd $LFS/sources
tar -xvf gcc-11.2.0.tar.xz
cd gcc-11.2.0

#修复一些问题
sed -e '/static.*SIGSTKSZ/d' \
    -e 's/return kAltStackSize/return SIGSTKSZ * 4/' \
    -i libsanitizer/sanitizer_common/sanitizer_posix_libcdep.cpp
    
#修改存放64位库的默认路径为“lib”
case $(uname -m) in
  x86_64)
    sed -e '/m64=/s/lib64/lib/' \
        -i.orig gcc/config/i386/t-linux64
  ;;
esac

mkdir -v build
cd build

#准备编译
../configure --prefix=/usr            \
             LD=ld                    \
             --enable-languages=c,c++ \
             --disable-multilib       \
             --disable-bootstrap      \
             --with-system-zlib
             
make

#增加栈空间
ulimit -s 32768

#以非特权用户身份测试编译结果
chown -Rv tester . 
su tester -c "PATH=$PATH make -k check"

#查看测试结果的摘要
../contrib/test_summary

#安装软件包，并移除一个不需要的目录
make install
rm -rf /usr/lib/gcc/$(gcc -dumpmachine)/11.2.0/include-fixed/bits/

#将GCC的所有者修改为root用户和组
chown -v -R root:root \
    /usr/lib/gcc/*linux-gnu/11.2.0/include{,-fixed}
    
ln -svr /usr/bin/cpp /usr/lib
ln -sfv ../../libexec/gcc/$(gcc -dumpmachine)/11.2.0/liblto_plugin.so \
        /usr/lib/bfd-plugins/
        
echo 'int main(){}' > dummy.c
cc dummy.c -v -Wl,--verbose &> dummy.log
readelf -l a.out | grep ': /lib'

grep -o '/usr/lib.*/crt[1in].*succeeded' dummy.log

grep -B4 '^ /usr/include' dummy.log

grep 'SEARCH.*/usr/lib' dummy.log |sed 's|; |\n|g'

grep "/lib.*/libc.so.6 " dummy.log

grep found dummy.log

#删除测试文件
rm -v dummy.c a.out dummy.log

#移动一个位置不正确的文件
mkdir -pv /usr/share/gdb/auto-load/usr/lib
mv -v /usr/lib/*gdb.py /usr/share/gdb/auto-load/usr/lib

cd ../..
rm -rf gcc-11.2.0
```

#### 8.25 Pkg-config-0.29.2

pkg-config软件包提供一个在软件包安装的配置和编译阶段，向构建工具传递头文件和/或库文件路径的工具。

估计构建时间：0.3 SBU

需要硬盘空间：29 MB

```shell
cd $LFS/sources
tar -xvf pkg-config-0.29.2.tar.gz
cd pkg-config-0.29.2

#准备编译
./configure --prefix=/usr              \
            --with-internal-glib       \
            --disable-host-tool        \
            --docdir=/usr/share/doc/pkg-config-0.29.2
            
make
make check
make install

cd ..
rm -rf pkg-config-0.29.2
```

#### 8.26 Ncurses-6.2

Ncurses软件包包含使用时不需考虑终端特性的字符屏幕处理函数库。

估计构建时间：0.4 SBU

需要硬盘空间：34 MB

```shell
cd $LFS/sources
tar -xvf ncurses-6.2.tar.gz
cd ncurses-6.2

#准备编译
./configure --prefix=/usr           \
            --mandir=/usr/share/man \
            --with-shared           \
            --without-debug         \
            --without-normal        \
            --enable-pc-files       \
            --enable-widec
            
make
make install

#通过使用符号链接和链接脚本，诱导它们链接到宽字符库
for lib in ncurses form panel menu ; do
    rm -vf                    /usr/lib/lib${lib}.so
    echo "INPUT(-l${lib}w)" > /usr/lib/lib${lib}.so
    ln -sfv ${lib}w.pc        /usr/lib/pkgconfig/${lib}.pc
done

#确保那些在构建时寻找 -lcurses 的老式程序仍然能够构建
rm -vf                     /usr/lib/libcursesw.so
echo "INPUT(-lncursesw)" > /usr/lib/libcursesw.so
ln -sfv libncurses.so      /usr/lib/libcurses.so

#删除一个 configure 脚本未处理的静态库
rm -fv /usr/lib/libncurses++w.a

#安装Ncurses文档
mkdir -v       /usr/share/doc/ncurses-6.2
cp -v -R doc/* /usr/share/doc/ncurses-6.2

cd ..
rm -rf ncurses-6.2
```

#### 8.27 Sed-4.8

Sed软件包包含一个流编辑器。

估计构建时间：0.5 SBU

需要硬盘空间：30 MB

```shell
cd $LFS/sources
tar -xvf sed-4.8.tar.xz
cd sed-4.8

#准备编译
./configure --prefix=/usr

make
make html

#测试编译结果
chown -Rv tester .
su tester -c "PATH=$PATH make check"

#安装该软件机器文档
make install
install -d -m755           /usr/share/doc/sed-4.8
install -m644 doc/sed.html /usr/share/doc/sed-4.8

cd ,,
rm -rf sed-4.8
```

#### 8.28 Psmisc-23.4

Psmisc软件包包含显示正在运行的进程信息的程序。

估计构建时间：< 0.1 SBU

需要硬盘空间：5.6 MB

```shell
cd $LFS/sources
tar -xvf psmisc-23.4.tar.xz
cd psmisc-23.4

#准备编译
./configure --prefix=/usr

make
make install

cd ..
rm -rf psmisc-23.4
```

#### 8.29 Gettext-0.21

Gettext软件包包含国际化和本地化工具，它们允许程序在编译时加入NLS(本地语言支持) 功能，使它们能够以用户的本地语言输出消息。

估计构建时间：2.9 SBU

需要硬盘空间：231 MB

```shell
cd $LFS/sources
tar -xvf gettext-0.21.tar.xz
cd gettext-0.21

#准备编译
./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/gettext-0.21
            
make
make check
make install
chmod -v 0755 /usr/lib/preloadable_libintl.so

cd ..
rm -rf gettext-0.21
```

#### 8.30 Bison-3.7.6

估计构建时间：6.3 SBU

需要硬盘空间：53 MB

```shell
cd $LFS/sources
tar -xvf bison-3.7.6.tar.xz
cd bison-3.7.6

#准备编译
./configure --prefix=/usr --docdir=/usr/share/doc/bison-3.7.6

make
make check
make install

cd ..
rm -rf bison-3.7.6
```

#### 8.31 Grep-3.7

Grep软件包包含在文件内容中进行搜索的程序。

估计构建时间：0.8 SBU

需要硬盘空间：36 MB

```shell
cd $LFS/sources
tar -xvf grep-3.7.tar.xz
cd grep-3.7

#准备编译
./configure --prefix=/usr

make
make check
make install

cd ..
rm -rf grep-3.7
```

#### 8.32 Bash-5.1.8

Bash软件包包含Bourne-Again SHell。

估计构建时间：1.6 SBU

需要硬盘空间：50 MB

```shell
cd $LFS/sources
tar -xvf bash-5.1.8.tar.gz
cd bash-5.1.8

#准备编译
./configure --prefix=/usr                      \
            --docdir=/usr/share/doc/bash-5.1.8 \
            --without-bash-malloc              \
            --with-installed-readline

make
chown -Rv tester .

#测试
su -s /usr/bin/expect tester << EOF
set timeout -1
spawn make tests
expect eof
lassign [wait] _ _ _ value
exit $value
EOF

make install
exec /bin/bash --login +h

cd ..
rm -rf bash-5.1.8
```

#### 8.33 Libtool-2.4.6

Libtool软件包包含GNU通用库支持脚本。它在一个一致、可移植的接口下隐藏了使用共享库的复杂性。

估计构建时间：1.5 SBU

需要硬盘空间：43 MB

```shell
cd $LFS/sources
tar -xvf libtool-2.4.6.tar.xz
cd libtool-2.4.6

#准备编译
./configure --prefix=/usr

make
make check TESTSUITEFLAGS=-j4
make install

rm -fv /usr/lib/libltdl.a
cd ..
rm -rf libtool-2.4.6
```

#### 8.34 GDBM-1.20

GDBM软件包包含GNU数据库管理器。它是一个使用可扩展散列的数据库函数库，工作方法和标准UNIX dbm类似。该库提供用于存储键值对、通过键搜索和获取数据，以及删除键和对应数据的原语。

估计构建时间：0.1 SBU

需要硬盘空间：11 MB

```shell
cd $LFS/sources
tar -xvf gdbm-1.20.tar.gz
cd gbdm-1.20

#准备编译
./configure --prefix=/usr    \
            --disable-static \
            --enable-libgdbm-compat
            
make
make -k check
make install

cd ..
rm -rf gbdm-1.20
```

#### 8.35 Gperf-3.1

Gperf根据一组键值，生成完美散列函数。

估计构建时间：< 0.1 SBU

需要硬盘空间：6.0 MB

```shell
cd $LFS/sources
tar -xvf gperf-3.1.tar.gz
cd gperf-3.1

#准备编译
./configure --prefix=/usr --docdir=/usr/share/doc/gperf-3.1

make
make -j1 check
make install

cd ..
rm -rf gperf-3.1
```

#### 8.36 Expat-2.4.1

Expat软件包包含用于解析XML文件的面向流的C语言库。

估计构建时间：0.1 SBU

需要硬盘空间：13 MB

```shell
cd $LFS/sources
tar -xvf expat-2.4.1.tar.xz
cd expat-2.4.1

#准备编译
./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/expat-2.4.1
            
make
make check
make install

install -v -m644 doc/*.{html,png,css} /usr/share/doc/expat-2.4.1

cd ..
rm -rf expat-2.4.1
```

#### 8.37 Inetutils-2.1

Inetutils软件包包含基本网络程序。

估计构建时间：0.3 SBU

需要硬盘空间：30 MB

```shell
cd $LFS/sources
tar -xvf inetutils-2.1.tar.xz
cd inetutils-2.1

#准备编译
./configure --prefix=/usr        \
            --bindir=/usr/bin    \
            --localstatedir=/var \
            --disable-logger     \
            --disable-whois      \
            --disable-rcp        \
            --disable-rexec      \
            --disable-rlogin     \
            --disable-rsh        \
            --disable-servers
            
make
make check
make install
mv -v /usr/{,s}bin/ifconfig

cd ..
rm -rf inetutils-2.1
```

#### 8.38 Less-590

Less软件包包含一个文本文件查看器。

估计构建时间：< 0.1 SBU

需要硬盘空间：4.2 MB

```shell
cd $LFS/sources
tar -xvf less-590.tar.gz
cd less-590

#准备编译
./configure --prefix=/usr --sysconfdir=/etc

make
make install

cd ..
rm -rf less-590
```

#### 8.39 Perl-5.34.0

Perl软件包包含实用报表提取语言。

估计构建时间：10 SBU

需要硬盘空间：226 MB

```shell
cd $LFS/sources
tar -xvf perl-5.34.0.tar.xz
cd perl-5.34.0

#应用补丁
patch -Np1 -i ../perl-5.34.0-upstream_fixes-1.patch

export BUILD_ZLIB=False
export BUILD_BZIP2=0

sh Configure -des                                         \
             -Dprefix=/usr                                \
             -Dvendorprefix=/usr                          \
             -Dprivlib=/usr/lib/perl5/5.34/core_perl      \
             -Darchlib=/usr/lib/perl5/5.34/core_perl      \
             -Dsitelib=/usr/lib/perl5/5.34/site_perl      \
             -Dsitearch=/usr/lib/perl5/5.34/site_perl     \
             -Dvendorlib=/usr/lib/perl5/5.34/vendor_perl  \
             -Dvendorarch=/usr/lib/perl5/5.34/vendor_perl \
             -Dman1dir=/usr/share/man/man1                \
             -Dman3dir=/usr/share/man/man3                \
             -Dpager="/usr/bin/less -isR"                 \
             -Duseshrplib                                 \
             -Dusethreads
             
make
make test
make install
unset BUILD_ZLIB BUILD_BZIP2

cd ..
rm -rf perl-5.34.0
```

#### 8.40 XML::Parser-2.46

XML::Parser模块是James Clark的XML解析器Expat的Perl接口。

估计构建时间：< 0.1 SBU

需要硬盘空间：2.4 MB

```shell
cd $LFS/sources
tar -xvf XML-Parser-2.46.tar.gz
cd XML-Parser-2.46

#准备编译
perl Makefile.PL

make
make test
make install

cd ..
rm -rf XML-Parser-2.46
```

#### 8.41 Intltool-0.51.0

Intltool是一个从源代码文件中提取可翻译字符串的国际化工具。

估计构建时间：< 0.1 SBU

需要硬盘空间：1.5 MB

```shell
cd $LFS/sources
tar -xvf intltool-0.51.0.tar.gz
cd intltool-0.51.0

#修复警告
sed -i 's:\\\${:\\\$\\{:' intltool-update.in

#准备编译
./configure --prefix=/usr

make
make check
make install
install -v -Dm644 doc/I18N-HOWTO /usr/share/doc/intltool-0.51.0/I18N-HOWTO

cd ..
rm -rf intltool-0.51.0
```

#### 8.42 Autoconf-2.71

Autoconf软件包包含生成能自动配置软件包的shell脚本的程序。

估计构建时间：< 0.1 SBU

需要硬盘空间：24 MB

```shell
cd $LFS/sources
tar -xvf autoconf-2.71.tar.xz
cd autoconf-2.71

#准备编译
./configure --prefix=/usr

make
make check
make install

cd ..
rm -rf autoconf-2.71
```

#### 8.43 Automake-1.16.4

Automake软件包包含自动生成Makefile，以便和Autoconf一同使用的程序。

估计构建时间：< 0.1 SBU

需要硬盘空间：115 MB

```shell
cd $LFS/sources
tar -xvf automake-1.16.4.tar.xz
cd automake-1.16.4

#准备编译
./configure --prefix=/usr --docdir=/usr/share/doc/automake-1.16.4

make
make -j4 check
make install

cd ..
rm -rf automake-1.16.4
```

#### 8.44 Kmod-29

Kmod软件包包含用于加载内核模块的库和工具。

估计构建时间：0.1 SBU

需要硬盘空间：12 MB

```shell
cd $LFS/sources
tar -xvf kmod-29.tar.xz
cd kmod-29

#准备编译
./configure --prefix=/usr          \
            --sysconfdir=/etc      \
            --with-xz              \
            --with-zstd            \
            --with-zlib
            
make

make install

for target in depmod insmod modinfo modprobe rmmod; do
  ln -sfv ../bin/kmod /usr/sbin/$target
done

ln -sfv kmod /usr/bin/lsmod

cd ..
rm -rf kmod-29
```

#### 8.45 Elfutils-0.185 中的 Libelf

Libelf是一个处理ELF(可执行和可链接格式) 文件的库。

估计构建时间：0.9 SBU

需要硬盘空间：115 MB

```shell
cd $LFS/sources
tar -xvf elfutils-0.185.tar.bz2
cd elfutils-0.185

#准备编译
./configure --prefix=/usr                \
            --disable-debuginfod         \
            --enable-libdebuginfod=dummy
            
make
make check

make -C libelf install
install -vm644 config/libelf.pc /usr/lib/pkgconfig
rm /usr/lib/libelf.a

cd ..
rm -rf elfutils-0.185
```

#### 8.46 Libffi-3.4.2

Libffi库提供一个可移植的高级编程接口，用于处理不同调用惯例。这允许程序在运行时调用任何给定了调用接口的函数。

估计构建时间：2.0 SBU

需要硬盘空间：10 MB

```shell
cd $LFS/sources
tar -xvf libffi-3.4.2.tar.gz
cd libffi-3.4.2

#准备编译
./configure --prefix=/usr          \
            --disable-static       \
            --with-gcc-arch=native \
            --disable-exec-static-tramp
            
make
make check
make install

cd ..
rm -rf libffi-3.4.2
```

#### 8.47 OpenSSL-1.1.1l

OpenSSL软件包包含密码学相关的管理工具和库。它们被用于向其他软件包提供密码学功能，例如OpenSSH，电子邮件程序和Web浏览器 (以访问HTTPS站点)。

估计构建时间：2.2 SBU

需要硬盘空间：154 MB

```shell
cd $LFS/sources
tar -xvf openssl-1.1.1l.tar.gz
cd openssl-1.1.1l

#准备编译
./config --prefix=/usr         \
         --openssldir=/etc/ssl \
         --libdir=lib          \
         shared                \
         zlib-dynamic
         
make
make test
sed -i '/INSTALL_LIBS/s/libcrypto.a libssl.a//' Makefile
make MANSUFFIX=ssl install

#将版本号添加到文档目录名
mv -v /usr/share/doc/openssl /usr/share/doc/openssl-1.1.1l

#安装额外文档
cp -vfr doc/* /usr/share/doc/openssl-1.1.1l

cd ..
rm -rf openssl-1.1.1l
```

#### 8.48 Python-3.9.6

Python 3软件包包含Python开发环境。它被用于面向对象编程，编写脚本，为大型程序建立原型，或者开发完整的应用。

估计构建时间：4.4 SBU

需要硬盘空间：260 MB

```shell
cd $LFS/sources
tar -xvf Python-3.9.6.tar.xz
cd  Python-3.9.6

#准备编译
./configure --prefix=/usr       \
            --enable-shared     \
            --with-system-expat \
            --with-system-ffi   \
            --with-ensurepip=yes \
            --enable-optimizations
            
make
make install

install -v -dm755 /usr/share/doc/python-3.9.6/html 

tar --strip-components=1  \
    --no-same-owner       \
    --no-same-permissions \
    -C /usr/share/doc/python-3.9.6/html \
    -xvf ../python-3.9.6-docs-html.tar.bz2
    
cd ..
rm -rf Python-3.9.6
```

#### 8.49 Ninja-1.10.2

Ninja是一个注重速度的小型构建系统。

估计构建时间：0.2 SBU

需要硬盘空间：64 MB

```shell
cd $LFS/sources
tar -xvf ninja-1.10.2.tar.gz
cd ninja-1.10.2

export NINJAJOBS=4
sed -i '/int Guess/a \
  int   j = 0;\
  char* jobs = getenv( "NINJAJOBS" );\
  if ( jobs != NULL ) j = atoi( jobs );\
  if ( j > 0 ) return j;\
' src/ninja.cc

#构建Ninja
python3 configure.py --bootstrap

#测试编译结果
./ninja ninja_test
./ninja_test --gtest_filter=-SubprocessTest.SetWithLots

#安装该软件包
install -vm755 ninja /usr/bin/
install -vDm644 misc/bash-completion /usr/share/bash-completion/completions/ninja
install -vDm644 misc/zsh-completion  /usr/share/zsh/site-functions/_ninja

cd ..
rm -rf ninja-1.10.2
```

#### 8.50 Meson-0.59.1

Meson是一个开放源代码构建系统，它的设计保证了非常快的执行速度，和尽可能高的用户友好性。

估计构建时间：< 0.1 SBU

需要硬盘空间：40 MB

```shell
cd $LFS/sources
tar -xvf meson-0.59.1.tar.gz
cd meson-0.59.1

#编译
python3 setup.py build

python3 setup.py install --root=dest
cp -rv dest/* /
install -vDm644 data/shell-completions/bash/meson /usr/share/bash-completion/completions/meson
install -vDm644 data/shell-completions/zsh/_meson /usr/share/zsh/site-functions/_meson

cd ..
rm -rf meson-0.59.1
```

#### 8.51 Coreutils-8.32

Coreutils软件包包含用于显示和设定系统基本属性的工具。

估计构建时间：2.6 SBU

需要硬盘空间：153 MB

```shell
cd $LFS/sources
tar -xvf coreutils-8.32.tar.xz
cd coreutils-8.32

#应用补丁
patch -Np1 -i ../coreutils-8.32-i18n-1.patch

#准备编译
autoreconf -fiv
FORCE_UNSAFE_CONFIGURE=1 ./configure \
            --prefix=/usr            \
            --enable-no-install-program=kill,uptime
            
make

#为root用户进行测试
make NON_ROOT_USERNAME=tester check-root

echo "dummy:x:102:tester" >> /etc/group

chown -Rv tester . 
su tester -c "PATH=$PATH make RUN_EXPENSIVE_TESTS=yes check"

#删除临时组
sed -i '/dummy/d' /etc/group
make install

#将程序移动到FHS要求的位置
mv -v /usr/bin/chroot /usr/sbin
mv -v /usr/share/man/man1/chroot.1 /usr/share/man/man8/chroot.8
sed -i 's/"1"/"8"/' /usr/share/man/man8/chroot.8

cd ..
rm -rf coreutils-8.32
```

#### 8.52 Check-0.15.2

Check是一个C语言单元测试框架。

估计构建时间：0.1 SBU

需要硬盘空间：12 MB

```shell
cd $FLS/sources
tar -xvf check-0.15.2.tar.gz
cd check-0.15.2

#准备编译
./configure --prefix=/usr --disable-static

make
make check
make docdir=/usr/share/doc/check-0.15.2 install

cd ..
rm -rf check-0.15.2
```

#### 8.53 Diffutils-3.8

Diffutils软件包包含显示文件或目录之间差异的程序。

估计构建时间：0.7 SBU

需要硬盘空间：36 MB

```shell
cd $LFS/sources
tar -xvf diffutils-3.8.tar.xz
cd diffutils-3.8

#准备编译
./configure --prefix=/usr

make
make check
make install

cd ..
rm -rf diffutils-3.8
```

#### 8.54 Gawk-5.1.0

Gawk软件包包含操作文本文件的程序。

估计构建时间：0.4 SBU

需要硬盘空间：42 MB

```shell
cd $LFS/sources
tar -xvf gawk-5.1.0.tar.xz
cd gawk-5.1.0

#确保不安装某些不需要的文件
sed -i 's/extras//' Makefile.in

#准备编译
./configure --prefix=/usr

make
make check
make intsall

#安装文档
mkdir -v /usr/share/doc/gawk-5.1.0
cp    -v doc/{awkforai.txt,*.{eps,pdf,jpg}} /usr/share/doc/gawk-5.1.0

cd ..
rm -rf gawk-5.1.0
```

#### 8.55 Findutils-4.8.0

Findutils软件包包含用于查找文件的程序。这些程序能够递归地搜索目录树，以及创建、维护和搜索文件数据库 (一般比递归搜索快，但在数据库最近没有更新时不可靠)。

估计构建时间：0.9 SBU

需要硬盘空间：52 MB

```shell
cd $LFS/sources
tar -xvf findutils-4.8.0.tar.xz
cd findutils-4.8.0

#准备编译
./configure --prefix=/usr --localstatedir=/var/lib/locate

make
chown -Rv tester .
su tester -c "PATH=$PATH make check"

make install
cd ..
rm -rf findutils-4.8.0
```

#### 8.56 Groff-1.22.4

Groff软件包包含处理和格式化文本的程序。

估计构建时间：0.5 SBU

需要硬盘空间：88 MB

```shell
cd $LFS/sources
tar -xvf groff-1.22.4.tar.gz
cd groff-1.22.4

#准备编译
PAGE=A4 ./configure --prefix=/usr

make -j1
make install

cd ..
rm -rf groff-1.22.4
```

#### 8.57 GRUB-2.06

GRUB软件包包含 “大统一” (GRand Unified)启动引导器。

估计构建时间：0.8 SBU

需要硬盘空间：158 MB

```shell
cd $LFS/sources
tar -xvf grub-2.06.tar.xz
cd grub-2.06

#准备编译
./configure --prefix=/usr          \
            --sysconfdir=/etc      \
            --disable-efiemu       \
            --disable-werror
            
make
make install
mv -v /etc/bash_completion.d/grub /usr/share/bash-completion/completions

cd ..
rm -rf grub-2.06
```

#### 8.58 Gzip-1.10

Gzip软件包包含压缩和解压缩文件的程序。

估计构建时间：0.1 SBU

需要硬盘空间：19 MB

```shell
cd $LFS/sources
tar -xvf gzip-1.10.tar.xz
cd gzip-1.10

#准备编译
./configure --prefix=/usr

make
make check
make install

cd ..
rm -rf gzip-1.10
```

#### 8.59 IPRoute2-5.13.0

IPRoute2软件包包含基于IPv4的基本和高级网络程序。

估计构建时间：0.2 SBU

需要硬盘空间：15 MB

```shell
cd $LFS/sources
tar -xvf iproute2-5.13.0.tar.xz
cd iproute2-5.13.0

sed -i /ARPD/d Makefile
rm -fv man/man8/arpd.8

#禁用模块
sed -i 's/.m_ipt.o//' tc/Makefile

make
make SBINDIR=/usr/sbin install
mkdir -v              /usr/share/doc/iproute2-5.13.0
cp -v COPYING README* /usr/share/doc/iproute2-5.13.0

cd ..
rm -rf iproute2-5.13.0
```

#### 8.60 Kbd-2.4.0

Kbd软件包包含按键表文件、控制台字体和键盘工具。

估计构建时间：0.2 SBU

需要硬盘空间：33 MB

```shell
cd $LFS/sources
tar -xvf kbd-2.4.0.tar.xz
cd kbd-2.4.0

#补丁修复
patch -Np1 -i ../kbd-2.4.0-backspace-1.patch

sed -i '/RESIZECONS_PROGS=/s/yes/no/' configure
sed -i 's/resizecons.8 //' docs/man/man8/Makefile.in

./configure --prefix=/usr --disable-vlock

make
make check
make install

mkdir -v            /usr/share/doc/kbd-2.4.0
cp -R -v docs/doc/* /usr/share/doc/kbd-2.4.0

cd ..
rm -rf kbd-2.4.0
```

#### 8.61 Libpipeline-1.5.3

Libpipeline软件包包含用于灵活、方便地处理子进程流水线的库。

估计构建时间：0.1 SBU

需要硬盘空间：9.1 MB

```shell
cd $LFS/sources
tar -xvf libpipeline-1.5.3.tar.gz
cd libpipeline-1.5.3

#准备编译
./configure --prefix=/usr

make
make check
make install

cd ..
rm -rf libpipeline-1.5.3
```

#### 8.62 Make-4.3

Make软件包包含一个程序，用于控制从软件包源代码生成可执行文件和其他非源代码文件的过程。

估计构建时间：0.6 SBU

需要硬盘空间：13 MB

```shell
cd $LFS/sources
tar -xvf make-4.3.tar.gz
cd make-4.3

#准备编译
./configure --prefix=/usr

make
make check
make install

cd ..
rm -rf make-4.3
```

#### 8.63 Patch-2.7.6

Patch软件包包含通过应用 “补丁” 文件，修改或创建文件的程序，补丁文件通常是diff程序创建的。

估计构建时间：0.2 SBU

需要硬盘空间：12 MB

```shell
cd $LFS/sources
tar -xvf patch-2.7.6.xz
cd path-2.7.6

#准备编译
./configure --prefix=/usr

make
make check
make install

cd ..
rm -rf path-2.7.6
```

#### 8.64 Tar-1.34

Tar软件包提供创建tar归档文件，以及对归档文件进行其他操作的功能。Tar可以对已经创建的归档文件进行提取文件，存储新文件，更新文件，或者列出文件等操作。

估计构建时间：1.9 SBU

需要硬盘空间：40 MB

```shell
cd $LFS/sources
tar -xvf tar-1.34.tar.xz
cd tar-1.34

#准备编译
FORCE_UNSAFE_CONFIGURE=1  \
./configure --prefix=/usr

make
make check
make install

make -C doc install-html docdir=/usr/share/doc/tar-1.34

cd ..
rm -rf tar-1.34
```

#### 8.65 Texinfo-6.8

Texinfo软件包包含阅读、编写和转换info页面的程序。

估计构建时间：0.6 SBU

需要硬盘空间：112 MB

```shell
cd $LFS/sources
tar -xvf texinfo-6.8.tar.xz
cd texinfo-6.8

#准备编译
./configure --prefix=/usr

#修复已知问题
sed -e 's/__attribute_nonnull__/__nonnull/' \
    -i gnulib/lib/malloc/dynarray-skeleton.c
    
make
make check
make install
make TEXMF=/usr/share/texmf install-tex

cd ..
rm -rf texinfo-6.8
```

#### 8.66 Vim-8.2.3337

Vim软件包包含强大的文本编辑器。

>如果您喜爱其他编辑器 —— 例如 Emacs、Joe、或者 Nano —— 参考 https://www.linuxfromscratch.org/blfs/view/11.0/postlfs/editors.html 中建议的安装说明。

估计构建时间：2.3 SBU

需要硬盘空间：199 MB

```shell
cd $LFS/sources
tar -xvf vim-8.2.3337.tar.gz
cd vim-8.2.3337

#修改默认位置为/etc
echo '#define SYS_VIMRC_FILE "/etc/vimrc"' >> src/feature.h

#准备编译
./configure --prefix=/usr

make
chown -Rv tester .
su tester -c "LANG=en_US.UTF-8 make -j1 test" &> vim-test.log

make install

#适配vi
ln -sv vim /usr/bin/vi
for L in  /usr/share/man/{,*/}man1/vim.1; do
    ln -sv vim.1 $(dirname $L)/vi.1
done

#创建符号链接
ln -sv ../vim/vim82/doc /usr/share/doc/vim-8.2.3337

cd ..
rm -rf vim-8.2.3337
```

#### 8.67 MarkupSafe-2.0.1

MarkupSafe是一个为XML/HTML/XHTML标记语言实现字符串安全处理的Python模块。

估计构建时间：< 0.1 SBU

需要硬盘空间：516 KB 

```shell
cd $LFS/sources
tar -xvf MarkupSafe-2.0.1.tar.gz 
cd MarkupSafe-2.0.1

python3 setup.py build
python3 setup.py install --optimize=1

cd ..
rm -rf MarkupSafe-2.0.1
```

#### 8.68 Jinja2-3.0.1

Jinja2是一个实现了简单的，Python风格的模板语言的Python模块。

估计构建时间：< 0.1 SBU

需要硬盘空间：3.7 MB

```shell
cd $LFS/sources
tar -xvf tar -xvf Jinja2-3.0.1.tar.gz
cd Jinja2-3.0.1

python3 setup.py install --optimize=1

cd ..
rm -rf Jinja2-3.0.1
```

#### 8.69 Systemd-249

Systemd软件包包含控制系统引导、运行和关闭的程序。

估计构建时间：2.7 SBU

需要硬盘空间：277 MB

```shell
cd $LFS/sources
tar -xvf systemd-249.tar.gz
cd systemd-249

#应用补丁修复
patch -Np1 -i ../systemd-249-upstream_fixes-1.patch

sed -i -e 's/GROUP="render"/GROUP="video"/' \
        -e 's/GROUP="sgx", //' rules.d/50-udev-default.rules.in
        
mkdir -p build
cd build

LANG=en_US.UTF-8                    \
meson --prefix=/usr                 \
      --sysconfdir=/etc             \
      --localstatedir=/var          \
      --buildtype=release           \
      -Dblkid=true                  \
      -Ddefault-dnssec=no           \
      -Dfirstboot=false             \
      -Dinstall-tests=false         \
      -Dldconfig=false              \
      -Dsysusers=false              \
      -Db_lto=false                 \
      -Drpmmacrosdir=no             \
      -Dhomed=false                 \
      -Duserdb=false                \
      -Dman=false                   \
      -Dmode=release                \
      -Ddocdir=/usr/share/doc/systemd-249 \
      ..
      
LANG=en_US.UTF-8 ninja
LANG=en_US.UTF-8 ninja install
tar -xf ../../systemd-man-pages-249.tar.xz --strip-components=1 -C /usr/share/man
rm -rf /usr/lib/pam.d
systemd-machine-id-setup
systemctl preset-all
systemctl disable systemd-time-wait-sync.service

cd ../..
rm -rf systemd-249
```

#### 8.70 D-Bus-1.12.20

D-bus是一个消息总线系统，即应用程序之间互相通信的一种简单方式。D-Bus提供一个系统守护进程 (负责 “添加了新硬件” 或 “打印队列发生改变” 等事件)，并对每个用户登录会话提供一个守护进程 (负责一般用户程序的进程间通信)。另外，消息总线被构建在一个通用的一对一消息传递网络上，它可以被任意两个程序用于直接通信 (不需通过消息总线守护进程)。

估计构建时间：0.2 SBU

需要硬盘空间：18 MB

```shell
cd $LFS/sources
tar -xvf dbus-1.12.20.tar.gz
cd dbus-1.12.20

#准备编译
./configure --prefix=/usr                        \
            --sysconfdir=/etc                    \
            --localstatedir=/var                 \
            --disable-static                     \
            --disable-doxygen-docs               \
            --disable-xml-docs                   \
            --docdir=/usr/share/doc/dbus-1.12.20 \
            --with-console-auth-dir=/run/console \
            --with-system-pid-file=/run/dbus/pid \
            --with-system-socket=/run/dbus/system_bus_socket
            
make
make install
ln -sfv /etc/machine-id /var/lib/dbus

cd ..
rm -rf dbus-1.12.20
```

#### 8.71 Man-DB-2.9.4

Man-DB软件包包含查找和阅读man页面的程序。

估计构建时间：0.4 SBU

需要硬盘空间：38 MB

```shell
cd $LFS/sources
tar -xvf man-db-2.9.4.tar.xz
cd man-db-2.9.4

#准备编译
./configure --prefix=/usr                        \
            --docdir=/usr/share/doc/man-db-2.9.4 \
            --sysconfdir=/etc                    \
            --disable-setuid                     \
            --enable-cache-owner=bin             \
            --with-browser=/usr/bin/lynx         \
            --with-vgrind=/usr/bin/vgrind        \
            --with-grap=/usr/bin/grap
            
make 
make check
make install

cd ..
rm -rf man-db-2.9.4
```

#### 8.72 Procps-ng-3.3.17 

Procps-ng 软件包包含监视进程的程序。

估计构建时间：0.5 SBU

需要硬盘空间：19 MB

```shell
cd $LFS/sources
tar -xvf procps-ng-3.3.17.tar.xz
cd procps-ng-3.3.17

#准备编译
./configure --prefix=/usr                            \
            --docdir=/usr/share/doc/procps-ng-3.3.17 \
            --disable-static                         \
            --disable-kill                           \
            --with-systemd
            
make
make check
make install

cd ..
rm -rf procps-ng-3.3.17
```

#### 8.73 Util-linux-2.37.2

Util-linux软件包包含若干工具程序。这些程序中有处理文件系统、终端、分区和消息的工具。

估计构建时间：1.1 SBU

需要硬盘空间：261 MB

```shell
cd $LFS/sources
tar -xvf util-linux-2.37.2.tar.xz
cd util-linux-2.37.2

./configure ADJTIME_PATH=/var/lib/hwclock/adjtime   \
            --libdir=/usr/lib    \
            --docdir=/usr/share/doc/util-linux-2.37.2 \
            --disable-chfn-chsh  \
            --disable-login      \
            --disable-nologin    \
            --disable-su         \
            --disable-setpriv    \
            --disable-runuser    \
            --disable-pylibmount \
            --disable-static     \
            --without-python     \
            runstatedir=/run
            
make
chown -Rv tester .
su tester -c "make -k check"
make install

cd ..
rm -rf util-linux-2.37.2
```

#### 8.74 E2fsprogs-1.46.4

E2fsprogs软件包包含处理*ext2*文件系统的工具。此外它也支持*ext3*和*ext4*日志文件系统。

估计构建时间：机械硬盘4.4SBU，固态硬盘上1.5SBU

需要硬盘空间：93 MB

```shell
cd $LFS/sources
tar -xvf e2fsprogs-1.46.4.tar.gz
cd e2fsprogs-1.46.4

mkdir -v build
cd build

#准备编译
../configure --prefix=/usr           \
             --sysconfdir=/etc       \
             --enable-elf-shlibs     \
             --disable-libblkid      \
             --disable-libuuid       \
             --disable-uuidd         \
             --disable-fsck
             
make
make check
make install
rm -fv /usr/lib/{libcom_err,libe2p,libext2fs,libss}.a

gunzip -v /usr/share/info/libext2fs.info.gz
install-info --dir-file=/usr/share/info/dir /usr/share/info/libext2fs.info

makeinfo -o      doc/com_err.info ../lib/et/com_err.texinfo
install -v -m644 doc/com_err.info /usr/share/info
install-info --dir-file=/usr/share/info/dir /usr/share/info/com_err.info

cd ..
rm -rf e2fsprogs-1.46.4
```

#### 8.75 移除调试符号

```shell

```

