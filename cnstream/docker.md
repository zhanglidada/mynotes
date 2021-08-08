hello#关于docker的相关学习
**docker可以产生基础镜像，每加一层新的内容也形成新的镜像。每个镜像都可以去在加新的内容。变化无穷，复用资源.**
1.关于docker run/start的区别：
docker run 只在第一次运行时使用，将镜像放到容器中，以后再次启动这个容器时，只需要使用命令docker start 即可。
docker run相当于执行了两步操作：将镜像放入容器中（docker create）,然后将容器启动，使之变成运行时容器（docker start）。
而docker start的作用是，重新启动已存在的镜像。也就是说，如果使用这个命令，我们必须事先知道这个容器的ID，或者这个容器的名字，我们可以使用docker ps找到这个容器的信息。

2.关于docker容器的重命名：
因为容器的ID是随机码，而容器的名字又是看似无意义的命名，我们可以使用命令方式：

	docker rename  old_name  new_name
给这个容器命名。这样以后，我们再次启动或停止容器时，就可以直接使用这个名字。

	docker [stop] [start]  new_name

3.docker容器简单操作：

4.nvidia docker的使用方式：
在19.03版本的docker之后，可以直接在docker中挂载gpu

5.docker images和container的关系：

镜像的概念更多偏向于一个环境包，这个环境包可以移动到任意的Docker平台中去运行；而容器就是你运行环境包的实例。你可以针对这个环境包运行N个实例。换句话说container是images的一种具体表现形式。
