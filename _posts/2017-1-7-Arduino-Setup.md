--- 
layout: post
title: Home IoT 구축 - 아두이노 연결 및 LED테스트
description: 라즈베리파이3+아두이노 연결 및 LED테스트
tags: 라즈베리파이 raspberrypi raspbian 아두이노 arduino
date: 2017-1-7
---

드디어 알리에서 주문한 아두이노+센서 셋트가 도착하였다.  

- 내용물  
    - 1 X Arduino Uno R3 보드  
    - 1 X USB 케이블  
    - 1 X 점프 케이블  
    - 1 X 830 브레드보드  
    - 5 X LED 빛  
    - 1 갑 저항  
    - 1 X 암수 dupond 라인  
    - 1 X 전위차계  
    - 1 X 부저  
    - 1X74HC595  
    - 1 X 적외선 수신기  
    - 1 X LM35  
    - 1 X 불꽃 센서  
    - 1 X 볼 스위치  
    - 1 X 포토 레지스터  
    - 1 X 키 버튼  
    - 1 X 원격 제어  
    - 1 X 자리 디스플레이 튜브  
    - 1X8*8 도트 매트릭스 모듈  
    - 1 X 1-digit 디스플레이 튜브  
    - 1 X 스테퍼 모터 드라이버 보드  
    - 1 X 스테퍼 모터  
    - 1X9 그램 서보  
    - 1 X IIC 1602 LCD  
    - 1 X XY 조이스틱 모듈  
    - 1 X 모듈  
    - 1 X 물 테스트 모듈  
    - 1 X RFID 모듈  
    - 1 X RFID 키 체인  
    - 1 X RFID 흰색 카드  
    - 1 X 사운드 모듈  
    - 1 X 릴레이 모듈  
    - 1 X 시계 모듈  
    - 1X4*4 키 보드  
    - 1 X RGB 3 컬러 모듈  
    - 1X9 볼트 배터리 스냅   

![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/arduino-6.jpg?raw=true)


- 아두이노 패키지 설치

    1. 먼저 라즈베리파이에 아두이노 패키지를 설치하자.

            #  sudo apt-get install arduino
            ..
            ..
            // 시리얼포트에 엑세스하기 위해 pi계정에 권한을 부여한다.
            # sudo usermod -a -G tty pi
            # sudo usermod -a -G dialout pi 

    2. 다음과 같이 라즈베리파이와 아두이노를 연결한다.  
    ![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/arduino-2.jpg?raw=true)  


    3. 연결 후 라즈베리파이에서 아두이노 포트를 확인하자.

            # dmesg |tail
            [ 92.762419] usb 1-1.4: New USB device found, idVendor=1a86, idProduct=7523
            [ 92.762442] usb 1-1.4: New USB device strings: Mfr=0, Product=2, SerialNumber=0
            [ 92.762456] usb 1-1.4: Product: USB2.0-Serial
            [ 93.819867] usbcore: registered new interface driver usbserial
            [ 93.819982] usbcore: registered new interface driver usbserial_generic
            [ 93.820077] usbserial: USB Serial support registered for generic
            [ 93.822843] usbcore: registered new interface driver ch341
            [ 93.823022] usbserial: USB Serial support registered for ch341-uart
            [ 93.823501] ch341 1-1.4:1.0: ch341-uart converter detected
            [ 93.828088] usb 1-1.4: ch341-uart converter now attached to 
            **ttyUSB0**

- 아두이노 LED 샘플 테스트

    아두이노에서 Hello Adahn이라는 메세지를 라즈베리파이로 전송하고, 라즈베리파이에서는 아두이노LED 깜박임 횟수를 지정해 전송하면 아두이노에서는 전송된 값을 받아 횟수만큼 LED를 깜박하는 테스트이다.

    1. 라즈베리파이에서 시리얼연결로 아두이노와 통신하려면 pyserial 패키지가 필요하다.
    
            # sudo pip install pyserial

    2. 라즈베리파이 python 코드 작성  


            import serial

            // 위에서 확인한 아두이노 연결 포트
            port = "/dev/ttyUSB0"
            serialFromArduino = serial.Serial(port,9600)
            serialFromArduino.flushInput()
            a=1

            while 1:

                // 아두이노에서 보내온 값을 받아 출력한다.
                input = serialFromArduino.readline()
                print input[:-1]

                // 아두이노로 10이라는 메세지를 일회 보낸다.
                if a==1:
                    serialFromArduino.write('10')
                a+=1
    
    3. 아두이노 코드 작성
    
            // 13번 LED 
            const int pinNum=13;

            // 아두이노 코드는 setup()과 loop()함수가 반드시 있어야 한다.
            // setup()에서는 초기 환경 설정 값들이 들어간다.
            void setup()
            {
                // 13번 LED를 OUTPUT로 지정
                pinMode(pinNum,OUTPUT);
                Serial.begin(9600);
            }
            // 주기적으로 실행되는 loop()함수
            void loop()
            {
                // Hello Adahn이라는 메세지를 보낸다.
                 Serial.println("Hello adahn");

                // 라즈베리파이에서 전송된 메세지가 있으면 blink(num) 함수 실행
                 while( Serial.available() > 0)
                 {
                   int num = Serial.parseInt();
                   Serial.print(num);
                    Serial.println(" recived");
                   blink(num);
                 }
                 delay(1000);

            }

            // 전송된 횟수 만큼 LED 깜박임(0.1초 주기)
            void blink(int n)
            {
              for(int i=0;i<n;i++)
              {
               digitalWrite(pinNum,HIGH);
               delay(100);
                digitalWrite(pinNum,LOW);
               delay(100) ;
              }

            }

    4. 컴파일 및 업로드
    
        아두이노IDE를 통해 컴파일 및 업로드를 한다.

        ![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/arduino-7.png?raw=true)  


    5. 확인

        [![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/arduino-8.png?raw=true)](https://www.youtube.com/watch?v=LGYxOTKig40)


