1.ipod touch4 6.1.6越狱
https://www.feng.com/post/13072298

cydia 打开后出现无效证书的问题：
最近看到些吧友们在自己的ios6设备越狱后出现证书失效问题出下教程  
打开这个网站[网页链接](http://jump.bdimg.com/safecheck/index?url=rN3wPs8te/pL4AOY0zAwh5kCGKFf/n6y7tQJVgBrXasqAvrdSF6B6w2Wyr7d91hxgcryPyJATrQTYus57EiVrzwrpR8Nlzh8K9sAjxLE+i1THokc0MpaIxpem6Q1VRu6DbpqsOWF+Qx0ZuDoDuT6fwbpbXrCbuBh+E6cTq6MFFjRnP8y/bGnkqwuOXZoLxkv00GbCw2OLAY=)  
打开后找到这个证书下载那个后面那个选项download cer/crt  
![[Pasted image 20251212223133.png]]
下载完证书后我们把它改成mobileconfig拓展名后,连接设备进入爱思助手工具箱,找到管理描述文件把他导入到设备信任即可

对于一些不信任问题，主要是证书以及描述文件需要更新，这个时候可以去上面下载证书的地方下载新的证书文件

2.iphone5 ios10 降级8.4.1以后越狱问题
降级就按照常规方法降级就好了

<mark style="background: #FF5582A6;">关键是降级以后需要将系统重置一下，不然越狱会失败，爱思助手安装签名软件也会提示内存不足</mark>，这可能就是从ios10 降级到8的问题。可能是ios10原本的系统内存还在磁盘中，然后因为是修改系统文件欺骗苹果服务器以为是ios6并升级到ios8，即从ios10替换成了8，导致原本的系统空间并没有释放。