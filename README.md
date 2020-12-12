# Nvidia_TLT_DS
Explanation of model training in Transfer learning Toolkit and deploying inn deepstream. Build and deployed an example of AI face mask Detector.


tep by step procedure:


Download the data such that all images and label files are in one folder.


Convert dataset to KITTI format:
Modify the file "detectnet_v2_tfrecords_kitti_trainval.txt";
Specify the directory for images and labels;
specify the partition mode to random or sequence-wise partitioning(used for splitting the data to N folds).


Create the tfrecords using the tlt-dataset-convert


Download pretrained model from Ngc registry.


Provide training specification in the "detectnet_v2_train_resnet18_kitti.txt" file:


a) Update the fold number to use for evaluation. In case of random data split, please use fold 0.
b) Augmentation and Preprocessing:
Preprocessing:Specify scaling in this part.It sets the shape of the input tensor.you can add parameters such as crop,scale etc.
Spatial_augmentation: supports basic augmentation such as flip,zoom,translate which may be configured accordingly.
colour_augmentation: supports color augmenttion such as color_shift,hue_rotation,saturation_shift,contrast_adjustment etc.
c) Also provide class labels mapping and their configuration accordingly.
d) Specify Other training (hyper-)parameters such as batch size, number of epochs, learning rate regularization, optimizer etc.
Note 1: for object detection you can also specify config values for anchor boxes,IOU etc.
6)Train the model using tlt-train

Evaluate the model using tlt-evaluate and check the accuracy.

8)Prune the trained model using tlt-prune:
Usually, you just need to adjust -pth (threshold) for accuracy and model size trade off. Higher pth gives you smaller model (and thus higher inference speed) but worse accuracy.The threshold to use is depend on the dataset. A pth value 5.2e-6 is just a start point.If the retrain accuracy is good, you can increase this value to get smaller models. Otherwise, lower this value to get better accuracy.
9)Retrain the prune model:
Model needs to be re-trained to bring back accuracy after pruning.
Specify re-training specification with pretrained weights as pruned model in this "detectnet_v2_retrain_resnet18_kitti.txt" file.


After training and evaluating again, check whether accuracy is satisfactory. if not try using different 				augmentations configs.


Exporting the model using tlt-export:
We can also perform quantization using tlt-int8-tensorfile. that will generate calibration cache which can be used 		with tlt-converter for genearting model engine.


Deploy
There are two ways through which we can deploy model to Deepstream.
a) Integrate the model(.etlt) with the encrypted key directly in the deepstream.
b) generate a device specific optimized TensorRT engine using tlt-converter.


Deployment using deepstream:
->we have primary inference file "config_infer_primary_masknet_gpu.txt" where we update paths to our model.
-> Also, we have deepstream app file "deepstream_app_source1_video_masknet_gpu.txt" where we specify our input and output configs.
deploy using Deepstream-app -c deepstream_app_source1_video_masknet_gpu.txt
