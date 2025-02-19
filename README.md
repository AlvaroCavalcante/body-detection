# Large-Scale Dataset and Benchmarking for Hand and Face Detection Focused on Sign Language

Detecting the hands and the face is an important task for sign language, as these channels contain the majority of the information necessary for classifying signs. This repository includes the source code, pre-trained models, and the dataset developed in our [paper](https://www.esann.org/sites/default/files/proceedings/2023/ES2023-185.pdf), accepted on [ESANN](https://www.esann.org/) (31th European Symposium on Artificial Neural Networks, Computational Intelligence and Machine Learning). Although the models and dataset can be used for other problems, they are specifically designed for the domain of sign language, contributing to further research in this field.

## Sign Language Hand and Face Dataset

The large-scale hand and face dataset for sign language is based on the [AUTSL](https://cvml.ankara.edu.tr/datasets/) dataset, which contains 43 interpreters, 20 backgrounds, and more than 36,000 videos. To create the annotations, we trained an initial detector using the [Autonomy](https://autonomy.cs.sfu.ca/hands_and_faces/) data. After that, we employed this initial model and an [auto-annotation tool](https://github.com/AlvaroCavalcante/auto_annotate) to generate the annotations following the PASCAL VOC format. Finally, we manually reviewed all the images and the bounding boxes to fix the mistakes made by the model and better fit the objects. The generated dataset has the following statistics:

| Frames  | Hands  | Faces  |
|---|---|---|
| 477,480  |  954,960 | 477,480

**NOTE:** We detected a maximum of 16 frames per video using a confidence threshold of 35%. The Figure below shows some samples of the dataset.

![Image](/assets/hand_face_example.png "Annotated dataset")

### Dataset split

The dataset was split according to the [Chalearn](https://chalearnlap.cvc.uab.cat/dataset/40/description/) competition guidelines. That said, we employed 31 interpreters for training, 6 for validation, and 6 for testing, ensuring that the same interpreter did not appear in multiple splits. The distribution of images per split amounted to 369,053 for training, 49,041 for testing, and 59,386 for validation.

## Downloading the dataset and pre-trained models

You can download the dataset and pre-trained models in this [link](https://drive.google.com/drive/folders/1cKV8GuqBgVMhf_pAiWu-3zmuNdYcA7Dg?usp=sharing). It's just necessary to ask for permission to use a Google account, and I will share the dataset with you as soon as possible.

The folder "**saved_models.zip**" contains each of the models trained in this research. As the name suggests, the models were saved using the [SavedModel](https://www.tensorflow.org/guide/saved_model) format. The folder "**hand_face_detection_dataset.zip**", on the other hand, contains all the images and labels, totaling around 26 GB of data. The folder structure is as follows:

```bash
├── labels
│   ├── validation
│   │   ├── *labels.txt*
│   ├── train
│   ├── test
├── images
│   ├── validation
│   │   ├── *images.jpg*
│   │   ├── *labels.xml*
│   ├── train
│   ├── test
```

The folder named "images" contains all the images and the labels in PASCAL VOC (XML) format. The "labels" folder, in contrast, contains the labels in ".txt" format for YOLO.

### TFRecord creation

In order to train the models using TensorFlow, the initial step involves converting the images and XML annotations into the TFRecord format. This conversion process mandates the creation of a CSV file that establishes the correlation between the images and their corresponding annotations. This can be achieved using the following command:

```bash
python3 utils/xml_to_csv.py -i /xml-input-path -o /csv-output-path
```

Where "xml-input-path" represents the path to the folder containing the XML files, and "csv-output-path" designates the location for the resulting CSV file. After that, the TFRecord files can be generated through the execution of the subsequent command:

```bash
python3 utils/generate_tfrecord.py --csv_input=/path-to-csv --output_path ./output.record --img_path=/path-to-images --label_map=src/utils/label_map.pbtxt --n_splits n_files_to_generate 
```

The command parameters are the following:

- csv_input: The path of the csv file generated by the previous script.
- output_path: The output path of the TFRecord files.
- img_path: The path to the folder containing the images.
- label_map: The path to the label_map.pbtxt file (available in src/utils).
- n_splits: The number of TFRecord files to generate.

For our dataset, it is advised to generate a total of 15 TFRecord files for both the test and validation sets, while the training set requires 110 files. Each of these individual files is approximately 200 MB in size.

## Object Detection Results

We trained and optimized different object detection architectures for the given task of hand and face detection for sign language, achieving good results while reducing the models' complexity. The Table bellow shows the mean Average Precision (mAP) and inference time (milliseconds) of each detector. The values in parentheses correspond to the inference time before applying the optimizations.

**Note:** CPU Intel Core I5 10400, GPU Nvidia RTX 3060.

| Architecture  | Inf. time CPU  | Inf. time GPU  | mAP@50 | mAP@75 |
|---|---|---|---|---|
| SSD640  |  53.2 (108.0) | 11.6 (44.1) | 98.5 | 95.0
| SSD320  |  **15.7** (32.7) | 9.9 (25.7) | 92.1 | 73.1
| EfficientDet D0  |  67.8 (124.5) | 16.1 (53.4) | 96.7 | 85.8
| YoloV7  |  123.9 (211.1) | **7.4** (7.6) | 98.6 | 95.7
| Faster R-CNN  |  281.0 (811.5) | 26.3 (79.1) | **99.0** | 96.2
| CenterNet  |  40.0 | 7.9 | **99.0** | **96.7**

As observed, the fastest models achieved over 135 frames per second (FPS) on GPU (YoloV7) and 63 FPS on CPU (SSD320), reaching a real-time performance for the task of hand and face detection.

The models were trained using the [TensorFlow Object Detection API](https://github.com/tensorflow/models/tree/master/official#object-detection-and-segmentation) and the configuration files of each architecture can be found at **src/utils/pipelines**, making it easy to reproduce the results. To understand in detail how optimizations were made, refer to the original paper (available soon).

## Project setup

The project was developed using Python 3.8, but it's probably compatible with newer versions. It's recommended to use a [virtual environment](https://docs.python.org/pt-br/3/library/venv.html) to complete the setup on your machine. After creating the venv, you can install the dependencies using the following command:

```bash
pip install -r requirements.txt
```

If you want to retrain the models, you'll also need to install the [TensorFlow Object Detection API](https://github.com/tensorflow/models/tree/master/research/object_detection). There are some great tutorials on how to do that, like [this one](https://neptune.ai/blog/how-to-train-your-own-object-detector-using-tensorflow-object-detection-api). Finally, if you have a GPU available, follow this instructions to setup [TensorFlow on GPU](https://www.tensorflow.org/install/pip#windows-native_1).

## Using the trained models

You can use the **hand_face_detection.py** script to find the model that better works for you. To run the code, use the following arguments:

- **saved_model_path**: Path of the saved_model folder that contains the *saved_model.pb* file.
- **source_path**: Path of the video file to test the model. The default behavior is to use the webcam stream. There is a benchmarking video inside *utils/test_videos* folder that can be used to test the models.
- **label_map_path**: Path of the label map file (defaults to src/utils/label_map.pbtxt).
- **show_results**: Either to show or not the detection results. Defaults to True.
- **img_res**: Image resolution. Defaults to 512x512.
- **device**: Device to run the model. Defaults to cpu.

Here is an example of how to run the code:

````bash
python src/hand_face_detection.py --saved_model_path C:\Users\saved_models\centernet_mobilenet_v2_fpn\saved_model --device gpu --img_res 640
````

If everything worked fine, you'll see your detections:

<p align="center">
    <img src="assets/libras_gif.gif" width="480" height=840>
    <br>
    <sup>Sign language interpreter <a href="https://www.linkedin.com/in/esther-sato-ramos-cavalcante-a6b125254/" target="_blank">Esther Sato</a> testing hand and face detection.</sup>
</p>

After the model inference, a file called "output.avi" is produced with the detection results.

## **Train, test, and export new models**

As mentioned above, the training, evaluation, and export of the object detection models were made using TF object detection API. After cloning the repository and installing the dependencies, the training can be done with the following command:

```bash
python models/research/object_detection/model_main_tf2.py \
    --pipeline_config_path={pipeline_fname} \
    --model_dir={model_dir} \
    --alsologtostderr \
    --num_train_steps={num_steps} \
    --checkpoint_every_n=1000 \
    --num_eval_steps={num_eval_steps}
```

Where the main arguments are:

- **pipeline_config_path**: Path of the pipeline file (.config). Use one of the pipeline files inside src/utils/pipelines to have similar results to those reported in this research.
- **model_dir**: Path of the folder to save the model checkpoint.
- **num_train_steps & num_eval_steps**: Number of steps to train and evaluate the model. Each step represents a batch of data.

To evaluate the model performance during the training (or after, if you prefer), you just need to run the following command:

```bash
python models/research/object_detection/model_main_tf2.py \
    --pipeline_config_path={pipeline_fname} \
    --model_dir={model_dir} \
    --alsologtostderr \
    --eval_on_train_data=True \
    --checkpoint_dir={model_dir} \
```

When specifying the **checkpoint_dir** parameter, the last checkpoint will be used to evaluate the model performance on eval data.

Finally, after training your object detector, you would probably like to export it to **savedModel** format for use in the inference code. To do so, just run the following command:

```bash
python models/research/object_detection/exporter_main_v2.py \
    --input_type image_tensor \
    --pipeline_config_path {pipeline_fname} \
    --trained_checkpoint_dir {model_dir} \
    --output_directory {output_path}
```

## Doubts and contribution

If you have any doubts or trouble when using this project, open a new issue on this GitHub repository, or send an e-mail to <alvaroleandro250@gmail.com>. Also, contributions are always welcome, feel free to open a pull request or give any suggestions on how to improve this project.
