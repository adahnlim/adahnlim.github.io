--- 
layout: post
title:  Spark - Naive Bayes 분류 모델
description: Spark MLlib(Naive Bayes) 분류 모델을 통한 성별 예측
tags: spark scala Naive-Bayes classification data-science Mastering-Apache-Spark mllib
--- 

PACKT사의 'Mastering Apache Spark'에서 사용되는 예제들을 중심으로 학습하였다.

학습에 사용된 데이터 또한 책에서 소개한 데이터 [영국정부공공데이터](https://data.gov.uk/dataset/road-accidents-safety-data/resource/a7583887-cbc2-4bb7-be1f-17b3bb5e0e11) 를 이용하였다.

- Naive Bayes 분류 기법
    
    [Wikipia](https://ko.wikipedia.org/wiki/나이브_베이즈_분류)에 나와있는 것 처럼 대상의 특성을 학습하여 확률에 근거하여 대상을 분류한다.

- Source

    [Github 참고](https://github.com/adahnlim/Mastering_Apache_Spark_Exam/blob/master/naive_bayes/src/main/scala/naive.scala)

- Result
    
        Accuracy : 50.73...

- Summary

    정확도가 50%밖에 되지 않으므로 이 데이터는 Naive Bayes 모델로 예측하는 것은 적합하지 않다.
    
