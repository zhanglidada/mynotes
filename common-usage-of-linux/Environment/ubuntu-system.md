[[ubuntu下面的一些环境配置问题]]
##1.python环境问题：
###1.1.ubuntu的默认python环境问题：
其实在ubuntu18中默认安装了python2.7和python3.6的环境，但是默认使用的是python2的环境，所以需要进行如下设置：
首先需要确定python3的版本以及位置
```
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2.7   1

sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.6   2
```
**这里用到的是命令:**
```
sudo    update-alternatives: --install   <链接> <名称> <路径> <优先级>
```
链接：   `/usr/bin/python`
名称：   `python`
路径：   `/usr/bin/python2.7` 或者 `/usr/bin/python3.6`
优先级：1或者2（使用1和2将python3的优先级调高）

###1.2./usr/include 中的python3.6m 中的m是什么意思？和python3.6有什么区别？
```
python3.6m是python3.6的软连接，在目录中输入命令：ls -all会看到python3.6m中有python3.6m -> python3.6的标识。
```
###1.3.查找python的pip3安装的库的位置：
`python -m site`

##2.ubuntu下解压文件命令：
###2.1 .tar文件
```
tar -xvf FileName.tar           # 解包
tar -cvf FileName.tar DirName   # 将DirName和其下所有文件（夹）打包成为FileName.tar文件
```
###2.2 .gz文件
```
gunzip FileName.gz  # 解压1
gzip -d FileName.gz # 解压2
gzip FileName       # 压缩，只能压缩文件
```
###2.3 .tar.gz文件或.tgz文件
```
tar -zxvf FileName.tar.gz               # 解压
tar -zcvf FileName.tar.gz DirName       # 将DirName和其下所有文件（夹）压缩
tar -C DesDirName -zxvf FileName.tar.gz # 解压到目标路径DesDirName下
```
###2.4 .zip文件
```
unzip FileName.zip          # 解压
zip FileName.zip DirName    # 将DirName本身压缩
zip -r FileName.zip DirName # 压缩，递归处理，将指定目录下的所有文件和子目录一并压缩
```
###2.5 .rar文件
```
# mac和linux并没有自带rar，需要自行下载
rar x FileName.rar          # 解压
rar a FileName.rar DirName  # 压缩
```

**补充**：`tar`是打包，`.tar.gz`才是压缩过的文件，`.tar.gz`常见于unix系统，在ubuntu或macos可以直接解压，而`.zip`常见于windows系统。

##3.ubuntu上caffe安装事项
###3.1关于ubuntu安装boost
这里默认的安装放方式 ：`apt-get install libboost-all-dev`匹配对应的 python3.6
###3.2 配置caffe python并使用GPU：
1) 配置安装python所需要的依赖：
```
sudo apt-get install python-numpy python-scipy python-matplotlib python-sklearn python-skimage \
python-h5py python-protobuf python-leveldb python-networkx python-nose python-pandas python-gflags \
Cython ipython
```
2）可以在caffe/python文件夹下面的查看依赖安装方式：
```
cd /home/caffe/python [[这里进入caffe下的python目录]]
for req in $(cat requirements.txt); do pip3 install $req; done [[这里检查要求的依赖文件命进行安装]]
```
3）关于pip安装超时崩溃时可以设置超时等待时间：
`python -m pip --default-timeout=100 install -U pip`
或者更换国内的源，虽然国内的源有时候会有些库或者依赖找不到：
`https://blog.csdn.net/xiangxianghehe/article/details/80112149`

###3.3 caffe编译依赖
注意，caffe在编译的时候需要glog依赖，但是goole 的glog千万不要下载源码编译，因为它会产生动态依赖库和静态依赖库的混淆，直接apt下载最好！
```
[[安装]]
sudo apt-get install libgoogle-glog-dev
[[卸载]]
sudo apt-get remove libgoogle-glog-dev
```
如果是源码安装的化，一般会在以下位置有头文件和依赖库
```
/usr/local/include/glog
/usr/local/include/glog/config.h
/usr/local/include/glog/log_severity.h
/usr/local/include/glog/logging.h
/usr/local/include/glog/raw_logging.h
/usr/local/include/glog/stl_logging.h
/usr/local/include/glog/vlog_is_on.h
/usr/local/lib/libglog.a
/usr/local/lib/cmake/glog
```
##4.ubuntu修改环境变量的一些方式：
1)：用于当前终端：
在当前终端中输入：`export PATH=$PATH:<你的要加入的路径>`
不过上面的方法只适用于当前终端，一旦当前终端关闭或在另一个终端中，则无效。

2)：用于当前用户：
在用户主目录下有一个`.bashrc`隐藏文件，可以在此文件中加入`PATH`的设置：

`vim ~/.bashrc`

加入：
`export PATH=<你的要加入的路径>:$PATH`

如果要加入多个路径，只要：

`export PATH=<你要加入的路径1> : <你要加入的路径2>: ...... :$PATH`
当中每个路径要以冒号分隔,这样每次登录都会生效。

3)：用于所有用户：

`sudo vim /etc/.profile`

加入：
`export PATH=<你要加入的路径>:$PATH`就可以了。

我们在终端输入：`echo $PATH `可以查看环境变量

<font color=red>**注意**:修改环境变量后，除了第一种方法立即生效外，第二第三种方法要立即生效，可以`source ~/.bashrc`/`source ~/.profile`或者注销再次登录后就可以了！</font>

9.ubuntu查看一些软件或者安装包的版本：
pkg-config是一个linux下的命令，用于获得某一个库/模块的所有编译相关的信息

例如：
`pkg-config --modversion opencv`即查看安装的ku/模块的版本
