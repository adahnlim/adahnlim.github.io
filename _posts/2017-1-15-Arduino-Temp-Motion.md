--- 
layout: post
title: Home IoT 구축 - WeMos D1 + 모션센서(HC-SR505) + 온습도센서(DHT11)
description: 아두이노(WeMos)에 모션센서와 온습도센서를 달아보자.
tags: 라즈베리파이 raspberrypi raspbian 아두이노 arduino ESP8266 Wemos HC-SR505 DHT11
date: 2017-1-15
---

이제 1단계 프로젝트(각 방에 모션센서와 온/습도현황 체크)가 거의 마무리 되어간다.  

오늘은 실제 패키징에 앞서 생각했던데로 동작하는지 목업 테스트를 진행하였다.

1. 회로 연결 

  - 온습도계 : D5 + 5V + GND
  - 모션센서 : D4 + 5v + GND

![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/motion1.jpg?raw=true)

2. 아두이노 소스 코드

  온/습도 및 모션센싱을 하여 2초마다 상태를 출력한다.
  

       #include "DHT11.h"

      // pin 5 is DHT11
      // pin 4 is HCSR505
       int pin=5;
       DHT11 dht11(pin);

      void setup()  {
      Serial.begin(9600);
       pinMode(4,INPUT);
       digitalWrite(4,LOW); }

      void loop()  {

      if(digitalRead(4)==HIGH)
       {
       Serial.println("Somebody is here.");
      }
       else  {
       Serial.println("Nobody.");
      }

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

      delay(2000);

       }

4. 결과

  정상적으로 시리얼모니터에 값이 출력되는 것을 확인 할 수 있다.

![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/motion2.png?raw=true)

이제 라즈베리파이에서 데이터를 받을 수 있게 웹서버 코드개발을 진행하면 된다.

~~1. WeMos보드에 온/습도, 모션센서 회로를 구성하고 센싱 값 확인 및 데이터 전송~~  
2. 라즈베리파이에서 DB화 및 웹서버 개발(이 부분은 계속 진행 중)  
3. 알리에서 주문한 마이크로USB전원 케이블(3m)와 보드 케이스가 지금 배송 중이니 보드와 브레드보드를 패키징해서 거실 및 방에 설치  

그럼 다음 포스팅은 3번 주제로!(웹서버 개발은 별도로 포스팅 계획이 없습니다.)

