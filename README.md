# 세종대학교 M.S.L.B Capstone
세종대학교 지능기전공학부 문인이동체전공 졸업프로젝트

## 1. 팀명: M.S.L.B (Makes Shopping Life Better)
## 2. 일정: 2023.03.03 ~ 2021.05.26 (약 12주)
## 3. 팀원: 김동영(팀장), 백근주, 박준서, 김대식
## 4. 목표: 실내 자율주행 카트가 특정 인물을 Tracking 및 SLAM을 통한 Localization
## 5. 역할
|역할|main|
|---|---|
|Tracking|김동영, 김대식|
|App|백근주, 박준서|
|H/W|IVL LAB|
## 6. 개발환경
- Ubuntu 18.04    
- ros-melodic-desktop-full    
- nvdia-smi 11.4 CUDA and cudnn   
- cmake-3.23.0    
- opencv-4.2.0    
- tensorflow==2.3.1   
- python 3.7 (Anaconda3-2021.11-Linux-x86_64.sh)    
- scout_mini    
- pcan-driver (peak-linux-driver-8.10.1)  

# Code 사용 방법
## 설치(Installation)
1. ROS  
2. 총 3가지 pkg 설치
3. 경로요약 및 참고 

## 1. ROS 설치 & workspace init
1) [ROS 설치 링크](http://wiki.ros.org/melodic/Installation/Ubuntu)로 이동  
2) ROS Melodic(Ubuntu 18.04 호환 버전) 설치  
3) update까지 마치고 desktop-full 실행  
`$ sudo apt install ros-melodic-desktop-full`  
4) 1.6.1까지 진행  
5) 설치 후 터미널에서 `roscore` 실행으로 정상적으로 설치되었는지 확인  

![roscore](image/roscore.png)  

참고) ROS Melodic에서 Python3를 사용하기 위해서는 아래 명령어 입력 필요  
`$ sudo apt-get install python3-catkin-pkg-modules`  
`$ sudo apt-get install python3-rospkg-modules`

6) 터미널 창에서 아래와 같이 작업 공간(폴더)를 생성한다. (catkin_ws 이외에 다른 폴더 이름을 해도 상관없다.)  
`$ cd ~ && mkdir -p catkin_ws/src`  
`$ cd ~/catkin_ws/src`  
7) workspace init 실시  
`$ catkin_init_workspace`  

## 2. 다양한 PKG 설치    
### 2-1) 가장 먼저 CAN 통신을 위해 scout_mini_ros github code를 clone해야 한다.

Installation
```
cd ~/catkin_ws/src/
git clone https://github.com/roasinc/scout_mini_ros.git  
cd ~/catkin_ws/src/scout_mini_ros/scout_mini_base/lib/
sudo dpkg -i ros-melodic-scout-mini-lib_0.2.0-0bionic_amd64.deb
cd ~/catkn_ws/   
rosdep install --from-paths src --ignore-src -y
catkin_make
```

Upstart 실행 후 재시작 필요

```
rosrun scout_mini_lib install_upstart -r scout_mini
sudo reboot
```

Setting up can for scout mini -> can0 를 터미널로 연결하는 코드    
재시작 이후 실행 -> 오류없이 연결되면 다음으로 넘어가기 (오류시 참고 부분 참조)
```
cd catkin_ws
source devel/setup.bash
cd src/scout_mini_ros
sudo ip link set can0 up type can bitrate 500000
roslaunch scout_mini_base base.launch
```      
예시코드) 위 코드를 작성 후 새로운 터미널을 열어 밑의 코드 작성
```
rostopic pub -1 /cmd_vel geometry_msgs/Twist '{linear: {x: 1.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.0}}’
```
통신이 잘 된다면 앞으로 조금 움직여야 함

------------

### 2-2) 다음으로 Detection & Tracking 부분 설치

Installation
```
cd ~ && mkdir -p wego_ws/src
cd ~/wego_ws/src
catkin_init_workspace
git clone https://github.com/We-Go-Autonomous-driving/main2_one_person.git
cd .. && catkin_make
sudo chmod +x ./*
```

참고) Python 파일을 새로 생성한 후에는 해당 파일의 권한 설정이 필요하다.  
`$ sudo chmod +x (파일이름)`  
또는 모든 파일에 대해서 한 번에 할 때는 아래와 같은 명령어 사용  
`$ sudo chmod +x ./*`  

여기까지 하면 scout-mini를 제어할 수 있는 단계가 된다.  

