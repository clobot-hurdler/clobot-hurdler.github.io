---
layout: blog
title: ROS 디버깅 개발환경 설정 
category: blog
tags: [ROS, debugging, gdb, dump file]   
summary: ros 플랫폼 디버깅 환경설정 
image: /images/blog/ros_devel_env/ros_devel_env.png
user_img: /images/users/brian.png
user_name: 브라이언
comments: true
---

## ROS 디버깅 개발환경 설정
 
## core dump 파일 생성 셋팅
ros 플랫폼을 이용한 개발이 아니더라도, 리눅스환경에서의 디버깅은 윈도우에 비해 쉽지 않다. 
보통 자신이 개발하거나 혹은 남이 개발한 실행파일에서 crash가 나면, segmentation fault (core dumped) 라는 메세지가 뜬다. 
core파일을 찾을수 없다면 우선 다음 명령어를 통해 코어파일 사이즈를 확인한다. 

- ulimit -a

![1](/images/blog/ros_devel_env/1.png)

그림에서 빨간색 선으로 표시한 것처럼 core파일의 크기가 0으로 설정되어 있으면 core파일이 생성되지 않는다. 
다음 명령어를 통하면 코어파일의 크기를 무한대로 설정할 수 있다. 

- ulimit -c unlimited

![2](/images/blog/ros_devel_env/2.png)

하지만 시스템을 reboot하면 다시 설정값이 초기화 되기 때문에 영구적으로 설정하기 위해서는 **limits.conf을 수정**해야한다. 

 - vi /etc/security/limits.conf
 - 빨간색 박스로 표시된 부분에서 주석을(#) 지우고 **core가 명시된 부분의 value 값을 0에서 -1**로 변경   

![3](/images/blog/ros_devel_env/3.png)

현재까지 설정한 상태로 core파일이 생성되면 core라는 이름으로 생성되는데, ros플랫폼 기반에서는 다수의 프로세스를 실행시키기 때문에, 
필자는 생성되는 코어파일의 pid와 실행명을 core파일로 생성하도록 설정하였다. 이를 위해 **sysctl.conf 파일 수정**한다. 

- vi /etc/sysctl.conf
- kernel.core_uses_pid = 1  추가
- kernel.core_pattern = core.%e 추가 

저장 후 다음 명령어를 통해 적용한다. 
 
 - sysctl -p 


GDB를 통해 분석할 core파일을 만들기 위해선 다음 두 가지 옵션 중에서 하나를 선택해서 빌드하면 된다. 

- catkin_make -DCMAKE_BUILD_TYPE=RelWithDebInfo     --> 배포 목적의 빌드지만, 디버깅 정보 포함
- catkin_make -DCMAKE_BUILD_TYPE=Debug              --> 디버깅 목적의 빌드