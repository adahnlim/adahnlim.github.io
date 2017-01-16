--- 
layout: post
title: Home IoT 구축 - LCD(1602 I2C)
description: 아두이노에 LCD를 연결해보자.
tags: 라즈베리파이 raspberrypi raspbian 아두이노 arduino 1602 I2C
date: 2017-1-8
---


- 16*2 LCD 연결  

    1602 LCD는 16*2 LCD이며, 16핀이다보니 핀연결을 불편함을 줄이고자 1602 I2C(Inter-Interfrated Circuit)모듈(4핀)이 같이 나온 제품이다.

    연결 회로도는 아래와 같다.  

    ![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/arduino-12.png?raw=true)


- LCD에 메세지 출력하기

    아두이노에서 LCD로 메세지를 출력해보자.

    1. 라이브러리(LiquidCrystal_I2C) 설치
    
      아두이노IDE에 설치된 LiquidCrystal_I2C라이브러리는 구버전이므로, 최신버전의 IDE를 사용하면 라이브러리를 설치해야한다.

        1. https://bitbucket.org/fmalpartida/new-liquidcrystal/downloads에서 버전(v1.2.1) 다운로드
        2. 아두이노IDE -> Sketch -> Import Library -> Add Library 를 선택해 다운로드 받은 파일을 선택

    ![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/arduino-13.png?raw=true)

        3. 아두이노 IDE 재시작

    2. 아두이노 코드 작성
    
            #include <Wire.h>
            #include <LiquidCrystal_I2C.h>

            // LCD Address(0x3F, 보통은 0x27)
            LiquidCrystal_I2C lcd(0x3F, 2, 1, 0, 4, 5, 6, 7, 3, POSITIVE);  

            void setup()
            {


              lcd.begin(16,2);

              // 3번 백라이트 온오프
              for(int i = 0; i< 3; i++)
              {
                lcd.backlight();
                delay(250);
                lcd.noBacklight();
                delay(250);
              }
              lcd.backlight();

              // 1번째 줄에 Hello, Adahn! 출력
              lcd.setCursor(0,0);
              lcd.print("Hello, Adahn!");

              // 1초 뒤에 2번째ㅈ 줄에 I'M LCD!!! 출력
              delay(1000);
              lcd.setCursor(0,1);
              lcd.print("I'm LCD!!!");
              delay(8000);


            }

            void loop()
            {

            }

    3. 컴파일 및 업로드
    
        아두이노IDE를 통해 컴파일 및 업로드를 한다.

    4. 확인

          [![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/arduino-14.png?raw=true)](https://www.youtube.com/watch?v=nfTcqlJBAHs
          )

        


