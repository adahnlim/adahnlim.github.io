--- 
layout: post
title: 라즈베리파이3+아두이노를 이용한 Home IoT 구축 - Node.js 설치
description: 라즈베리파이3에 Node.js 설치
tags: 라즈베리파이 rasberrypi raspbian node.js
date: 2016-12-20
---

이 포스팅에서는 라즈베리파이3에 웹서버로 사용하는 최신 Node.js를 설치해보자.

- 기존 Node.js 삭제

    GUI로 설치했으면 이미지에 Node.js 0.x 버전이 설치되어있다.  
    이 노드버전을 먼저 삭제해주자.

        # dpkg -l |grep node 
        nodejs
        node..
        node..

        # sudo dpkg -r node... 

- Node.js 설치

    다양한 설치방법이 존재하지만 라즈베리파이에 맞게 빌드된 node를 [node-arm.heroku](http://node-arm.herokuapp.com)에서 배포중이므로 해당 패키지를 이용하여 설치해보자.

        # wget http://node-arm.herokuapp.com/node_latest_armhf.deb
        # sudo dpkg -i node_latest_armhf.deb
        # node -v
        v4.2.1

- Node.js App 올리기

    Node.js를 설치했으니 App을 올려보자.  
    필자는 개인적인 용도로 사용하는 노드(Express)기반 앱이 이미 있어 그 앱에 Home IoT 대시보드를 구성할 예정이다.

        # npm run start
        Express listening on port 3000

    1. 앞으로 만들어갈 Home IoT Dashboard 화면
    ![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/node-1.png?raw=true)

    2. 사이드 메뉴  
    ![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/node-2.png?raw=true)

    3. 기존에 사용중이던 기능들(FTP List & Delete)  
    ![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/node-3.png?raw=true)

    4. 반응형으로 개발되어 모바일&타블렛 환경에도 어울  
    ![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/node-4.png?raw=true)