yolov4-deepsort를 사용하기 위해서는 [yolov4.weights](https://drive.google.com/open?id=1cewMfusmPjYWbrnuJRuKhPMwRe_b9PaT) 를 다운받거나 혹은 [yolov4-tiny.weights](https://github.com/AlexeyAB/darknet/releases/download/darknet_yolo_v4_pre/yolov4-tiny.weights)를 다운받아야 한다. 그리고 `weights`파일을 `scout_bringup/data`경로에 넣어줘야 함.

또한 아래 명령어를 실행해서 darknet weights를 Tensorflow model에 사용할 수 있게 convert해야 함.  
`$ python save_model.py --model yolov4` (yolov4.weights 사용)  
`$ python save_model.py --weights ./data/yolov4-tiny.weights --output ./checkpoints/yolov4-tiny-416 --model yolov4 --tiny` (yolov4-tiny.weights 사용)  

yolov4-deepsort에 대해 더 자세히 알고 싶다면 [여기](https://github.com/theAIGuysCode/yolov4-deepsort) 참고할 것  
scout-mini에 대해 더 자세히 알고 싶다면 [여기](https://github.com/roasinc/scout_mini_ros) 침고할 것  



수정사항들
------------
- utils.py 파일 line 77에 있는 read_class_names 클래스에서 class_file_name 을 './scout_bringup/data/classes/coco.names'로 수정

- object_track_one_person.py에 설정된 경로가 2개 있는데 이를 내 환경에 맞게 수정

- object_track_one_person.py에 초기에 Depth 카메라가 기본으로 있는데 use_webcam = True

- 304번 줄에 not use_webcam and 추가

- 341번 줄 기본 drive4에서 drive1 or 2로 변경  

------------     

### 2-3) 다음으로 SLAM 실행
### RPLiDAR를 사용허기 위한 github code 설치
  
Installation
```
cd ~ && mkdir -p point_ws/src
cd ~/point_ws/src
catkin_init_workspace
git clone https://github.com/robopeak/rplidar_ros
cd .. && catkin_make
sudo chmod +x ./*
```
  
Upstart

```
cd point_ws
source devel/setup.bash
cd src/rplidar_ros
roslaunch rplidar_ros view_rplidar.launch
```
  
### SLAM 알고리즘의 하나인 Hector Slam github code 설치
![MAP](image/rviz_screenshot_2023_05_23-15_55_42.png)  
Installation
```
cd ~/point_ws/src
git clone https://github.com/tu-darmstadt-ros-pkg/hector_slam
cd .. && catkin_make
sudo chmod +x ./*
```

Upstart

```
cd point_ws
source devel/setup.bash
roslaunch rplidar_ros view_slam.launch
```
------------

## 3. 최종 경로(요약)  
wego_ws      
├build  
├devel/setup.bash  
└src  
　└scout_mini_ros  
　　└scout_bringup  
　　　├core  
　　　├data/yolov4.weights(or yolov4-tiny.weights)  
　　　├deep_sort  
　　　├launch  
　　　├model_data  
　　　├outputs  
　　　├scripts  
　　　├tools  
　　　├Default_dist.py --> 깊이 초깃값 측정 (이를 토대로 장애물 영역의 깊이를 측정해 장애물 유무를 판단할 수 있다.)    
　　　├camera.py --> depth camera를 이용할 수 있게 하는 class code  
　　　├convert_tflite.py  
　　　├convert_trt.py  
　　　├drive.py --> 입력 이미지에 대한 주행 알고리즘(depth값과 RGB값이 입력되어 전진/정지/우회전/좌회전/속도감속 등을 정한다)    
　　　├key_move.py --> 추적 & 주행 알고리즘을 거쳐 나온 결과값(string)에 따라 속도와 방향을 변경해주는 메소드    
　　　├object_track_one_person.py --> 입력 이미지에 대한 추적 실시  
　　　├save_model.py  
　　　├scout_motor_light_pub.py --> key_move.py에서 나온 결과를 ROS topic으로 발행하는 코드(모터 및 조명 제어)  
　　　└utils2.py --> 깊이값을 이용해 사람과의 거리 및 장애물 영역 측정

catkin_ws      
├build  
├devel/setup.bash  
└src     
　└scout_mini_ros     
　　├scout_mini_base     
　　├scout_mini_control     
　　├scout_mini_description    
　　├scout_mini_msgs     
　　├scout_mini_navigation     
　　├scout_mini_ros      
　　└READ.md    
  
point_ws      
├build  
├devel/setup.bash   
└src     
　├hector_slam      
　└rplidar_ros     
   
## 4. 사용 방법 (각각 다른 터미널에서 실행)
- 가장먼저 CAN0를 통해 scout_mini랑 연결하기
```
cd catkin_ws
source devel/setup.bash
cd src/scout_mini_ros
sudo ip link set can0 up type can bitrate 500000
roslaunch scout_mini_base base.launch
```   

- `scout_bringup/object_track_one_person.py` 를 rosrun 하면 된다.      
Webcam을 이용한 detection 실행하기 (can통신을 연결 후 실행할 것)
```
cd wego_ws
source devel/setup.bash
cd src/scout_mini_ros
rosrun scout_bringup object_track_one_person.py
```
--> 시작 시 최초1인을 추적하는 코드    

- SLAM
```
cd point_ws
source devel/setup.bash
roslaunch rplidar_ros view_slam.launch
```

## 5. 모듈 파일 설명(scout_bringup 폴더 내에 있음)
1. key_move.py --> 추적 & 주행 알고리즘을 거쳐 나온 결과값(string)에 따라 속도와 방향을 변경해주는 메소드  
2. scout_motor_light_pub.py --> key_move.py에서 나온 결과를 ROS topic으로 발행하는 코드(모터 및 조명 제어)  
3. camera.py --> depth camera를 이용할 수 있게 하는 class code  
4. drive.py --> 입력 이미지에 대한 주행 알고리즘(depth값과 RGB값이 입력되어 전진/정지/우회전/좌회전/속도감속 등을 정한다)  
5. utils2.py --> 깊이값을 이용해 사람과의 거리 및 장애물 영역 측정  
6. Default_dist.py --> 깊이 초깃값 측정 (이를 토대로 장애물 영역의 깊이를 측정해 장애물 유무를 판단할 수 있다.)  
7. object_track_one_person.py --> 입력 이미지에 대한 추적 실시  


**참고**  

- USB 권한주기 (연결이 잘 안될때 시도해보기)
```
ls -l /dev |grep ttyUSB
sudo chmod 666 /dev/ttyUSB0 
```
    
- SLAM map 저장하기
```
cd point_ws
source devel/setup.bash
rosrun map_server map_saver -f ~/map1
```
도움주신 [My cat is Rockstar](https://velog.io/@bbirong/series/Human-Following-Robot)
