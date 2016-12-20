--- 
layout: post
title: 라즈베리파이3+아두이노를 이용한 Home IoT 구축 프로젝트 - 라즈베리파이 설치
description: 라즈베리파이3에 RASPBIAN 설치 및 설정
tags: 라즈베리파이 rasberrypi raspbian
---

이 포스팅에서는 라즈베리파이3의 OS설치 및 설정을 해보고자 한다.

- OS 이미지 만들기 및 부팅

    microSD(8GB 이상 추천)와 리더기를 미리 준해놓자.

    1. [라즈베리파이 공식홈페이지](https://www.raspberrypi.org/downloads/raspbian/)에 접속하여 OS 이미지를 다운받는다.  
    RASPBIAN JESSIE WITH PIXEL은 GUI와 기타 유틸리티가 설치된 이미지이고 LITE는 기본 이미지이다.
    ![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/rasberry-3.png?raw=true)
    2. 다운로드를 받고 압축을 풀면 .img 파일이 생성된다.
    3. 헤당 이미지를 microSD에 담기 위해서는 유틸리티가 필요하다.  
    (window - [win32diskimager](http://sourceforge.net/projects/win32diskimager))  
    (mac - 기본 diskutil 이용)
    4. 필자는 MAC을 사용중이어서 다음과 같이 작업하였다.
    
            # df -h (microSD 볼룸명 확인)
            # sudo diskutil unmountDisk /dev/disk#
            # sudo dd bs=1m if=image_name.img of=/dev/rdisk#
            # sudo diskutil eject /dev/rdisk#

    5. 배포된 microSD를 라즈베리파이에 꽂는다.
    6. 키보드/마우스/HDMI모니터를 연결한다. (키보드/마우스/모니터가 없어도 설정할 수는 있지만 이방법은 생략하도록 한다.)
    7. 라즈베리파이에 5V전압(핸드폰충전기면 된다.)을 인가하여 라즈베리파이를 킨다.
    
- 환경 설정

    부팅시키고 정상적으로 OS가 올라오면 다음과 같이 GUI(필자는 GUI 이미지로 설치) 또는 터미널 프롬프트가 뜬다.

    1. 로그인(초기 로그인 정보 : pi / raspberry)
    ![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/rasberry-4.png?raw=true)

    2. 초기에는 microSD용량이 다 설정된 것이아니라 이미지 크기정도 사용할 수 있는데 이것을 확장 시켜준다.  
    GUI) Preperence -> Raspbeery Pi Configuration -> Expand Filesystem  
    CLI) sudo raspi-config -> 1. Expand Filesystem

    수행 후에 microSD용량만큼 늘어난 / 볼륨을 볼 수 있다.

        # df -h 
        Filesystem      Size  Used Avail Use% Mounted on
        /dev/root        30G  4.3G   24G  16% /
        devtmpfs        459M     0  459M   0% /dev
        tmpfs           463M     0  463M   0% /dev/shm
        tmpfs           463M  6.3M  457M   2% /run
        tmpfs           5.0M  4.0K  5.0M   1% /run/lock
        tmpfs           463M     0  463M   0% /sys/fs/cgroup
        /dev/mmcblk0p1   63M   21M   43M  34% /boot
        tmpfs            93M     0   93M   0% /run/user/1000
   
   3. 기타 설정
   
        기타 필요한 설정들은 다음과 같다.  
        GUI) Preperence -> Raspbeery Pi Configuration   
        CLI) sudo raspi-config

        1. 기본 패스워드 변경
        2. 언어 및 타임존 변경
        3. SSH 또는 VNC 서비스 및 기타 시리얼장치 실행
        4. 이 외의 나머지는 환경에 맞게 구성하면 된다.
    
    4. 리부팅 진행
    
        설정한 것들을 적용하기 위해 리부팅을 수행한다.

            # sudo reboot

    5. 패키지 업데이트 수행

        현재 설치되어있는 패키지들은 OS이미지가 제작될때의 패키지 버전을 가지고 있으므로 현재 최신본을 유지하기위해 패키지 업데이트를 수행한다.

            # sudo apt-get update 
            # sudo apt-get upgrade



