# Slope_MC

本程序是基于有限元的边坡可靠度分析程序，改进自[Griffith教授](https://inside.mines.edu/~vgriffit) 的非饱和黄土边坡稳定性计算程序

本程序使用强度折减法计算稳定系数，使用Monte-Carlo法计算失效概率；

将计算作业量，分成**线程数量**的段数，将作业量平均分配给每一个线程；

**本程序使用MPI实现并行运算**

**程序文件**

```
slope_new/Subroutine.py
slope_new/slope2.f90
```

**示例数据文件**

```
exmaple/1.dat
exmaple/1_c.txt
exmaple/1_phi.txt
example/1_gamma.txt !未给出
exmaple/1_mstd.txt
exmaple/1_grid.txt
  
```

**结果文件** 未给出

```
1_fs.res
1_fail.res
```

## 前置安装教程

1.Fortran程序前置安装：[OpenCoarrays](https://github.com/sourceryinstitute/OpenCoarrays)

以Ubuntu22.04安装Open-coarrays 2.9.2为例：

```
sudo apt-get update
sudo apt-get install cmake gfortran
wget https://github.com/sourceryinstitute/OpenCoarrays/archive/refs/tags/2.9.2.tar.gz
tar xf 2.9.2.tar.gz
cd OpenCoarrays-2.9.2
makedir build
cd build
cmake ..
make
sudo make install
```

！若[OpenCoarrays2.9.2](https://github.com/sourceryinstitute/OpenCoarrays/archive/refs/tags/2.9.2.tar.gz) 下载失败可手动下载并上传到`/home/username/`再解压编译安装

2.可选安装--后台运行[Screen](https://www.gnu.org/software/screen/)

```
安装：
sudo apt-get install screen
列出全部会话：
screen -ls
新建：
screen
screen -S name
删除：
screen -S name -X quit ! 或在窗口内使用快捷键ctrl + D
放置后台：
使用快捷键ctrl+A再ctrl+D 
返回窗口：
screen -R
screen -r name 
```

3.Python程序须安装[Python3.x](https://www.python.org/downloads/) ！无须安装在服务器中，仅做数据处理工作，在本地使用即可

程序运行依赖`numpy`库，可使用pip安装：

```
pip install numpy
```

## 程序使用教程

### Python程序--本地运行

1.运行[Subroutine.py](https://github.com/liudh56/Slope/blob/main/Slope_new/Subroutine.py)

```
cd 代码所在目录
python ./Subroutine.py
```

 根据提示操作即可[使用教程](https://github.com/liudh56/Slope/tree/main/example)

2.将生成文件与`slope2.f90`放在同一文件夹内（本文件夹即要上传的文件夹）

文件包括：

```
xxx.dat
xxx_c.txt xxx_phi.txt xxx_gamma.txt #具体文件数量与设为随机的参数有关
```

[dat文件详细](https://github.com/liudh56/Slope/blob/main/example/1.dat) 

其他python程序使用文件（无需上传）：

```
xxx_mstd.txt
xxx_grid.txt
```

### Fortran程序--在服务器中运行

1.将上文中的文件夹上传至服务器并打开目录

2.编译命令：

```
caf slope2.f90 -o slope
```

3.运行：

```
cafrun -n 4 slope
！ 4代表线程数，可更改，若线程数报错改用下面的命令
```

 或者：使用下面命令适用于多主机的情况，根据主机实际情况分配线程数。

```
mpirun --host host1:40 host2:40 -n 80 filename

mpirun --host hostfile -n 80 filename !使用该命令须先创建hostfile文件
```

4.保存结果文件到本地与Subroutine.py文件放置在同一文件内

运行Subroutine.py程序并选择功能3计算失效概率，结果保存在`xxx_fail.res`文件内。

# Subroutine.py程序使用方式

## 1.输入项目名称xxx。

## 2.选择功能（输入序号）。

```
1. （首次使用）引导填写并重新生成xxx.dat和xxx_mstd.txt文件
   
2. 根据xxx_mstd.txt生成phi、c和gamma的值
   
3. 计算失效概率,并生成xxx_fail.res文件,储存失效概率
   
4. 生成xxx_grid.txt文件,储存网格点的序号(多层土的情况使用)
   
5. 根据xxx_mstd.txt重新生成某参数值

0. 退出程序(默认)
```

## 功能使用教程

1.第一次使用选择`1`填写`xxx.dat`和`xxx_mstd.txt`文件（也可直接修改示例文件来使用）

**建议先查看示例文件**

**若填写过程中出现填写错误，可继续填写，等待填写完成再从生成文件内修改错误**

**功能4，可协助填写`xxx.dat`文件，后文介绍**

2.再选择`2`根据提示进行选择（以内摩擦角和粘聚力为随机参数为例）

```
请输入模拟次数:10000
请输入参数类型(10:内摩擦角,20:粘聚力,30:重度) 10
请输入分布类型(0:定值（默认）,1:正态分布,2:对数正态分布) 1
已生成xxx_c.txt文件!
请输入参数类型(10:内摩擦角,20:粘聚力,30:重度) 20
请输入分布类型(0:定值（默认）,1:正态分布,2:对数正态分布): 1
已生成xxx_phi.txt文件!
```

3.文件上传计算

4.计算完成后，将结果文件`xxx_fs.res`放置与本程序在同一文件夹内，运行程序选择`3`，结果保存在`xxx_fail.res`文件

**功能4介绍（单层土不需要使用）**

```
！根据自己划分的每层土的单元格数（不是高度）填写即可，注意与ny1和ny2区分
请输入第1层土的厚度(竖向单元格数): 16
请输入第2层土的厚度(竖向单元格数): 5
已生成xxx_grid.txt文件!
```

xxx_grid.txt的内容复制到以下位置使用：

```
"Choose whether to set it as a random variable (the order is: phi c gamma,1 is random)"
1 1 0

"Property group assigned to each element (etype, data not needed if np_types=1)"
（--------此位置--------）

"Pseudo-static analysis: Horizontal acceleration factor (k_h)"
0.0
```
