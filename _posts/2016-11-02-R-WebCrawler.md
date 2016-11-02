--- 
layout: post
title:  Naver 뉴스 기사 Crawling하기 + 시각화 in R
description: R을 이용한 Web Crawler(Naver 뉴스 기사) & ggplot으로 시각화 하기 
tags: R r web-crawler ggplot 
---

R 이용한 Web Crawler를 만들때 필요한 HTML Parser로 rvest를 이용하였다. 

사용법은 [https://cran.r-project.org/web/packages/rvest/rvest.pdf](https://cran.r-project.org/web/packages/rvest/rvest.pdf) 를 참고하자.

시각화는 ggplot2 + ggiraph extension을 이용하여 기사 추이를 시각화하였고 tooltip으로 기사 제목을 보이게 하였다.

- Source
    
    [https://github.com/adahnlim/webcrawler/blob/master/naver_news.R](https://github.com/adahnlim/webcrawler/blob/master/naver_news.R")