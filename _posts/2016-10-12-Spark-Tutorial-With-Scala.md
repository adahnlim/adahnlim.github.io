---	
layout: post
title:  Spark Tutorial With Scala
description: Spark Tutorial With Scala
tags: spark scala sbt sbt-package spark-tutorial spark-install scala-dependency
---	

Spark홈페이지에 보면 따라할 수 있는 예제가 있다.
[http://spark.apache.org/docs/latest/quick-start.html](http://spark.apache.org/docs/latest/quick-start.html "Apache Spark")  

- spark 설치

		# wget http://d3kbcqa49mib13.cloudfront.net/spark-2.0.1-bin-hadoop2.7.tgz
		# tar xvfz  spark-2.0.1-bin-hadoop2.7.tgz
		# vi .bashrc
			PATH=$PATH:YOUR_SPARK_HOME/bin


예제를 따라하기 전에 Scala Build환경을 만들어주어야 하는데, 가장 많이 쓰이는 sbt tool을 먼저 설치해야 한다.

- sbt 설치 [http://www.scala-sbt.org/download.html](http://www.scala-sbt.org/download.html)


		# wget https://repo.typesafe.com/typesafe/ivy-releases/org.scala-sbt/sbt-launch/0.13.12/sbt-launch.jar

		# mv sbt-launch.jar ~/bin
		# cd ~/bin
		# vi sbt

		!/bin/bash
		SBT_OPTS="-Xms512M -Xmx1536M -Xss1M -XX:+CMSClassUnloadingEnabled -XX:MaxPermSize=256M"
		java $SBT_OPTS -jar `dirname $0`/sbt-launch.jar "$@"

		# chmod u+x sbt 

		# ./sbt
		[info] Set current project to bin (in build file:~/bin/)
		> 
		
		이렇게 나오면 성공!


- scala directory 구조
	
	- workdir/prj-example
		+ src - main - scala - *.scala            
			* scala source 디렉토리
		+ target - scala-x.x.x - *.jar, classes
			* sbt package 실행시 .jar로 빌드되는 디렉토리
		+ build.sbt
			* sbt build 환경 설정 파일

- 튜토리얼 예제 

		# vi src/main/scala/SimpleApp.scala

		/* SimpleApp.scala */
		import org.apache.spark.SparkContext
		import org.apache.spark.SparkContext._
		import org.apache.spark.SparkConf

		object SimpleApp {
		  def main(args: Array[String]) {
		    val logFile = "README.md" // file:/// hdfs:/// ...
		    val conf = new SparkConf().setAppName("Simple Application")
		    val sc = new SparkContext(conf)
		    val logData = sc.textFile(logFile, 2).cache()
		    val numAs = logData.filter(line => line.contains("a")).count()
		    val numBs = logData.filter(line => line.contains("b")).count()
		    println("Lines with a: %s, Lines with b: %s".format(numAs, numBs))
		  }
		}

		# build.sbt

		name := "Simple Project"

		version := "1.0"

		scalaVersion := "2.11.7"

		libraryDependencies += "org.apache.spark" %% "spark-core" % "2.0.1"

- 튜토리얼 예제 빌드

		# sbt package
		..
		[success] Total time: 1 s, completed 2016. 10. 12 오후 2:23:49

		# ls target/scala-2.11/
		classes				simple-project_2.11-1.0.jar

		// master는 local, local[n], yarn 등, 기타 spark-submit 옵션은 자신의 환경에 맞게 구성한다.
		# YOUR_SPARK_HOME/bin/spark-submit --master local target/scala-2.11/simple-project_2.11-1.0.jar
		.
		.
		.
		Lines with a: 20, Lines with b: 10
		.
		.
		.


- Tip & Trick
	
	서버가 온라인 환경(인터넷 가능)일 경우에는 sbt(ivy)가 dependency library를 관리해주지만 대부분 오프라인 환경이 많을 것이다. 
	그래서 sbt를 실행하면 각종 dependency가 해결되지 않았다고 에러메세지를 주욱 출력할 것이다.

		[error] unresolved dependency : org.apache.spark ~~~~

	이 때에는 spark 기본 library(YOUR_SPARK_HOME/jars)를 프로젝트 홈/lib에 링크를 걸어주고 build.sbt의 libraryDependencies를 삭제해준다.

		# ln -s YOUR_SPARK_HOME/jars prj-example/lib
	그러면 프로젝트홈/lib에 있는 jar를 참조해서 패키징한다. 
	그 이외의 라이브러리가 필요할 경우 lib에 .jar파일을 수동(인터넷에서 찾아 복사)으로 추가하여 패키징한다.

