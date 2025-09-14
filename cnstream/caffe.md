[[cnstream项目小结]]
##杂项：
1.cnstream在编译后类似于opencv,也就是提供提供一些支持库以及一些可执行的samples二进制文件
2.关于在docker中使用的问题：

	1)docker其实就是一个虚拟机，和外界环境隔离开
	2)在进入docker时会有一个映射，指定将当前目录映射到docker下的某个目录中
	3)在docker中会配置有自己的一些环境和依赖库，和外部系统除了映射的文件外完全隔离
3.HDF（Hierarchical Data Format）分级数据格式:
指一种为存储和处理大容量科学数据设计的文件格式及相应库文件。具有极高的压缩率。
4.在对数据进行训练以及测试的时候，记得归一化，这个很重要
##关于在cnn的神经网中图像的处理问题
###图像的前处理
cnstream的图像前处理文件为clas_off_post.cpp
###图像的后处理
cnstream的图像后处理头文件为data_provider.hpp，其中off_data_provider.hpp和on_data_provider.hpp均为他的子类


##caffe中的一些知识点：
###1.solver.prototxt：
solver算是caffe的核心的核心，它协调着整个模型的运作。caffe程序运行必带的一个参数就是solver配置文件。solver其实就是设置caffe训练的时候的超参数，即参数的参数。（都是作者根据训练的效果
进行的调整，例如学习率，步长，动量等）
在Deep Learning中，往往loss function是非凸的，没有解析解，我们需要通过优化方法求解。solver的主要作用就是交替调用前向（forword）算法和后向（backward）算法来更新参数，从而最小化loss，实际上就是一种
迭代的优化算法。

**Solver的流程：**
1.     设计好需要优化的对象，以及用于学习的训练网络和用于评估的测试网络。（通过调用另外一个配置文件prototxt来进行）

2.     通过forward和backward迭代的进行优化来跟新参数。

3.     定期的评价测试网络。 （可设定多少次训练后，进行一次测试）

4.     在优化过程中显示模型和solver的状态

在每一次的迭代过程中，solver做了这几步工作：

1、调用forward算法来计算最终的输出值，以及对应的loss

2、调用backward算法来计算每层的梯度

3、根据选用的slover方法，利用梯度进行参数更新

4、记录并保存每次迭代的学习率、快照，以及对应的状态。

###2.关于caffe网络模型的训练：

	LOG=./log/DRRN_B1U25_52C128_291_31.log
	CAFFE=/data2/taiying/MSU_Code/my_caffe/build/tools/caffe # your caffe path
	$CAFFE train --solver=./DRRN_B1U25_52C128_solver.prototxt -gpu 0 2>&1 | tee $LOG

在训练的时候，train.prototxt里面就已经指定了输入的训练数据; 同时，由于在训练的同时需要在指定训练完一批数据后进行测试集的测试，即在train.prorotxt中同时有test的layer以及test的数据输入

###3.matlab中关于caffe的使用：
可以用caffe.Net(model_1, weights, 'test'); 来shanghai一个caffe网络。

###4.caffe forward和backword：
caffe中最重要的两个部分就是forward和backward的过程，<font color=#546987>farward是根据输入数据正向预测输入属于哪一类；backward是根据输出的结果求得代价函数，然后根据代价函数反向求去其相对于各层网络参数的梯度的过程。</font>

在caffe中，train和test的时候都会用到forward。
