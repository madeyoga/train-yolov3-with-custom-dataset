# Train-yolov3-with-custom-dataset
Train custom object detection yolov3 with your own custom dataset using cv2, LabelImg and Darknet Python and Google Colab. 


### 0. Gather Dataset & Download LabelImg
- Gather your own dataset and Download [labelImg by tzutalin](https://github.com/tzutalin/labelImg/releases/tag/v1.8.1)

### 1. Label Data to Yolo Annotation & zip it & upload it to google drive.
- Follows yolo steps https://github.com/tzutalin/labelImg#steps-yolo
- Make sure annotations & images are at the same directory

### 2. Prepare Environment
- Connect Google Drive
```py
from google.colab import drive
drive.mount('/content/gdrive')
!ln -s /content/gdrive/My\ Drive/ /mydrive
!ls /mydrive
```
- Clone darknet repository 
```sh
!git clone https://github.com/AlexeyAB/darknet.git
```
- Use makefile to enable GPU & OPENCV
```sh
%cd darknet
!sed -i 's/OPENCV=0/OPENCV=1/' Makefile
!sed -i 's/GPU=0/GPU=1/' Makefile
!sed -i 's/CUDNN=0/CUDNN=1/' Makefile
!make
```
### 3. Make a copy & rename the yolov3 configuration
https://github.com/AlexeyAB/darknet/blob/master/cfg/yolov3.cfg

### 4. Edit the configuration file
- Change filters, example for 1 class: `filters=18`. filters=(classes + 5)x3
- Change `[filters=255]` to `filters=(classes + 5)x3` in the 3 [convolutional] before each [yolo] layer, keep in mind that it only has to be the last [convolutional] before each of the [yolo] layers.<br>
So if `classes=1` then should be `filters=18`. If `classes=2` then write `filters=21`.

- Full Guide: https://github.com/AlexeyAB/darknet#how-to-train-to-detect-your-custom-objects
```sh
!sed -i 's/batch=1/batch=64/' cfg/yolov3_training.cfg
!sed -i 's/subdivisions=1/subdivisions=16/' cfg/yolov3_training.cfg
!sed -i 's/max_batches = 500200/max_batches = 6000/' cfg/yolov3_training.cfg
!sed -i '610 s@classes=80@classes=1@' cfg/yolov3_training.cfg
!sed -i '696 s@classes=80@classes=1@' cfg/yolov3_training.cfg
!sed -i '783 s@classes=80@classes=1@' cfg/yolov3_training.cfg
!sed -i '603 s@filters=255@filters=18@' cfg/yolov3_training.cfg
!sed -i '689 s@filters=255@filters=18@' cfg/yolov3_training.cfg
!sed -i '776 s@filters=255@filters=18@' cfg/yolov3_training.cfg
```

### 5. Unzip dataset to data/obj directory & Create :
- data/obj.names -> list of classes
- data/obj.data -> path information

```sh
classes=1
train=data/train.txt
valid=data/test/txt
names=data/obj.names
backup=path_to_project
```

- Example Code for 2 classes:
```sh
!echo -e "left_data\nright_data" > data/obj.names
!echo -e 'classes= 2\ntrain = data/train.txt\nvalid = data/test.txt\nnames = data/obj.names\nbackup = /mydrive/yolov3' > data/obj.data
```

### 6. Create data/train.txt -> list of images filepath
- data/obj/*.jpg

```py
import glob
images_list = glob.glob("data/obj/*[jpg|png|jpeg]")
print(images_list)

#Create training.txt file
file = open("data/train.txt", "w") 
file.write("\n".join(images_list)) 
file.close() 
```

### 7. Download Pretrained Model
```sh
!wget https://pjreddie.com/media/files/darknet53.conv.74
```

### 8.  Start training 
```sh
!./darknet detector train data/obj.data cfg/yolov3_training.cfg <pretrained_model> -dont_show
```
