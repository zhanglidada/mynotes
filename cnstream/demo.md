###1.下载源代码：
进入自己的根目录，`git clone https://github.com/Cambricon/CNStream.git`
###2.切换目录：
`cd /cnstream/samples/detection-demo`
###3.增加pre-proc-SR.cpp文件：(用于图像的前处理)
(增加的文件放在在/cnstream/samples/detection-demo/preprocess目录下)
**附：代码在最后提供**
1). `pre-proc-SR.cpp`中定义自己的类PreprocSR继承父类preproc：
`class PreprocSR : public Preproc, virtual public libstream::ReflexObjectEx<Preproc> `

2). `pre-proc-SR.cpp`中实现继承自父类的虚函数Execute：
`int Execute(const std::vector<float*>& net_inputs, const std::shared_ptr<libstream::ModelLoader>& model,
    const std::shared_ptr<CNFrameInfo>& package)`

3). `pre-proc-SR.cpp`中声明并实现子类PreprocSR关于父类Preproc的反射机制：
```
......
//preprocSR类中的反射实现
private:
  DECLARE_REFLEX_OBJECT_EX(PreprocSR, Preproc);
};
  IMPLEMENT_REFLEX_OBJECT_EX(PreprocSR, Preproc);
}
```
###4.增加post-proc-SR.cpp文件：(用于图像的后处理)
(增加的文件放在/cnstream/samples/detection-demo/postprocess目录下)
**附：代码在最后提供**
1). `post-proc-SR.cpp`中定义自己的类 PostprocSR继承父类postproc：
`class PostprocSR : public Postproc, virtual public libstream::ReflexObjectEx<Postproc> `

2). `post-proc-SR.cpp`中实现继承自父类的虚函数Execute：
`int Execute(const std::vector<float*>& net_outputs, const std::shared_ptr<libstream::ModelLoader>& model,
    const std::shared_ptr<CNFrameInfo>& package)`

3). `post-proc-SR.cpp`中声明并实现子类PostprocSR关于父类Postproc的反射机制：
```
......
//postprocSR类中的反射实现
private:
    DECLARE_REFLEX_OBJECT_EX(PostprocSR, Postproc);
};
    IMPLEMENT_REFLEX_OBJECT_EX(PostprocSR, Postproc);
}
```
###5编译并使用离线模型
**注：后文会提供编译好的离线模型供使用。**
1). 首先切换出当前cnstream目录，进入根目录，并`git clone https://github.com/Cambricon/caffe.git`下载Cambricon caffe下来后进行编译。
(具体编译可以参考Cambricon caffe项目：`https://github.com/Cambricon/caffe`)

2). 根据我们提供的超分demo中的prototxt和训练好的caffemodel生成离线模型offline.cambricon(切换到刚刚下载的caffe项目下执行)
`你的caffe可行性文件位置/caffe genoff -model deploy.prototxt  -weights weights.caffemodel -mcore MLU100`
注：deploy.prototxt 和 weights.caffemodel可以替换为你自己的prototxt文件和caffemodel文件，并且默认生成的为`offline.cambricon`离线模型

3). 将生成的离线模型放到cnstream的目录下：
在cnstream/samples/data/models/MLU100目录下新建一个名为SR的文件夹，将offline.cambricon文件放进去。

4).运行前需要的一些处理： 
a.图像存储路径：
`/cnstream/samples/data/images`

b. 切换进入cnstream/samples/detection-demo目录下

c.创建执行需要的配置文件`SR.json`：
```
{
    "source" : {
      "class_name" : "cnstream::DataSource",
      "parallelism" : 0,
      "next_modules" : ["detector"],
      "custom_params" : {
        "source_type" : "ffmpeg",
        "output_type" : "mlu",
        "decoder_type" : "mlu",
        "reuse_cndec_buf" : "true",
        "device_id" : 0
      }
    },
    "detector" : {
      "class_name" : "cnstream::Inferencer",
      "parallelism" : 4,
      "max_input_queue_size" : 20,
      "next_modules" : [],
      "custom_params" : {
        "model_path" : "../data/models/MLU100/SR/offline.cambricon",
        "func_name" : "subnet0",
        "preproc_name" : "PreprocSR",
        "postproc_name" : "PostprocSR",
        "device_id" : 0
      }
    }
  }
```

