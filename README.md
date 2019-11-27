# vins_fusion_cg

Modified version of [VINS-Fusion](https://github.com/HKUST-Aerial-Robotics/VINS-Fusion) (commit 0c32069 on Oct 9, 2019), an optimization-based multi-sensor state estimator.

-----

VINS-Fusion is an optimization-based multi-sensor state estimator, which achieves accurate self-localization for autonomous applications (drones, cars, and AR/VR). VINS-Fusion is an extension of [VINS-Mono](https://github.com/HKUST-Aerial-Robotics/VINS-Mono), which supports multiple visual-inertial sensor types (mono camera + IMU, stereo cameras + IMU, even stereo cameras only). We also show a toy example of fusing VINS with GPS.

**Features:**
- multiple sensors support (stereo cameras / mono camera+IMU / stereo cameras+IMU)
- online spatial calibration (transformation between camera and IMU)
- online temporal calibration (time offset between camera and IMU)
- visual loop closure

**Related Paper:** (paper is not exactly same with code)

* **Online Temporal Calibration for Monocular Visual-Inertial Systems**, Tong Qin, Shaojie Shen, IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS, 2018), **best student paper award** [pdf](https://ieeexplore.ieee.org/abstract/document/8593603)

* **VINS-Mono: A Robust and Versatile Monocular Visual-Inertial State Estimator**, Tong Qin, Peiliang Li, Shaojie Shen, IEEE Transactions on Robotics [pdf](https://ieeexplore.ieee.org/document/8421746/?arnumber=8421746&source=authoralert)

*If you use VINS-Fusion for your academic research, please cite our related papers. [bib](https://github.com/HKUST-Aerial-Robotics/VINS-Fusion/blob/master/support_files/paper_bib.txt)*

# Prerequisites

* Ubuntu 64-bit 16.04 or 18.04.
* ROS Kinetic or Melodic
* [Ceres Solver](http://ceres-solver.org/installation.html)

# Build

```sh
catkin_make -j2
# or
catkin build
```

# Run

## EuRoC Example

Download [EuRoC MAV Dataset](http://projects.asl.ethz.ch/datasets/doku.php?id=kmavvisualinertialdatasets) to YOUR_DATASET_FOLDER. Take MH_01 for example, you can run VINS-Fusion with three sensor types (monocular camera + IMU, stereo cameras + IMU and stereo cameras).
Open four terminals, run vins odometry, visual loop closure(optional), rviz and play the bag file respectively.
Green path is VIO odometry; red path is odometry under visual loop closure.

* Monocualr camera + IMU

  ```sh
  roslaunch vins vins_rviz.launch
  rosrun vins vins_node <path-to-euroc_mono_imu_config.yaml>
  (optional) rosrun loop_fusion loop_fusion_node <path-to-euroc_mono_imu_config.yaml>
  rosbag play YOUR_DATASET_FOLDER/MH_01_easy.bag
  ```

* Stereo cameras + IMU

  ```sh
  roslaunch vins vins_rviz.launch
  rosrun vins vins_node <path-to-euroc_stereo_imu_config.yaml>  
  (optional) rosrun loop_fusion loop_fusion_node <path-to-euroc_stereo_imu_config.yaml>
  rosbag play YOUR_DATASET_FOLDER/MH_01_easy.bag
  ```

* Stereo cameras

  ```sh
  roslaunch vins vins_rviz.launch
  rosrun vins vins_node <path-to-euroc_stereo_config.yaml>
  (optional) rosrun loop_fusion loop_fusion_node <path-to-euroc_stereo_config.yaml>
  rosbag play YOUR_DATASET_FOLDER/MH_01_easy.bag
  ```

## KITTI Example

### KITTI Odometry (Stereo)

Download [KITTI Odometry dataset](http://www.cvlibs.net/datasets/kitti/eval_odometry.php) to YOUR_DATASET_FOLDER. Take sequences 00 for example,
Open two terminals, run vins and rviz respectively.
(We evaluated odometry on KITTI benchmark without loop closure funtion)

```sh
roslaunch vins vins_rviz.launch
(optional) rosrun loop_fusion loop_fusion_node <path-to-kitti_config00-02.yaml>
rosrun vins kitti_odom_test <path-to-kitti_config00-02.yaml> YOUR_DATASET_FOLDER/sequences/00/
```

### KITTI GPS Fusion (Stereo + GPS)

Download [KITTI raw dataset](http://www.cvlibs.net/datasets/kitti/raw_data.php) to YOUR_DATASET_FOLDER. Take [2011_10_03_drive_0027_synced](https://s3.eu-central-1.amazonaws.com/avg-kitti/raw_data/2011_10_03_drive_0027/2011_10_03_drive_0027_sync.zip) for example.
Open three terminals, run vins, global fusion and rviz respectively.
Green path is VIO odometry; blue path is odometry under GPS global fusion.

```sh
roslaunch vins vins_rviz.launch
rosrun vins kitti_gps_test <path-to-kitti_10_03_config.yaml> YOUR_DATASET_FOLDER/2011_10_03_drive_0027_sync/
rosrun global_fusion global_fusion_node
```

## VINS-Fusion on car demonstration

Download [car bag](https://drive.google.com/open?id=10t9H1u8pMGDOI6Q2w2uezEq5Ib-Z8tLz) to YOUR_DATASET_FOLDER.
Open four terminals, run vins odometry, visual loop closure(optional), rviz and play the bag file respectively.
Green path is VIO odometry; red path is odometry under visual loop closure.

```sh
roslaunch vins vins_rviz.launch
rosrun vins vins_node <path-to-vi_car.yaml>
(optional) rosrun loop_fusion loop_fusion_node <path-to-vi_car.yaml>
rosbag play YOUR_DATASET_FOLDER/car.bag
```

## Run with your devices

VIO is not only a software algorithm, it heavily relies on hardware quality. For beginners, we recommend you to run VIO with professional equipment, which contains global shutter cameras and hardware synchronization.

### Configuration file

Write a config file for your device. You can take config files of EuRoC and KITTI as the example.

### Camera calibration

VINS-Fusion support several camera models (pinhole, mei, equidistant). You can use [camera model](https://github.com/hengli/camodocal) to calibrate your cameras. We put some example data under /camera_models/calibrationdata to tell you how to calibrate.

```sh
rosrun camera_models Calibrations -w 12 -h 8 -s 80 -i calibrationdata --camera-model pinhole
```

## Docker Support

To further facilitate the building process, we add docker in our code. Docker environment is like a sandbox, thus makes our code environment-independent. To run with docker, first make sure [ros](http://wiki.ros.org/ROS/Installation) and [docker](https://docs.docker.com/install/linux/docker-ce/ubuntu/) are installed on your machine. Then add your account to `docker` group by `sudo usermod -aG docker $YOUR_USER_NAME`. **Relaunch the terminal or logout and re-login if you get `Permission denied` error**, type:
```
cd ~/catkin_ws/src/VINS-Fusion/docker
make build
```
Note that the docker building process may take a while depends on your network and machine. After VINS-Fusion successfully built, you can run vins estimator with script `run.sh`.
Script `run.sh` can take several flags and arguments. Flag `-k` means KITTI, `-l` represents loop fusion, and `-g` stands for global fusion. You can get the usage details by `./run.sh -h`. Here are some examples with this script:
```
# Euroc Monocualr camera + IMU
./run.sh ~/catkin_ws/src/VINS-Fusion/config/euroc/euroc_mono_imu_config.yaml

# Euroc Stereo cameras + IMU with loop fusion
./run.sh -l ~/catkin_ws/src/VINS-Fusion/config/euroc/euroc_mono_imu_config.yaml

# KITTI Odometry (Stereo)
./run.sh -k ~/catkin_ws/src/VINS-Fusion/config/kitti_odom/kitti_config00-02.yaml YOUR_DATASET_FOLDER/sequences/00/

# KITTI Odometry (Stereo) with loop fusion
./run.sh -kl ~/catkin_ws/src/VINS-Fusion/config/kitti_odom/kitti_config00-02.yaml YOUR_DATASET_FOLDER/sequences/00/

#  KITTI GPS Fusion (Stereo + GPS)
./run.sh -kg ~/catkin_ws/src/VINS-Fusion/config/kitti_raw/kitti_10_03_config.yaml YOUR_DATASET_FOLDER/2011_10_03_drive_0027_sync/

```
In Euroc cases, you need open another terminal and play your bag file. If you need modify the code, simply re-run `./run.sh` with proper auguments after your changes.
