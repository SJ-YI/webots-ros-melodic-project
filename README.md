# Webots World (ongoing)

[실행환경]
* Ubuntu 18.04 
* ROS Melodic  
* Nvidia GPU driver (CUDA 11.2가 가능한 GPU 디바이스)



### 로컬 PC에 Webots 설치하기 
(1) 로컬 PC에 [ROS Melodic](http://wiki.ros.org/melodic)이 설치 된 것으로 간주한다 <br/>
(2) [Installation Procedure](https://cyberbotics.com/doc/guide/installation-procedure#installation-procedure)을 참고해서 [webots을 설치](https://cyberbotics.com/)한다 <br/>
(3) [universal_robots](http://wiki.ros.org/universal_robots) 패키지를 ROS 버전에 맞게 설치한다 <br/>
(4) [WEBOTS_HOME](https://cyberbotics.com/doc/guide/tutorial-8-using-ros) 환경변수 할당<br/>
(5) Webots [Environment Variables](https://www.cyberbotics.com/doc/guide/running-extern-robot-controllers?tab-os=linux&tab-language=python#environment-variables) 할당 (webots 함수 패키지 추가하기)
  ```bash
  # 로컬 PC의 ~/.bashrc 에서 
  export WEBOTS_HOME=/usr/local/webots
  export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$WEBOTS_HOME/lib/controller
  export PYTHONPATH=$PYTHONPATH:$WEBOTS_HOME/lib/controller/python27
  ```
(6) 테스트: [UR 로봇 프로젝트 파일을 catkin_ws로 옮기고 rviz에 띄우기](https://cyberbotics.com/doc/guide/ure)

***

### Webots World 설치하기 
#### 1. Install Docker and Nvidia-docker 
* [docker.com](https://docs.docker.com/engine/install/ubuntu/)으로 가서 자신의 컴퓨터 환경에 맞는 Docker Engine을 설치한다 
* 루트(root) 권한 설정; ```sudo``` 없이 도커 사용하기 
  ``` bash
  ~$ sudo usermod -aG docker $USER #현재 접속중인 사용자에게 권한주기 
  ```
* [docker.com/compose](https://docs.docker.com/compose/install/)에서 Docker Compose를 설치한다 
* [nvidia.com](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker)를 참고해서 Nvidia-docker를 설치한다 
  * 단, 사전에 로컬 PC에서 ```~$ watch nvidia-smi``` 명령어가 가능해야한다. 
  * 위의 명령어가 안 된다면 본인의 환경에 맞게 GPU 드라이버를 설치해야 한다 


#### 2. Pull and Install the docker image 
```bash
~$ sudo chmod a+x make_webots-ros_container.bash run_webots-ros_container.bash  # 실행 권한 부여 

~$ ./make_webots-ros_container.bash  # 도커 이미지 설치 및 컨테이너 생성 

~$ ./run_webots-ros_container.bash

```


<br/>

* 위의 명령어가 실행되면 ```webots_melodic``` 이란 이름으로 도커 컨테이너가 생성된다 
* 해당 컨테이너를 다시 실행하고 싶다면 로컬 PC의 터미널에서 ```~$ ./run_webots-ros_container.bash``` 명령을 다시 실행한다 


- [x] ```--net```, ```--ipc``` 붙이니까 rviz가 안되네... 이부분 해결 필요 
  * 도커 컨테이너에서 ```ROS_IP```, ```ROS_HOSTNAME```, ```ROS_MASTER_URI``` 환경 변수 값만 잘 설정하면 굳이 해당 옵션을 줄 필요가 없음 [(참고 링크)](https://www.ybliu.com/2020/05/ros-remote-debgging-and-communication.html)

- 도커 환경에서 ROS network를 다음과 같이 설정한다: 
```bash 
# 도커의 ~/.bashrc 에서 

export ROS_IP=172.17.0.2  # in the docker container for webots world
export ROS_HOSTNAME=$ROS_IP
export ROS_ROS_MASTER_URI=http://172.17.0.1:11311   # IP of your local PC for ROS Master 
```
- [x] 로컬 PC의 ROS와 도커 컨테이너 상의 ROS가 서로 같은 roscore에 연결되도록 수정  
  

#### 3. Run the webots world and Connect with ROS (여기서 부터는 도커 환경에서...)
* 도커 환경에서 아래 명령어를 실행한다
* 그러면 대회에 쓰일 webots world가 열린다 
```bash
~$ ~/run_webots_world.bash
```

- [x] webots world 에서 센서 데이터(RGB, depth_map)를 ros topic/service 정보 받기 
- [x] 추후 ```roslaunch```로 명령어를 간소화 시키자 


***
### 로컬 PC에서 Webots World 제어하기 


#### 4. 영상 데이터 Subscribe 테스트 
* 여기까지 수행했다면 webots world 에서 ROS Master로 RGB 카메라 및 Range Finder 데이터 토픽(topic)을 발행한다 
* 아래 토픽 버스 이름을 통해 영상 정보를 확인할 수 있다 
```bash 
# RGB image topic 
/CAM/camera/image

# Depth image topic 
/CAM/range_finder/range_image
```

##### ※ Camera world → Robot world 변환 행렬 구하기 


* [3D Rotation Converter 계산기](https://www.andre-gaschler.com/rotationconverter/)




#### 5. UR10 로봇 제어
* 자세한 UR10 로봇 제어과 관련된 내용은 [메뉴얼](https://cyberbotics.com/doc/guide/ure)을 참고한다. 
* webots 시뮬레이션 상의 UR10e 로봇을 제어하기 위해서는 webots 상의 controller를 ```<extern>``` 값으로 할당해야 한다. 
* 제어 코드는 [ROS 통신으로 센싱 값을 주고 받으면서](https://github.com/cyberbotics/webots_ros/blob/master/scripts/ros_controller.py) 처리된 제어 신호를 [webots world의 로봇 객체에 할당하는](https://github.com/cyberbotics/webots_ros/blob/master/scripts/ros_python.py) 방식으로 코딩한다 

##### MoveIt 활용 예시 

  ```bash  
  # MoveIt 연동 패키지 예시 
  ~$ roslaunch ur10_e_moveit_config ur10_e_moveit_planning_execution.launch
  
  ~$ roslaunch ur10_e_moveit_config moveit_rviz.launch config:=true   
  ```
* 여기까지 실행이 끝났다면 webots의 가상환경이 ROS rviz에 연동된 것을 확인할 수 있다 
* 콘솔 명령을 통해 MoveIt에 간단한 명령을 입력하는 인터페이스 튜토리얼을 보고 싶다면, 다시 새로운 터미널을 켜고 해당 도커 컨테이너를 실행한 뒤 다음 ROS 패키지를 실행한다: 
  ```bash
  ~$ rosrun moveit_tutorials move_webots_ur_e_python_interface.py  
  ```
  * ```x, y, z``` 좌표를 입력하면 moveit 상의 머니퓰레이터가 움직이면서 webots 가상환경과 연동된 것을 확인할 수 있다 
  * (ex) ```0.8, -0.3, 0.8``` 

##### 직접 제어 방식 ?
* [webots_ros를 통한 컨트롤러 예시](https://github.com/cyberbotics/webots_ros/tree/master/scripts) 참고 

##### Grippers 
* [다음 링크](https://cyberbotics.com/doc/guide/gripper-actuators#robotiq-3f-gripper) 참고
* [예시 코드](https://github.com/cyberbotics/webots/blob/master/projects/robots/universal_robots/controllers/ure_can_grasper/ure_can_grasper.c) 참고 
```python
"""ure_grasper_tutorial controller."""

# You may need to import some classes of the controller module. Ex:
#  from controller import Robot, Motor, DistanceSensor
# 
# (refer to this code): https://github.com/cyberbotics/webots/blob/master/projects/robots/universal_robots/controllers/ure_can_grasper/ure_can_grasper.c


from controller import Robot
from controller import DistanceSensor #(ref) https://cyberbotics.com/doc/reference/distancesensor?tab-language=python
from controller import Motor # (ref) https://cyberbotics.com/doc/reference/motor?tab-language=python
from controller import PositionSensor # (ref) https://cyberbotics.com/doc/reference/positionsensor?tab-language=python 

counter = 0 


target_positions = [-1.88, -2.14, -2.38, -1.51]
speed = 0.5 


# create the Robot instance.
robot = Robot()

# get the time step of the current world.
timestep = int(robot.getBasicTimeStep())



# You should insert a getDevice-like function in order to get the
# instance of a device of the robot. Something like:
#  motor = robot.getDevice('motorname')
#  ds = robot.getDevice('dsname')
#  ds.enable(timestep)
hand_motor_joints = ["finger_1_joint_1", "finger_2_joint_1", "finger_middle_joint_1"]
hand_motors = [] 
for hand_motor in hand_motor_joints :
    """ (ref) https://cyberbotics.com/doc/guide/gripper-actuators#robotiq-3f-gripper
    """
    hand_motors.append(robot.getDevice(hand_motor))  

    

ur_motor_joints = ["shoulder_lift_joint", "elbow_joint", "wrist_1_joint", "wrist_2_joint"]
ur_motors = [] 
for ur_motor in ur_motor_joints:
    """(ref) https://cyberbotics.com/doc/guide/ure?tab-language=python
    """
    ur_motors.append(robot.getDevice(ur_motor))
    hand_motors[-1].setVelocity(speed) #(ref) https://cyberbotics.com/doc/reference/motor?tab-language=python#motor-functions


distance_sensor = robot.getDevice("distance sensor")
distance_sensor.enable(timestep) # (ref) https://cyberbotics.com/doc/reference/distancesensor?tab-language=c#distancesensor-functions

position_sensor = robot.getDevice("wrist_1_joint_sensor")
position_sensor.enable(timestep) # (ref) https://cyberbotics.com/doc/reference/positionsensor?tab-language=python#positionsensor-functions





# Main loop:
# - perform simulation steps until Webots is stopping the controller

state = 'WAITING' # init state  
print(state)

while robot.step(timestep) != -1:
    if counter <= 0 : 
        # Read the sensors:
        # Enter here functions to read sensor data, like:
        #  val = ds.getValue()
        if state == 'WAITING':
            if (distance_sensor.getValue() < 500 ): #(ref) https://cyberbotics.com/doc/reference/distancesensor?tab-language=python#distancesensor-functions
                state = "GRASPING"
                counter = 8 

                print("Grasping can \n")                
                for i in range(len(hand_motor_joints)): 
                    hand_motors[i].setPosition(0.85) # (ref) https://cyberbotics.com/doc/reference/motor?tab-language=python#motor-functions
        
        elif state == 'GRASPING':
            for j in range(len(ur_motor_joints)):
                ur_motors[j].setPosition(target_positions[j])

            print("Rotating arm\n")
            state = "ROTATING"


        elif state == "ROTATING":
            if (position_sensor.getValue() < -2.3): # (ref) https://cyberbotics.com/doc/reference/positionsensor?tab-language=python#positionsensor-functions
                counter = 8
                print("Releasing object\n")
                state = "RELEASING"
                for k in range(len(hand_motor_joints)):
                    hand_motors[k].setPosition(hand_motors[k].getMinPosition()) # (ref) https://cyberbotics.com/doc/reference/motor?tab-language=python#motor-functions


        elif state == "RELEASING":
            for x in range(len(ur_motor_joints)):                      
                ur_motors[x].setPosition(0.0)

            print("Rotating arm back \n") 
            state = "ROTATING_BACK"

        elif "ROTATING_BACK":
            if (position_sensor.getValue() > -0.1):
                state = 'WAITING'
                print("Waiting object \n")


    counter -= 1
                
                            
# Enterhere exit cleanup code.



```


- [x] 로봇 핸드의 pick-up 기능 코드 추가 


#### 6. 기타 
* 그 이외 webots과 관련된 내용은 [webots_ros github](https://github.com/cyberbotics/webots_ros) 및 [ros.org의 webots 위키](http://wiki.ros.org/webots) 참고. 



***

#### Reference 
* [ROS](http://wiki.ros.org/Distributions) & [webots](https://cyberbotics.com/) 설치 
* [universal_robots](http://wiki.ros.org/universal_robots) 설치 
* [WEBOTS_HOME](https://cyberbotics.com/doc/guide/tutorial-8-using-ros) 환경변수 설정 
* [UR 로봇 프로젝트 파일을 catkin_ws로 옮기고](https://cyberbotics.com/doc/guide/ure) rviz에 띄우기 
* webots의  URe 로봇 모델을 ROS로 보내기 위해서는 → webots 월드에서 URe의 컨트롤러를 ```extern```

