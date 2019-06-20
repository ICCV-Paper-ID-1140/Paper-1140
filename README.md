# Polarity Loss for Zero-shot Object Detection

This code is the official implementation of the following work (train + eval):

"Polarity Loss for Zero-shot Object Detection." 


## Requirements

* Other requirements:
    - Python 2.7 (or 3.6)
    - Keras 2.1.4
    - OpenCV 3.3.0
    - Tensorflow 1.3.0
 * We have provided `.yaml` files in the `Config/` directory to seamlessly set-up conda enviornemnt with all the required dependencies. Simply run `conda env create --file=environment_keras_pyZZ.yaml` and replace `ZZ` with `27` or `36` depending on the python version. 

## Files

* `sample_input.txt`: a sample input file containing test image paths
* `gzsd.py`: to perform generalized zero-shot detection task using sample_input.txt
* `keras_retinanet`: directory containing the supporting code of the model. This directory is a modified version from original RetinaNet implementation [1] ([Link](https://github.com/fizyr/keras-retinanet))
    - `bin/train_vocab_w2v.py`: use this file to train the PL-ZSD network on MSCOCO dataset
    - `bin/evaluate.py`: use this file to evaluate the trained PL-ZSD network on MSCOCO dataset
* `Dataset`: directory containing sample input and output images. For training, you will have to place train and val datasets in this directory.
* `Config`: directory containing configuration files to set up conda environment. 
* `Model`: directory containing a pre-trained model using the polarity loss `resnet5-_polar_loss.h5`, used in the `gzsd.py` demo code. If you want to train the model, you will need to place a version of retina-net trained with wordvectors `resnet50_csv_50_focal_seen_w2v.h5` in this directory to initiate the training process. 
* `MSCOCO`: This directory contains the source of data, proposed 65/15- seen/unseen split and experimental protocol used in experiments.
    - `cls_names_seen_coco.csv`: list of 65 MSCOCO seen classes. Each line contains a class name followed by an index.
    - `cls_names_test_coco.csv`: list of 80 MSCOCO object classes. Each line contains a class name followed by an index. Index 0 to 64 are from seen objects, and index 65 to 79 are from unseen.
    - `train_coco_seen_all.zip`: it is a zip version of csv file `train_coco_seen_all.csv` containing training image paths and annotations used in the paper. Each line contains a training image path, a bounding box co-ordinate and the ground-truth class name of that bounding box. For example, Filepath,x1,y1,x2,y2,class_name
    - `validation_coco_seen_all.csv`: test images with annotations for traditional object detection on only seen objects. File format, Filepath,x1,y1,x2,y2,class_name
    - `validation_coco_unseen_all.csv`: test images with annotations for zero-shot object detection on only unseen objects. File format, Filepath,x1,y1,x2,y2,class_name
    - `validation_coco_unseen_seen_all_gzsd.csv`: test images with annotations for generalized zero-shot object detection on both seen and unseen objects together. File format, Filepath,x1,y1,x2,y2,class_name
    - `word_w2v.txt`, `word_glo.txt`, and `word_ftx.txt`: word2vec, GloVe and FastText word vectors for 80 classes of MSCOCO.  The ith column represents the 300-dimensional word vectors of the class name of the ith row of `cls_names_test_coco.csv`
    - `vocabulary_list`: The list of vocabulary atoms from NUS-WIDE tag dataset [2] used in the paper.
    - `vocabulary_w2v.txt`, `vocabulary_glo.txt`, and `vocabulary_ftx.txt`: word2vec, GloVe and FastText word vectors of all vocabulary tags.  The ith column represents the 300-dimensional word vectors of the class name of the ith row of `vocabulary_list.txt`

## Running instructions
* **Running Demo Code:** To run generalized zero-shot detection on sample input kept in `Dataset/Sampleinput`, simply run `gzsd.py` after installing all dependencies like Keras, Tensorflow, OpenCV or alternatively use the `.yaml` file (see above under `Requirements`) to create a new environment with all dependencies. Place the pre-trained model available from ([Link to pre-trained model for demo (h5 format)](https://www.dropbox.com/s/97gfrngizymricd/resnet50_polar_loss.h5?dl=0)) in the `Model` directory. This code will generate the output files for each input image to `Dataset/Sampleoutput`.
* **Running Train/Test Code on MSCOCO:** Extract the dataset `train2014.zip` and `val2014.zip` inside the folder Dataset. These files are downloadable from [Link](http://cocodataset.org/#download). Make sure the pre-trained model is present inside the Model folder ('Model/resnet50_csv_50_focal_seen_w2v.h5'). This pre-trained model is trained by focal loss on 65 seen classes without considering any vocabulary metric. This model is available to download from ([Link to pre-trained model for training (h5 format)](https://www.dropbox.com/s/dc0vit1dj83rd56/resnet50_csv_50_focal_seen_w2v.h5?dl=0)). Also, make sure the `snapshots` folder is already created to store intermediate models of each epoch. Then, run the following commands for training and testing.
- Training: `python keras_retinanet/bin/train_vocab_w2v.py --snapshot-path ./snapshots csv MSCOCO/train_coco_seen_all.csv MSCOCO/cls_names_seen_coco.csv`
- Testing GZSD: 
`python keras_retinanet/bin/evaluate.py csv MSCOCO/validation_coco_unseen_seen_all_gzsd.csv MSCOCO/cls_names_test_coco.csv snapshots/resnet50_csv_30.h5` 

## Notes on MSCOCO experiments
The resources required to reproduce results are kept in the directory `MSCOCO`. For training and testing, we used MSCOCO-2014 train images from `train2014.zip` and validation images from `val2014.zip`. These zipped archives are downloadable from MSCOCO website ([Link](http://cocodataset.org/#download)). Please find the exact list of images (with annotations) used for "training" in `MSCOCO/train_coco_seen_all.csv`. The lists of images used for "testing" different ZSL settings are:
* For traditional detection task: `MSCOCO/validation_coco_seen_all.csv`, 
* For zero-shot detection task: `MSCOCO/validation_coco_unseen_all.csv`, and 
* For generalized zero-shot detection task: `MSCOCO/validation_coco_unseen_seen_all_gzsd.csv`.


*The above results are for Generalized Zero-shot detection setting. The seen/unseen objects are enclosed in yellow/red bounding boxes.*

## Tests in the wild
We run the PL-ZSD model on two example videos from the [Youtube-8M](https://research.google.com/youtube8m/) dataset from Google AI. The demo videos contain several seen (e.g., pottend plant, person, hand-bag) and unseen classes (cat, train, suitcase). Note that we do not apply any pre/post processing procedure across temporal domain to smooth out the predictions. 

<!-- [![](http://img.youtube.com/vi/Qi5HfHatVXE/0.jpg)](http://www.youtube.com/watch?v=Qi5HfHatVXE "Demo Video (Cats)") 
[![](http://img.youtube.com/vi/UJFUqjEd3Rw/0.jpg)](http://www.youtube.com/watch?v=UJFUqjEd3Rw "Demo Video (Train station)") 

-->

*The above results are for Generalized Zero-shot detection setting. The seen/unseen objects are enclosed in yellow/red bounding boxes.*


## Reference
[1] Lin, Tsung-Yi, Priyal Goyal, Ross Girshick, Kaiming He, and Piotr Dollár. "Focal loss for dense object detection." IEEE transactions on pattern analysis and machine intelligence, 2018.

[2] Chua, Tat-Seng, et al. "NUS-WIDE: a real-world web image database from National University of Singapore." Proceedings of the ACM international conference on image and video retrieval. ACM, 2009.

