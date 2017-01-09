--- 
layout: post
title: Home IoT 구축 - 온/습도를 LCD에 표시하자
description: 아두이노에 온/습도계와 LCD를 연결하여 표시하자.
tags: 라즈베리파이 raspberrypi raspbian 아두이노 arduino 1602 I2C DHT11
date: 2017-1-8
---


1. DHT11 및 16*2 LCD 연결  

    
    연결 회로도는 아래와 같다.  

    ![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/arduino-10.png?raw=true)

    실제 연결 사진   

    ![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/arduino-4.jpg?raw=true)


2. LCD에 온/솝도 출력하기

    1. 아두이노 코드 작성
    
            #include <Wire.h>
            #include <LiquidCrystal_I2C.h>
            #include <DHT11.h>

            int pin=2;
            DHT11 dht11(pin);

            LiquidCrystal_I2C lcd(0x3F, 2, 1, 0, 4, 5, 6, 7, 3, POSITIVE);  // Set the LCD I2C address

            void setup()
            {
              lcd.begin(16,2);
              lcd.backlight();
            }

            void loop()
            {
                int err;
              float temp,humi;

              if((err = (dht11.read(humi,temp))==0))
              {
                lcd.setCursor(0,0);
                lcd.print("temp = ");
                lcd.print(temp);
                lcd.setCursor(0,1);
                lcd.print("humi = ");
                lcd.print(humi);


              }
              else
              {

              }
              delay(DHT11_RETRY_DELAY);
            }

    3. 컴파일 및 업로드
    
        아두이노IDE를 통해 컴파일 및 업로드를 한다.

    4. 확인

      ![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/arduino-5.jpg?raw=true)
          
        


