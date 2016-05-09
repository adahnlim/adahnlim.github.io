---	
layout: post
title:  Docker Engine을 설치해보자.
description: Docker Engine 설치 
tags: docker 
---

Docker는 Linux Container기반으로 이미지 생성/배포/관리 등에 초점을 맞춘 소프트웨어입니다.

- Installation

	1. https / ca certificate 인증을 위한 패키지 설치
	  
			# apt-get update  
			# apt-get install apt-transport-https ca-certificates

	2. Docker Repo 등록
	
			# vi /etc/apt/sources.list.d/docker.list
				# On Ubuntu Precise 12.04 (LTS)
				deb https://apt.dockerproject.org/repo ubuntu-precise main
				# On Ubuntu Trusty 14.04 (LTS)
				deb https://apt.dockerproject.org/repo ubuntu-trusty main
				# Ubuntu Wily 15.10
				deb https://apt.dockerproject.org/repo ubuntu-wily main
		
			# apt-get update

	3. 기존에 Docker repo 있으면 제거
	
			# apt-get purge lxc-docker

	4. Repo 확인
	
			# apt-cache policy docker-engine
	
	5. aufsstorage driver (For Ubuntu Trusty and Wily version)
	
			# apt-get install linux-image-extra-$(uname -r)
	
	6. Docker Install
	
			# apt-get install docker-engine

	7. Docker 설치 확인

			# docker -v
			Docker version 1.11.0, build 4dc5990	