d.创建并运行脚本`runSR.sh`: 
```
#!/bin/bash
source env.sh
mkdir -p output
#gdb --args  ./../bin/detection  \
./../bin/detection  \
    --data_path ./files.list_image \
    --src_frame_rate 27   \
    --wait_time 0 \
    --rtsp=false \
    --input_image=true \
    --loop=false \
    --config_fname "SR.json" \
    --alsologtostderr

```
e. 执行`sh runSR.sh`命令即可开始运行超分demo。
###5.简单的效果对比：
原始图像：
![](/home/zhangli/Documents/Markdown/cnstream/picture/)

利用opencv插值后的图像：
![](/home/zhangli/Documents/Markdown/cnstream/picture/)

利用超分网络实现的图像：
![](/home/zhangli/Documents/Markdown/cnstream/picture/)

另外两组案例对比：

###6.具体代码实现以及附录文件：
a. `pre-proc-SR.cpp`的实现：
```
#include "preproc.hpp"
#include <opencv2/opencv.hpp>
#include "cnbase/cnshape.h"
#include "cninfer/model_loader.h"
#include <vector>

//先对图像进行插值处理，然后利用神经网络调整其插值的精细度
namespace cnstream{
class PreprocSR : public Preproc, virtual public libstream::ReflexObjectEx<Preproc> {
public:
  int Execute(const std::vector<float*>& net_inputs, const std::shared_ptr<libstream::ModelLoader>& model,
    const std::shared_ptr<CNFrameInfo>& package){
      auto input_shapes = model->input_shapes();
      if (net_inputs.size() != 1 || input_shapes[0].c() != 1) {
        LOG(ERROR) << "[PreprocCpu] model input shape not supported";
        return -1;
      }
      int width = package->frame.width;
      int height = package->frame.height;
      int dst_w = input_shapes[0].w();
      int dst_h = input_shapes[0].h();
      uint8_t* img_data = new uint8_t[package->frame.GetBytes()];
      uint8_t* t = img_data;
      for (int i = 0; i < package->frame.GetPlanes(); ++i) {
        memcpy(t, package->frame.data[i]->GetCpuData(), package->frame.GetPlaneBytes(i));
        t += package->frame.GetPlaneBytes(i);
      }
      // convert color space
      cv::Mat img;
      cv::Mat img_ycrcb(img.rows,img.cols,CV_8UC3);
      switch (package->frame.fmt) {
        //在前处理获得数据时，从解码器上获得的数据类型
        case cnstream::CNDataFormat::CN_PIXEL_FORMAT_BGR24:
          img = cv::Mat(height, width, CV_8UC3, img_data);
          cv::cvtColor(img, img_ycrcb,cv::COLOR_BGR2YCrCb);
          break;
        case cnstream::CNDataFormat::CN_PIXEL_FORMAT_RGB24:
          img = cv::Mat(height, width, CV_8UC3, img_data);
          cv::cvtColor(img, img_ycrcb,cv::COLOR_RGB2YCrCb);
          break;
        //因为yuv图像特殊格式的问题，y：u：v为4：1：1，即表示一个像素点所需平均bite为rgb的1/2
        case cnstream::CNDataFormat::CN_PIXEL_FORMAT_YUV420_NV12: {
          img = cv::Mat(height * 3 / 2, width, CV_8UC1, img_data);//因为opencv不支持NV12格式的数据，所以采用特殊方式进行存储
          cv::Mat bgr(height, width, CV_8UC3);
          cv::cvtColor(img, bgr, cv::COLOR_YUV2BGR_NV12);
          cv::cvtColor(bgr, img_ycrcb, cv::COLOR_BGR2YCrCb);
        } break;
        case cnstream::CNDataFormat::CN_PIXEL_FORMAT_YUV420_NV21: {//因为是四个y共享一个uv数据
          img = cv::Mat(height * 3 / 2, width, CV_8UC1, img_data);
          cv::Mat bgr(height, width, CV_8UC3);
          cv::cvtColor(img, bgr, cv::COLOR_YUV2BGR_NV21);
          cv::cvtColor(bgr, img_ycrcb, cv::COLOR_BGR2YCrCb);
        } break;
        default:
          LOG(WARNING) << "[Encoder] Unsupport pixel format.";
          delete[] img_data;
          return -1;
      }
    cv::Mat img_h(img_ycrcb.rows, img_ycrcb.cols, CV_8UC1);
    std::vector<cv::Mat> channels;
    cv::split(img_ycrcb,channels);
    //获取图像的h维信息
    img_h = channels[0];
    if (height != dst_h || width != dst_w) {
      cv::Mat dst(dst_h, dst_w, CV_8UC1);
      cv::resize(img_h, dst, cv::Size(dst_w, dst_h));
      img_h = dst;
    }
    //在输入进网络钱对矩阵进行转置
    cv::Mat img_h_temp(dst_h, dst_w, CV_32FC1);
    for(int r=0;r<dst_h;r++){
      for(int c=0;c<dst_w;c++){
        //由于prototxt中没有归一化层，所以这里需要将其进行归一化操作
        img_h_temp.at<float>(r,c)=static_cast<float>(img_h.at<uint8_t>(r,c))/255;
      }
    }
    // since model input data type is float, convert image to float
    cv::Mat dst(dst_h, dst_w, CV_32FC1, net_inputs[0]);
    img_h_temp.convertTo(dst, CV_32F);
    delete[] img_data;
    return 0; 
    }//class PreprocSRCpu
private:
  DECLARE_REFLEX_OBJECT_EX(PreprocSR, Preproc);
};
  IMPLEMENT_REFLEX_OBJECT_EX(PreprocSR, Preproc);
}
```
b. `post-proc-SR.cpp`的实现：
```
#include <cmath>
#include "postproc.hpp"
#include <opencv2/opencv.hpp>
#include "cnbase/cnshape.h"
#include "cninfer/model_loader.h"
#include<vector>

namespace cnstream{

class PostprocSR : public Postproc, virtual public libstream::ReflexObjectEx<Postproc> {
public:
    int Execute(const std::vector<float*>& net_outputs, const std::shared_ptr<libstream::ModelLoader>& model,
    const std::shared_ptr<CNFrameInfo>& package) {
        int width = package->frame.width;
        int height = package->frame.height;
        auto output_shapes = model->output_shapes();
        int dst_w = output_shapes[0].w();
        int dst_h = output_shapes[0].h();
        std::cout<<"网络输出指定大小：宽： "<<dst_w<<" 高： "<<dst_h<<std::endl;

        cv::Mat img_h_temp(dst_h,dst_w,CV_32FC1,net_outputs[0]);
        cv::Mat img_h(dst_h,dst_w,CV_8UC1);
        for(int r=0;r<dst_h;r++){
            for(int c=0;c<dst_w;c++){
                img_h.at<uint8_t>(r,c)=static_cast<int>(img_h_temp.at<float>(r,c)*255);
                if(img_h.at<uint8_t>(r,c)>=255)
                    img_h.at<uint8_t>(r,c)=255;
                if(img_h.at<uint8_t>(r,c)<=0)
                    img_h.at<uint8_t>(r,c)=0;
                
            }
        }
        cv::imwrite("img_h_SR.png",img_h);
        std::cout<<"查看是否归一化2： "<<(img_h.at<uint8_t>(12,24))<<std::endl;

        uint8_t* img_data = new uint8_t[package->frame.GetBytes()];
        uint8_t* t = img_data;
        std::cout<<"package->frame.GetPlanes is : "<<package->frame.GetPlanes()<<std::endl;
        for (int i = 0; i < package->frame.GetPlanes(); ++i) {
            memcpy(t, package->frame.data[i]->GetCpuData(), package->frame.GetPlaneBytes(i));
            t += package->frame.GetPlaneBytes(i);
        }
        // convert color space
        cv::Mat img;
        cv::Mat img_ycrcb(width,height,CV_8UC3);
        switch (package->frame.fmt) {
        case cnstream::CNDataFormat::CN_PIXEL_FORMAT_BGR24:
            img = cv::Mat(height, width, CV_8UC3, img_data);
            cv::cvtColor(img, img_ycrcb,cv::COLOR_BGR2YCrCb);
            break;
        case cnstream::CNDataFormat::CN_PIXEL_FORMAT_RGB24:
            img = cv::Mat(height, width, CV_8UC3, img_data);
            cv::cvtColor(img, img_ycrcb,cv::COLOR_RGB2YCrCb);
            break;
        //因为yuv图像特殊格式的问题，y：u：v为4：1：1，即表示一个像素点所需平均bite为rgb的1/2
        case cnstream::CNDataFormat::CN_PIXEL_FORMAT_YUV420_NV12: {
            img = cv::Mat(height * 3 / 2, width, CV_8UC1, img_data);
            cv::Mat bgr(height, width, CV_8UC3);
            cv::cvtColor(img, bgr, cv::COLOR_YUV2BGR_NV12);
            cv::cvtColor(bgr, img_ycrcb, cv::COLOR_BGR2YCrCb);
        } break;
        case cnstream::CNDataFormat::CN_PIXEL_FORMAT_YUV420_NV21: {
            img = cv::Mat(height * 3 / 2, width, CV_8UC1, img_data);
            cv::Mat bgr(height, width, CV_8UC3);
            cv::cvtColor(img, bgr, cv::COLOR_YUV2BGR_NV21);
            cv::imwrite("bgr-post.png",bgr);
            cv::cvtColor(bgr, img_ycrcb, cv::COLOR_BGR2YCrCb);
            cv::imwrite("ycrcb-post.png",img_ycrcb);
        } break;
        default:
            LOG(WARNING) << "[Encoder] Unsupport pixel format.";
            delete[] img_data;
            return -1;
        }
        // resize if needed
        if (height != dst_h || width != dst_w) {
            cv::Mat dst_temp(dst_h, dst_w, CV_8UC3);
            cv::resize(img_ycrcb, dst_temp, cv::Size(dst_w, dst_h));
            img_ycrcb = dst_temp;
        }
        cv::imwrite("ycrcb-post-resize.png",img_ycrcb);
        cv::Mat bgr_resize;
        cv::cvtColor(img_ycrcb,bgr_resize,cv::COLOR_YCrCb2BGR);
        cv::imwrite("img_bgr_resize.png",bgr_resize);
        
        cv::Mat dst(dst_h,dst_w,CV_8UC1);
        std::vector<cv::Mat> channels;
        cv::split(img_ycrcb,channels);

        cv::Mat img_h_resize = channels[0];
        for(int r=0; r<dst_h;r++){
            for(int c=0;c<dst_w;c++){
                if(img_h_resize.at<uint8_t>(r,c)>=240||img_h_resize.at<uint8_t>(r,c)<=15){
                    if(abs(img_h_resize.at<uint8_t>(r,c)-img_h.at<uint8_t>(r,c))>=50)
                        img_h.at<uint8_t>(r,c) = img_h_resize.at<uint8_t>(r,c);
                }
            }
        }
        img_h.convertTo(dst,CV_8U);
        
        channels[0] = dst;
        cv::merge(channels,img_ycrcb);
        cv::imwrite("ycrcb-post-SR.png",img_ycrcb);
        cv::Mat img_dst(dst_h,dst_w,CV_8UC3);
        cv::cvtColor(img_ycrcb,img_dst,cv::COLOR_YCrCb2BGR);
        cv::imwrite("imgSR_dst.png",img_dst);

        return 0;
    }
private:
    DECLARE_REFLEX_OBJECT_EX(PostprocSR, Postproc);
};
    IMPLEMENT_REFLEX_OBJECT_EX(PostprocSR, Postproc);
}
```
c.附录文件：




