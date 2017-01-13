--- 
layout: post
title: Home IoT 구축 - 와이파이 모듈(ESP8266) 테스트
description: 아두이노에 인터넷 세상을 구경시켜주자.
tags: 라즈베리파이 raspberrypi raspbian 아두이노 arduino ESP8266 Wemos
date: 2017-1-12
---

아두이노에서 센서값을 라즈베리파이에 전송하기 위해서는 USB시리얼 연결, 이더넷 연결, 지그비 연결, 블루투스 연결, 와이파이 연결 등 여러가지 방법이 있다. Home IoT 구축시 유선은 제외하고 손쉽게 할 수 있는 와이파이 연결방법을 선택하였다.

그러나 아두이노 R3 UNO보드에서 와이파이연결을 하려면 ESP8266 모듈을 구매해서 회로를 구성해야하는 번거로움이 존재한다. 그러면 회로도 복잡해지고 나중에 제품 패키징할때 신경쓰일 것 같아서 아예 아두이노 호환 보드 중에 ESP8266 모듈이 내장되어있는 보드를 구입하자 해서 여러가지 보다가 선택한 것이 [WeMos D1 보드](https://www.wemos.cc/product/d1.html)이다.

![](https://www.wemos.cc/sites/default/files/2016-09/d1_1_3.jpg)

1. Wemos D1 Setup

  아래 두 사이트를 참고해서 셋업을 진행하였다.

  - [https://blog.rjdlee.com/getting-started-with-wemos-d1-on-mac-osx/](https://blog.rjdlee.com/getting-started-with-wemos-d1-on-mac-osx/)
  - [https://www.wemos.cc/tutorial/get-started-arduino.html](https://www.wemos.cc/tutorial/get-started-arduino.html)

  연결 사진  

  ![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/wemos-2.jpeg?raw=true)


2. WIFI&HTTP Client 모듈

  - ESP8266Wifi.h : WIFI 연결 및 인증
  - ESP8266HTTPClient : HTTP Request & Response

3. 아두이노 소스 코드

  아두이노에서 adahn=apitest, foo=bar 메세지를 보내면 라즈베리파이 웹서버 콘솔에서 받은 메세지 출력&받은 메세지 리턴 하는 코드이다.

        #include <ESP8266HTTPClient.h>
        #include <ESP8266WiFi.h>

        const char* ssid = "your ssid";
        const char* password = "your password";

        void setup() {
          Serial.begin(9600);
          delay(10);

          // Connect to WiFi network
          Serial.println();
          Serial.println();
          Serial.print("Connecting to ");
          Serial.println(ssid);

          WiFi.begin(ssid, password);

          while (WiFi.status() != WL_CONNECTED) {
            delay(500);
            Serial.print(".");
          }
          Serial.println("");
          Serial.println("WiFi connected");


          // Print the IP address
          Serial.print("WIFI IP : ");
          Serial.print(WiFi.localIP());
          Serial.println("");

           HTTPClient http;
          http.begin("http://yourapiserver");
          http.addHeader("Content-Type", "application/x-www-form-urlencoded");

          String postMessage="adahn=apitest&foo=bar";
          int httpCode = http.POST(postMessage);
          Serial.print("http result:");
          Serial.println(httpCode);
          http.writeToStream(&Serial);
          http.end();
        }


        void loop() {
          // Check if a client has connected


        }

4. 결과

  정상적으로 라즈베리파이 웹서버 콘솔과 시리얼모니터에 값이 출력되는 것을 확인 할 수 있다.
![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/wemos-1.png?raw=true)


이제 homeIoT를 구축하기 위한 기본적인 준비는 다 끝났다.  
앞으로 해야할 일은 다음과 같다.

1. WeMos보드에 온/습도, 모션센서 회로를 구성하고 센싱한 값을 라즈베리파이에 전송
2. 라즈베리파이에서 DB화 및 웹서버 개발(이 부분은 계속 진행 중)
3. 알리에서 주문한 마이크로USB전원 케이블(3m)와 보드 케이스가 지금 배송 중이니 보드와 브레드보드를 패키징해서 거실 및 방에 설치

그럼 다음 포스팅은 1번 주제로!

