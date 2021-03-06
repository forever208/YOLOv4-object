# YOLOv4-object: an efficient model and method for obejct discovery

[![Darknet](https://img.shields.io/badge/Powered%20by-Darknet-green)](https://github.com/AlexeyAB/darknet) [![Darknet](https://img.shields.io/badge/Train%20on-Colab-yellow)](https://colab.research.google.com/drive/11jaSKfF74bPPVO0-9o-ZgLE5yVWUeI3d?usp=sharing)

This repo is based on [Darknet](https://github.com/AlexeyAB/darknet).

### Abstract
Object discovery refers to recognising all unknown objects in images, which is crucial for robotic systems to explore the unseen environment. Recently, object detection models based on deep learning have shown remarkable achievements in object classification and localisation. However, these models have difficulties handling the unseen environment because it is infeasible to exhaustively predefine all types of objects. In this paper, we propose the model YOLOv4-object to recognise all objects in images by modifying the output space of YOLOv4 and related image labels. Experiments on COCO dataset demonstrate the effectiveness of our method by achieving 67.97% recall (6.49% higher than vanilla YOLOv4). We point out that the incomplete labels (COCO only labels for 80 categories) hurt the learning process of object discovery and a higher recall can be achieved by our method if the dataset is fully labelled. Moreover, our approach is transferable, extensible, and compressible, showing broad application scenarios. Finally, we conduct extensive experiments to illustrate the factors that affect the object discovery performance of our model and some suggestions on practical implementations are elaborated.

### 【1】Indtroduction
The comparison between object detection and object discovery in our project is shown below:
  - object detection indicates the category 
  - object discovery focuses on detecting all unknown objects, in our model ,all objects are recognised as "object"

<p align="left">
  <img src="https://github.com/forever208/YOLOv4-object/blob/master/data/demo.png" width='100%' height='100%'/>
</p>


We finetuned YOLOv4-object by 480 fully labelled images and witnessed a significant improvement of recall by discovering more objects.

The contrast of YOLOv4-object and finetune model can be seen from the below image:

<p align="left">
  <img src="https://github.com/forever208/YOLOv4-object/blob/master/data/object_vs_object_finetune.gif" width='70%' height='70%'/>
</p>


Our experimental results are shown in the table below:

Note that, for equal comparison, we modified the NMS process of YOLO from doing NMS for each class to doing NMS for all classes simultaneously.
the code mofification can be found in `src/box.c/void diounms_sort()`, around line 916.

| Models                    | Class  amount | Class names          | True Positive | False Positive | False Negative | Recall | mAP@50 | BFlops | FPS (Tesla V100) |
|---------------------------|---------------|----------------------|---------------|----------------|----------------|--------|--------|--------|------------------|
| YOLOv4                    | 80            | human, car, chair... | 22612         | 9754           | 14169          | 61.48% | 67.54% | 60.1   | 101              |
| YOLOv4-obejct             | 1             | object               | 23955         | 11282          | 12826          | 65.13% | 69.08% | 59.6   | 104              |
| YOLOv4-human              | 1             | human                | 8306          | 2871           | 2698           | 75.48% | 79.37% | 59.6   | 104              |
| YOLOv4-human-object       | 2             | human, object        | 23476         | 11132          | 13305          | 63.83% | 71.04% | 59.6   | 104              |
| YOLOv4-object (slim)      | 1             | object               | 23790         | 10813          | 12991          | 64.68% | 69.00% | 51.4   | 115              |
| YOLOv4-object (finetune)  | 1             | object               | 25001         | 19935          | 11780          | 67.97% | 65.77% | 59.6   | 104              |
| YOLOv4-tiny               | 80            | human, car, chair... | 12822         | 6920           | 23959          | 34.86% | 38.97% | 6.9    | 505              |
| YOLOv4-tiny-object        | 1             | object               | 13467         | 5830           | 23314          | 36.61% | 45.16% | 6.8    | 523              |
| YOLOv4-tiny-object (slim) | 1             | object               | 14596         | 7609           | 22185          | 39.68  | 45.53  | 6.7    | 558              |


### 【2】Install Darknet (Linux)
clone Darknet
```sh
$ git clone https://github.com/forever208/YOLOv4-object.git
$ cd YOLOv4-object
```

change the Makefile for using GPU
```sh
$ sed -i 's/OPENCV=0/OPENCV=1/' Makefile
$ sed -i 's/GPU=0/GPU=1/' Makefile
$ sed -i 's/CUDNN=0/CUDNN=1/' Makefile
$ sed -i 's/CUDNN_HALF=0/CUDNN_HALF=1/' Makefile
```

build darknet
```sh
$ make
```

download yolov4.weights and test YOLOv4 on your images
```sh
$ wget https://github.com/AlexeyAB/darknet/releases/download/darknet_yolo_v3_optimal/yolov4.weights
$ ./darknet detector test cfg/coco.data cfg/yolov4.cfg yolov4.weights <img_path> -thresh 0.3
```

### 【3】Run YOLOv4-object
First download our weights file (yolov4-obj_6577.weights) 
```sh
$ wget --load-cookies /tmp/cookies.txt "https://docs.google.com/uc?export=download&confirm=$(wget --quiet --save-cookies /tmp/cookies.txt --keep-session-cookies --no-check-certificate 'https://docs.google.com/uc?export=download&id=1-Z3nHBPEWpAEJ8-PkNKAyt1K9oB3FtNX' -O- | sed -rn 's/.*confirm=([0-9A-Za-z_]+).*/\1\n/p')&id=1-Z3nHBPEWpAEJ8-PkNKAyt1K9oB3FtNX" -O yolov4-obj_6577.weights && rm -rf /tmp/cookies.txt
```
Run the model YOLOv4-obejct on images, video or test its FPS on your computer.

**test on images**
```sh
$ ./darknet detector test <your obj.data> <your config> <your weights file> <your image> -thresh 0.3
# for example
$ ./darknet detector test data/obj.data cfg/yolov4-obj.cfg yolov4-obj_6577.weights ./data/000000013729.jpg -thresh 0.3
```

**test on videos**
```sh
$ ./darknet detector demo <your obj.data> <your config> <your weights file> -dont_show <your video> -i 0 -thresh 0.3 -out_filename <output path>
# for example
$ ./darknet detector demo data/obj.data cfg/yolov4-obj.cfg yolov4-obj_6577.weights -dont_show ./data/kitchen.mp4 -i 0 -thresh 0.3 -out_filename ./data/prediction.avi 
```

**test FPS**
```sh
$ ./darknet detector demo data/obj.data cfg/yolov4-obj.cfg yolov4-obj_6577.weights -dont_show ./data/kitchen.mp4 -benchmark
```

### 【4】Train your own YOLOv4-object
We recommend you to train your own model in Google Colab for saving the time of environment setting. A step-by-step introdution is included in each Jupyter Notebook.
- [YOLOv4-object](https://colab.research.google.com/drive/11jaSKfF74bPPVO0-9o-ZgLE5yVWUeI3d?usp=sharing)
- [YOLOv4-object (slim)](https://colab.research.google.com/drive/1zUucq9y5NeTvI5E7hCF4p2tTAh-TvZwO?usp=sharing)
- [YOLOv4-human-object](https://colab.research.google.com/drive/1N_hlL21sLejqjiJ-b9KXavcZ_guknmt3?usp=sharing)

We also highly recommend you to finetune our YOLOv4-object model using images that in your application scenario, the detailed training process can be found in 
- [YOLOv4-object (finefune)](https://colab.research.google.com/drive/17QSl3Eh3d1-MQH4xTpzMhUCTSLhSj7mN?usp=sharing)

If you run into problems in Colab, feel free to check this [tutorial](https://www.youtube.com/watch?v=mmj3nxGT2YQ) out   
