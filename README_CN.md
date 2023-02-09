# Inferential Visual Question Generation

## 环境依赖

操作系统：ubuntu 16.04

服务器显卡：NVIDIA GeForce RTX 3090

相关依赖：

- Python <= 3.8
- PyTorch >= 1.2
- torchvision >= 0.4
- cocoapi
- yacs
- matplotlib
- GCC >= 4.9
- OpenCV
- imageio
- spacy
- spacy_lookup
- sentence_transformers
- pandas
- random

## 数据集

参考 https://github.com/KaihuaTang/Scene-Graph-Benchmark.pytorch/blob/master/DATASET.md 下载及存放即可。（实际上该项目的基础框架都需下载下来，在生成初步场景图时会调用，故而不用改动数据集文件的相对存储位置。）

这里下载的数据包括Visual Genome的图像和场景图标注，由于我们需要分类器能够支持识别物体的属性，故而必须要使用这里提供的支持attribute-head的场景图及相应设置。

单另下载Visual Genome的描述文本标注，从 http://visualgenome.org/static/data/dataset/region_descriptions.json.zip 下载region_descriptions.json，与上面的文件放在一起。

## 代码运行

### 安装环境依赖

1.首先参考 https://github.com/KaihuaTang/Scene-Graph-Benchmark.pytorch/blob/master/INSTALL.md 的安装步骤，安装初步场景图生成所需的环境包。如果发现配置更新冲突等问题可以查询该项目的issues页面解决。

2.参考 https://github.com/wuzhe71/CPD 下载显著性区域检测所需的预训练模型CPD.th，放置在core-sg-generation文件夹下。环境依赖可与上一步安装包的共用，若有冲突可查询该项目的issues页面解决。

3.补充安装
```
pip3 install -r imageio spacy spacy_lookup sentence_transformers pandas random
```

### 运行生成初步场景图

参照 https://github.com/KaihuaTang/Scene-Graph-Benchmark.pytorch 项目生成初步场景图。

该步骤为插件式方法，故而可以更换各种SGG生成方法，在这里我们采用了一般性效果的Iterative-Message-Passing(IMP)方法进行示例，注意SGG任务为完全从图像检测生成场景图的SGDet任务。所以几个重要的参数设置为：
```
MODEL.ROI_RELATION_HEAD.USE_GT_BOX False
MODEL.ROI_RELATION_HEAD.USE_GT_OBJECT_LABEL False
MODEL.ROI_RELATION_HEAD.PREDICTOR IMPPredictor
```

### 运行生成核心场景图

代码见文件夹core-sg-generation

1.首先从文本中提取信息，增强初步场景图的置信度。testaddarticap.py和testaddcap.py分别使用人工制作的描述和VG中有噪的描述文本，sortrel.py用来在消融实验评估之前对元素进行重排序。注意将输入路径改为上一步运行后的结果存储路径。
```
python testaddarticap.py
python testaddcap.py
python sortrel.py
```

2. 根据调整后的置信度和显著目标检测生成核心场景图。gmnewsg.py, gmgtsg.py分别从上步生成的数据、基本事实场景图图来生成核心场景图。
```
python gmnewsg.py
python gmgtsg.py
```


### 运行生成推理性视觉问题

代码见文件夹question-generation

运行generate_questions_new.py可生成推理性视觉问题，--input_scene_dir要设置为上一步生成的核心场景图的路径。

运行generate_questions_new_gt.py可生成消融研究中所需的对比问题，--input_scene_dir需设置为基本事实场景图的存储路径。
```
python generate_questions_new.py --input_scene_dir [核心场景图存储路径]
python gmgtsg.py generate_questions_new_gt.py --input_scene_dir [基本事实场景图存储路径]
```

### 评估实验

代码见文件夹evaluation

1.提出的新指标及ROUGE均值评估
```
python evl.py
python evlgt.py
python evlrouge.py
```

2.人工评价（先准备数据，后运行评估工具）
```
python heveldata.py
python heveltk.py
```

## 小结

按以上步骤可实现推理性视觉问题的生成，实验过程中产生的结果与论文中基本一致。

本项目作为生成任务可能会在每一次生成中得到略微不同的生成结果，故而文中展示的样例仅作参考。