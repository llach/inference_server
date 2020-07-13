# inference_server

This package is a wrapper between ROS and the [Tensorflow Object Detection API](https://github.com/tensorflow/models/tree/master/research/object_detection). The Tensorflow Object Detection API is an open source framework built on top of TensorFlow that makes it easy to construct, train and deploy object detection models. This package helps us to use the object detection along with ROS in the form of an action_server. 

## Features:
   * ROS-TensorFlow Object Detection API Wrapper
   * Available as a ROS action_server

## ROS API
### actions
   * Inference.action
   
   ```
	---
	sensor_msgs/CompressedImage image
	int16 num_detections
	string[] classes
	float32[] scores
	sensor_msgs/RegionOfInterest[] bounding_boxes
	---
   ```

## Testing it with the Robot 
  * Run the object detection action_server with the following command:
 ```
rosrun inference_server inference_server_node.py
 ```
  * To run inference on the image published from the robot, run the action_client and send the goal using the following command:
```
rosrun actionlib axclient.py /inference_server inference_server/InferenceAction inference_server/InferenceAction
```
  * Receive the result with the following fields from the inference of the image, in a chronological order of the detection score.
	* image - Resultant image after inference from Object Detection API
	* num_detections - Number of detected objects in the inference image
	* classes - name of the class to which the object belongs (depends on the model used for the inference)
	* scores - detection scores or the confidence of the detection of the particular object as a particular class
	* bounding_boxes - bounding box of each of the detected object

## Local Testing

1. Install tensorflow

```bash
pip install tensorflow==1.15.0 # version is important as we're stuck with py2
```


2. Install the Tensorflow Object Detection API

```bash
cd ~/
git clone  https://github.com/tensorflow/models
cd models
git checkout v1.13.0 # supports py2

cd research
protoc object_detection/protos/*.proto --python_out=. # compile protobuf files
export PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim # must be in models/research (or replace both `pwd`s with absolute path)
```

You can check whether the installation was successful by running

```bash
python -c "import object_detection"
```

3. Download pre-trained weights

```bash
mkdir ~/weights && cd ~/weights
wget http://download.tensorflow.org/models/object_detection/ssd_inception_v2_coco_11_06_2017.tar.gz
```

4. Launch the node
From your catkin workspace, run 

```bash
roslaunch inference_server inference_server.launch \
	 model_database_path:=$HOME/weights \
	 tf_models_path:=$HOME/models/
```

Note, that the node expects images to be published on `/xtion/rgb/image_raw/compressed` by default. You can change this behavior by passing the `in_topic` parameter to the launchfile and pointing it to the image topic of your choice.

5. Make an action call to the node:

```bash
rostopic pub /inference_server/goal inference_server/InferenceActionGoal "header:
  seq: 0
  stamp:
    secs: 0
    nsecs: 0
  frame_id: ''
goal_id:
  stamp:
    secs: 0
    nsecs: 0
  id: ''
goal: {}"

```