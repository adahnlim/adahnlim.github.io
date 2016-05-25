---
layout: post
title:  Ubuntu에서 LDAP 서버 구성하기
description: Ubnutu LDAP 서버구성
tags: ldap

---


LDAP(Lightweight Directory Access Protocol)은 중앙에서 인증체계를 관리할 수 있는 디렉토리 프로토콜입니다.  

1. LDAP 서버 설치


		# vi /etc/hosts
		# dc=example,dc=com 
		127.0.1.1       hostname.example.com	hostname

		# sudo apt install slapd ldap-utils
		# vi /etc/ldap/ldap.conf 
		..
		BASE    dc=example,dc=com
		URI     ldap://hostname.example.com or ldap://ldapip
		..
		# dpkg-reconfigure slapd

![dd](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/ldap-1.PNG?raw=true)