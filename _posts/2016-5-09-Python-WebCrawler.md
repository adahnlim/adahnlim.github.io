---	
layout: post
title:  MongoDB ReleaseNote Crawling하기
description: Python을 이용한 Web Crawler(MongoDB ReleaseNote) 
tags: python crawler mongodb
---

Python을 이용한 Web Crawler를 만들때 필요한 HTML Parser는 대표적으로 BeautifulSoup이다.
사용법은 [https://www.crummy.com/software/BeautifulSoup/bs4/doc/](https://www.crummy.com/software/BeautifulSoup/bs4/doc/ "BeautifulSoup이다 공식 홈페이지") 를 참고하자.  
- Installation
  
		# pip install beatifulsoup

- Module Import

		# from bs4 import BeautifulSoup


- Example
	해당 예제는 Python BeautifulSoup을 이용하여 MongoDB ReleaseNote를 가져오는 예제이다.
	[https://github.com/adahnlim/python_webcrawler](https://github.com/adahnlim/python_webcrawler "MongoDB Releasenote Web Crawler")