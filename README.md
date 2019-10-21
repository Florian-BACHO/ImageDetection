# ImageDetection
Image Detection using TensorFlow 1.14

## How to make a dataset and train the model ?

### Make dataset

* Use [4K Storygram](https://www.4kdownload.com/fr/products/product-stogram) software to download images on instagram (with hashtags). Get at least 1000 images.
* Create a directory called "dataset".
* Create 2 subdirectories: one for the train set (75% of your dataset), and one for test set (25%) and move downloaded images in it.
* Use [labelimg](https://github.com/tzutalin/labelImg) to annotate your dataset by using the label "tofind"

### Install protoc

Install protobuf if not installed yet.

On Linux:
```shell
wget https://github.com/google/protobuf/releases/download/v3.3.0/protoc-3.3.0-linux-x86_64.zip
chmod 775 protoc-3.3.0-linux-x86_64.zip
unzip protoc-3.3.0-linux-x86_64.zip
rm -rf protoc-3.3.0-linux-x86_64.zip
mv bin/protoc /usr/local/bin
rm -rf bin include readme.txt
```

On macOS:
```shell
brew install protobuf
```

### Download tensorflow models

Clone [models repository](https://github.com/tensorflow/models) by tensorflow:
```shell
git clone https://github.com/tensorflow/models.git
```

### Compile and setup training environment

* Execute:
```shell
cd models/research
protoc object_detection/protos/*.proto --python_out=.
export PYTHONPATH=$PYTHONPATH:`pwd`
export PYTHONPATH=$PYTHONPATH:`pwd`/slim
cd ../..
```

* Create a data directory at the same level as the models directory
* Next, copy provided object-detection.pbtxt and ssd_mobilenet_v1_coco.config files into the data directory

* Download ssd_mobilenet_v1_coco model:
```shell
cd data
wget http://download.tensorflow.org/models/object_detection/ssd_mobilenet_v1_coco_2018_01_28.tar.gz
tar -xvf ssd_mobilenet_v1_coco_2018_01_28.tar.gz
rm -f ssd_mobilenet_v1_coco_2018_01_28.tar.gz
cd ..
```

### Convert dataset
* Move your dataset directory into the data directory.

* Use provided xml_to_csv.py python script to convert xml label files to csv files. From the parent directory of the data directory:
```shell
python3 xml_to_csv.py
```

* Next, you need to convert csv to tfrecords files to feed tensorflow model:
```shell
python3 generate_tfrecord.py --csv_input=data/train.csv --output_path=data/train.record --image_dir=data/dataset/train
python3 generate_tfrecord.py --csv_input=data/test.csv --output_path=data/test.record --image_dir=data/dataset/test
```

### Launch training

* Create a "training" directory into the data directory:
```shell
mkdir data/training
```

* Make that PYTHONPATH has been set correctly (in models/research/) and execute:
```shell
python3 models/research/object_detection/legacy/train.py --logtostderr --train_dir=data/training --pipeline_config_path=data/ssd_mobilenet_v1_coco.config
```

* You can watch the evolution of the training using tensorboard:
```shell
tensorboard --logdir=data/training
```

### Evaluate model

* Execute:
```shell
python3 models/research/object_detection/legacy/eval.py --logtostderr --pipeline_config_path=data/ssd_mobilenet_v1_coco.config --checkpoint_dir=data/training/ --eval_dir=data/eval/
tensorboard --logdir=data/eval/
```

* Go on your browser and visit: 127.0.0.1:6006