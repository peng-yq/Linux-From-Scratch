# Linux From Scratch 11.0-systemed

LFS──Linux from Scratch，就是一种从网上直接下载源码，从头编译Linux的安装方式。它不是发行版，只是一个菜谱，告诉你到哪里去买菜（下载源码），怎么把这些生东西( raw code) 作成符合自己口味的菜肴──个性化的Linux，不单单是个性的桌面。

[官方中文文档](https://bf.mengyan1223.wang/lfs/zh_CN/11.0-systemd/LFS-SYSD-BOOK.html)     [官方英文文档](https://www.linuxfromscratch.org/lfs/view/stable/index.html)      [FAQ](https://www.linuxfromscratch.org/faq/)      [镜像](https://www.linuxfromscratch.org/mirrors.html)     [Linux操作教程](https://www.runoob.com/linux/linux-tutorial.html)     [LFS视频教程(English-version)](https://www.youtube.com/watch?v=9TYr1mCzMcg&list=PLyc5xVO2uDsAlIkKBIGauDQ6LejoQovyL)    [LFS视频教程(English-version & Ubuntu-as-Host-Machine)](https://www.youtube.com/watch?v=5tRJgDJC7kY)

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

