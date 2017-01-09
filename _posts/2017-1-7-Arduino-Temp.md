--- 
layout: post
title: Home IoT 구축 - 온습도 센서(DHT11)
description: 아두이노에 온습도 센서를 연결해보자.
tags: 라즈베리파이 raspberrypi raspbian 아두이노 arduino DHT11 
date: 2017-1-7
---

이제 본격적으로 센서를 달아 아두이노를 가지고 놀아보자.

1. 온습도 센서(DHT11) 연결

    DHT-11은 저렴하게 온/습도를 같이 측정할 수 있는 센서이다. 알리에서 약 1달러 내외에 구입 가능한다.  
    연결 회로도는 아래와 같다.
    ![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/arduino-9.png?raw=true)

    실제 연결 사진  
    ![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/arduino-3.jpg?raw=true)

2. 온습도 값 확인

    아두이노에서 센서를 통해 받아온 값을 라즈베리파이로 전송한다.

    1. 라즈베리파이 python 코드 작성  

            import serial

            port = "/dev/ttyUSB0"
            serialFromArduino = serial.Serial(port,9600)
            serialFromArduino.flushInput()
            a=1
            while 1:
                input = serialFromArduino.readline()
                print input[:-1]
    

    2. 아두이노 코드 작성
    
            #include "DHT11.h"
            // 디지털 2번에 연결
            int pin=2;
            DHT11 dht11(pin);

            void setup()
            {
               Serial.begin(9600);
            }

            void loop()
            {
              int err;
              float temp,humi;

                // 에러가 발생하지 않으면 값을 발생하면 에러값을 출력
              if((err = (dht11.read(humi,temp))==0))
              {
                 Serial.print("temperature : ");
                 Serial.print(temp);
                 Serial.print(" ");
                 Serial.print("humidity : ");
                 Serial.print(humi);

                 Serial.println();

              }
              else
              {
                 Serial.print("err : ");
                 Serial.print(err);
                 Serial.println();
              }
              delay(DHT11_RETRY_DELAY);

            }

    3. 컴파일 및 업로드
    
        아두이노IDE를 통해 컴파일 및 업로드를 한다.

        ![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/arduino-7.png?raw=true)  


    4. 확인

            # python tempHumi.py
            temperature : 20 humidity: 48
            temperature : 20 humidity: 49
            temperature : 20 humidity: 48
            temperature : 20 humidity: 48
            temperature : 20 humidity: 47
            temperature : 20 humidity: 47
            .
            .

        


