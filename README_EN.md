# Inferential Visual Question Generation

## Dependencies

Operating System：ubuntu 16.04

Server GPU：NVIDIA GeForce RTX 3090

Experiments Require：

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

## Datasets

Refer to https://github.com/KaihuaTang/Scene-Graph-Benchmark.pytorch/blob/master/DATASET.md to download and store. (In fact, the basic framework of the project needs to be downloaded, and it will be called when generating the preliminary scene graph, so there is no need to change the relative storage location of the dataset file.)

Download the description text annotation of Visual Genome separately, download region_descriptions.json from http://visualgenome.org/static/data/dataset/region_descriptions.json.zip , and put it together with the above files.

## How to Run

### Install the dependencies

1. First, refer to the installation steps of https://github.com/KaihuaTang/Scene-Graph-Benchmark.pytorch/blob/master/INSTALL.md to install the environment package required for preliminary scene graph generation.

2. Refer to https://github.com/wuzhe71/CPD to download the pre-trained model CPD.th required for salient region detection, and place it in the core-sg-generation folder. Environmental dependencies can be shared with packages installed in the previous step.

3.Additional installation
```
pip3 install -r imageio spacy spacy_lookup sentence_transformers pandas random
```

### Run to generate a preliminary scene graph

Refer to the https://github.com/KaihuaTang/Scene-Graph-Benchmark.pytorch project to generate a preliminary scene graph.

This step is a plug-in method, so various SGG generation methods can be replaced. Here we use the Iterative-Message-Passing (IMP) method with general effects as an example. Note that the SGG task is SGDet, which generates a scene graph completely from image detection. So several important parameters are set as:
```
MODEL.ROI_RELATION_HEAD.USE_GT_BOX False
MODEL.ROI_RELATION_HEAD.USE_GT_OBJECT_LABEL False
MODEL.ROI_RELATION_HEAD.PREDICTOR IMPPredictor
```

### Run to generate the core scene graph

See the folder core-sg-generation for the code.

1. First extract information from the text to enhance the confidence of the preliminary scene graph. testaddarticap.py and testaddcap.py use human-crafted descriptions and noisy description text in VG, respectively, and sortrel.py is used to reorder elements before evaluation in ablation study. Pay attention to change the input path to the result storage path after the previous step.
```
python testaddarticap.py
python testaddcap.py
python sortrel.py
```
2. Generate a core scene graph from the adjusted confidence and salient object detections. gmnewsg.py and gmgtsg.py respectively generate the core scene graph from the data generated in the previous step and the ground truth scene graph.
```
python gmnewsg.py
python gmgtsg.py
```

### Run to generate inferential vision questions

See the folder question-generation for the code.

Run generate_questions_new.py to generate inferential visual questions, --input_scene_dir should be set to the path of the core scene graph generated in the previous step.

Run generate_questions_new_gt.py to generate the comparison questions required in the ablation study, --input_scene_dir needs to be set to the storage path of the ground truth scene graph.
```
python generate_questions_new.py --input_scene_dir [path of the core scene graph]
python gmgtsg.py generate_questions_new_gt.py --input_scene_dir [path of the ground truth scene graph]
```

### Evaluation experiment

See the folder evaluation for the code.

1. Evaluation on proposed new metrics and ROUGE(mean). 
```
python evl.py
python evlgt.py
python evlrouge.py
```

2. Human evaluation (prepare the data first, then run the evaluation tool)
```
python heveldata.py
python heveltk.py
```

## Summary

According to the above steps, the generation of inferential visual questions can be realized, and the results produced during the experiment are basically consistent with those in the paper.

As a generation task, this project may get slightly different generation results in each time, so the samples shown in the paper are for reference only.