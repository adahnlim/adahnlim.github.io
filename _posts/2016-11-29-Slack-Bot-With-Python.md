--- 
layout: post
title:  Python을 이용한 Slackbot 만들기
description: Python을 이용해서 Slackbot(구글캘린더 정보 가져오기)을 만들어보자.
tags: Python Slackbot Slacker Google-Calandar GoogleAPI
---

Python을 이용하여 Slackbot을 만들어보자.  
SlackAPI 파이선 인터페이스로 Slacker([https://github.com/os/slacker](https://github.com/os/slacker))를 이용하였다.

- 기능 

    1. 샘플 : 구글 캘린더 일정 보여주기
    2. 기타 원하는 기능을 Customizing하면 된다.

- 구글 캘린더 API이용하기
    
    구글 캘린더를 가져오기 위해서는 캘린더API를 이용해야 한다. [https://developers.google.com/google-apps/calendar/quickstart/dotnet](https://developers.google.com/google-apps/calendar/quickstart/dotnet)를 참고하여 원하는 언어에 대한 샘플을 참고하자.
 
- Slack Bot 만들기

    1. https://YOUR_SLACK_URL.slack.com/apps 접속
    2. 우상단 Build 클릭
    3. 다른팀 또는 나의팀에 대한 앱을 만들것인지 선
    4. Bots 선택
    5. Bot 이름 입력
    6. API-Token 보관 및 기타설정(아이콘, 이름 등)하기

- Source
    
    [https://github.com/adahnlim/slackbot](https://github.com/adahnlim/slackbot)

- Image

    ![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/slack-1.png?raw=true)