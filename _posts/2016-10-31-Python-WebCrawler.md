--- 
layout: post
title:  Naver 뉴스 기사 Crawling하기
description: Python을 이용한 Web Crawler(Naver 뉴스 기사) 
tags: python crawler naver-news
---

Python을 이용한 Web Crawler를 만들때 필요한 HTML Parser는 대표적으로 BeautifulSoup이다.
사용법은 [https://www.crummy.com/software/BeautifulSoup/bs4/doc/](https://www.crummy.com/software/BeautifulSoup/bs4/doc/ "BeautifulSoup이다 공식 홈페이지") 를 참고하자.
  
- Installation
  
        # pip install beatifulsoup

- Module Import

        # from bs4 import BeautifulSoup


- Example  
    해당 소스는 Python BeautifulSoup을 이용하여 네이버 뉴스 기사를 가져오는 예제이다.

        사용방법 : python naver_news.py {start_date} {end_date} {query_str}

        ex) python naver_news.py 2016-10-01 2016-10-30 helloworld

    [https://github.com/adahnlim/python_webcrawler/blob/master/naver_news.py](https://github.com/adahnlim/python_webcrawler/blob/master/naver_news.py")