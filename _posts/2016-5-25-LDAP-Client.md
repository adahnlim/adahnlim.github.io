---
layout: post
title:  Ubuntu에서 LDAP Client 구성 하기
description: Ubnutu LDAP Client
tags: ldap-client

---


1. LDAP Client 설치

		# sudo apt install libnss-ldapd

	- LDAP서버 URI 입력  
	![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/ldap-11.PNG?raw=true)

	- LDAP서버 baseDN 입력  
	![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/ldap-12.PNG?raw=true)

	- LDAP에서 이용할 정보 선택 (group,hosts,networks,passwd,shadow)
	![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/ldap-13.PNG?raw=true)


2. LDAP Client 재시작

		# /etc/init.d/nscds top
		# /etc/init.d/nslcd start

3. 확인

		# getent passwd
		..
		john:x:10000:5000:John Doe:/home/joh:/bin/bash

		# getent group
		..
		miners:*:5000:

		서버에서 등록된 User와 Group이 보인다.

		# ssh john@localhost
		passoword : 
		..
		john@client01:/$  
		성공적으로 LDAP User로 로그인이 되었다.
