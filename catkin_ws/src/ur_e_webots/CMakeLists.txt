cmake_minimum_required(VERSION 2.8.3)

project(ur_e_webots)

find_package(catkin REQUIRED)

catkin_package()

if (CATKIN_ENABLE_TESTING)
  find_package(roslaunch REQUIRED)
  roslaunch_add_file_check(tests/roslaunch_test.xml)
endif()




#############
## Install ##
#############
install(DIRECTORY launch 
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})


catkin_install_python(PROGRAMS scripts/joint_state_publisher.py
        DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

catkin_install_python(PROGRAMS scripts/trajectory_follower.py
        DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

catkin_install_python(PROGRAMS scripts/universal_robots_ros.py
        DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

catkin_install_python(PROGRAMS scripts/visual_sensor_enable.py
        DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})        


catkin_install_python(PROGRAMS scripts/webots_launcher.py
        DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})        
