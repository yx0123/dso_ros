# ROS Wrapper around DSO: Direct Sparse Odometry (adapted from original README)

# 1. Installation
Set up your catkin workspace by following [this guide](http://wiki.ros.org/catkin/Tutorials/create_a_workspace).
Then,
```
$ cd PATH/TO/CATKIN_WS/src
$ git clone https://github.com/yx0123/dso.git
$ git clone https://github.com/yx0123/dso_ros.git
```
#### 1.1 Required Dependencies

The following instructions are tested on Ubuntu 16.04 and 18.04. Other platforms might work with minor adjustments.
##### eigen3 and boost (required).
Required. Install with

    sudo apt-get install libeigen3-dev libboost-all-dev

#### 1.2 Optional Dependencies

##### OpenCV (highly recommended).
Used to read / write / display images.
OpenCV is **only** used in `IOWrapper/OpenCV/*`. Without OpenCV, respective 
dummy functions from `IOWrapper/*_dummy.cpp` will be compiled into the library, which do nothing.
The main binary will not be created, since it is useless if it can't read the datasets from disk.
Feel free to implement your own version of these functions with your prefered library, 
if you want to stay away from OpenCV.

Install with

	sudo apt-get install libopencv-dev


##### Pangolin (highly recommended).
Used for 3D visualization & the GUI.
Pangolin is **only** used in `IOWrapper/Pangolin/*`. You can compile without Pangolin, 
however then there is not going to be any visualization / GUI capability. 
Feel free to implement your own version of `Output3DWrapper` with your preferred library, 
and use it instead of `PangolinDSOViewer`

Install from [https://github.com/stevenlovegrove/Pangolin](https://github.com/stevenlovegrove/Pangolin)

## 2. Build
```
$ cd PATH/TO/CATKIN_WS
$ source /opt/ros/melodic/setup.bash
$ catkin init
$ catkin config -DCMAKE_BUILD_TYPE=Release
$ catkin build
```
# 3. Usage
Everything as described in the DSO project - only this is for real-time camera input.
Terminal 1:
```
roscore
```
Terminal 2:

```
$ source PATH/TO/CATKIN_WS/devel/setup.bash
$ rosrun dso_ros dso_live image:=IMAGE_TOPIC calib=PATH/TO/CALIB/FILE/CALIB.txt 
```
Example if using [mono2.bag](https://drive.google.com/file/d/1cSEnHqauJ9tl6UBW7JEQIBlxCeBdEElr/view):

```
$ rosrun dso_ros dso_live image:=/airsim_node/CV/front_center/Scene calib=~/catkin_ws/src/dso/AirSim-camera.txt
```
Terminal 3:
```
rosbag play mono2.bag
```

Odometry output published to `dso_odom` topic. Results saved to `dso_ros_result.txt`.

#### 3.1 Calibration File for Pre-Rectified Images
Sample [calibration file](https://github.com/yx0123/dso/blob/master/AirSim-camera.txt)

    Pinhole fx fy cx cy 0
    in_width in_height
    "crop" / "full" / "none" / "fx fy cx cy 0"
    out_width out_height

**Explanation:**
 Across all models `fx fy cx cy` denotes the focal length / principal point **relative to the image width / height**, 
i.e., DSO computes the camera matrix `K` as

		K(0,0) = width * fx
		K(1,1) = height * fy
		K(0,2) = width * cx - 0.5
		K(1,2) = height * cy - 0.5
For backwards-compatibility, if the given `cx` and `cy` are larger than 1, DSO assumes all four parameters to directly be the entries of K, 
and ommits the above computation. 

# Original README below:
# ROS Wrapper around DSO: Direct Sparse Odometry

For more information see
[https://vision.in.tum.de/dso](https://vision.in.tum.de/dso)

This is meant as simple, minimal example of how to integrate DSO from a different project, and run it on real-time input data.
It does not provide a full ROS interface (no reconfigure / pointcloud output / pose output).
To access computed information in real-time, I recommend to implement your own Output3DWrapper; see the DSO code.


### Related Papers

* **Direct Sparse Odometry**, *J. Engel, V. Koltun, D. Cremers*, In arXiv:1607.02565, 2016

* **A Photometrically Calibrated Benchmark For Monocular Visual Odometry**, *J. Engel, V. Usenko, D. Cremers*, In arXiv:1607.02555, 2016



# 1. Installation

1. Install DSO. We need DSO to be compiled with OpenCV (to read the vignette image), and with Pangolin (for 3D visualization).
2. run 

		export DSO_PATH=[PATH_TO_DSO]/dso
		rosmake
	


# 3 Usage
everything as described in the DSO project - only this is for real-time camera input.


		rosrun dso_ros dso_live image:=image_raw \
			calib=XXXXX/camera.txt \
			gamma=XXXXX/pcalib.txt \
			vignette=XXXXX/vignette.png \


## 3.1 Accessing Data.
see the DSO Readme. As of now, there is no default ROS-based `Output3DWrapper` - you will have to write your own.




# 4 Dependencies

## 4.1 Pangolin
removing

	    fullSystem->outputWrapper = new IOWrap::PangolinDSOViewer(
	    		 (int)undistorter->getSize()[0],
	    		 (int)undistorter->getSize()[1]);

will allow you to use DSO compiled without Pangolin. However, then there is no 3D visualization.
You can also implement your own Output3DWrapper to fit your needs.

## 4.2 OpenCV
you can use DSO compiled without OpenCV. 
In that case, the vignette image will not be read, and no photometric calibration can be used. Also, there will not be any image visualizations / image saving.
You can also implement your own version of ImageRW.h / ImageDisplay.h, instead of the dummies.


### 5 License
This ROS wrapper around DSO is licensed under the GNU General Public License
Version 3 (GPLv3).
For commercial purposes, we also offer a professional version, see
[http://vision.in.tum.de/dso](http://vision.in.tum.de/dso) for details.
