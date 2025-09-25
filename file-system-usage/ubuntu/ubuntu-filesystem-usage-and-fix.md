[[linux文件系统以及磁盘分区问题]]
##关于linux显示磁盘read-only file system问题：
其实就是磁盘超级块损坏：
###解决方案：
Ubuntu安装磁盘修复软件：用`sudo apt-get install ntfs-3g`安装ntfs-3g。然后在NTFS分区上运行ntfsfix命令。
	
	sanduo@archlinux:/home/sanduo> sudo ntfsfix /dev/sda2
修复成功后即可向磁盘进行读写操作。
###补充：
1. 对于较新的Ubuntus您可以一起使用-b和-d选项。 -b尝试修复坏群集和-d来修复脏状态。所以命令可以

	sudo ntfsfix -b -d /dev/sda6
--help显示它们

ntfsfix v2015.3.14AR.1 (libntfs-3g)

	Usage: ntfsfix [options] device
	    Attempt to fix an NTFS partition.

	    -b, --clear-bad-sectors Clear the bad sector list
	    -d, --clear-dirty       Clear the volume dirty flag
	    -h, --help              Display this help
	    -n, --no-action         Do not write anything
	    -V, --version           Display version information


2.可以使用parted命令进行查看磁盘分区信息以及挂载信息。
