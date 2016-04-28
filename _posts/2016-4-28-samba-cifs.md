---
layout: post
title: samba, cifs를 이용하여 Window공유폴더를 마운트하자
tags: samba cifs
---



1. 패키지 설치  
		 - CentOS or RHEL :  # yum samba-client cifs-utils  
	  	 - Ubnutu : # apt-get install sambaclient cifs-utils
			 
2. 공유 폴더 확인하기

		# smbclient -L 192.168.0.1 -Usamba -W workgroup
		Enter samba's password: 
		Domain=[workgroup] OS=[Windows Server 2012 R2 ] Server=[Windows Server 2012 R2]
	
		Sharename       Type      Comment
		---------       ----      -------
		ADMIN$          Disk      원격 관리
		C$              Disk      기본 공유
		D$              Disk      기본 공유
		E$              Disk      기본 공유
		IPC$            IPC       원격 IPC
		TEST            Disk      

3. 마운트 하기

		# mount -t cifs //192.168.0.1/test /cifsmnt -o user='samba',password='password',workgroup=workgroup


4. /etc/fstab에 등록

		# vi /etc/fstab
		//192.168.0.1/test /cifsmnt cifs defaults,user='samba',password='password',workgroup=workgroup,iocharset=utf8 0 0

5. 마운트 확인

		# df -h
		Filesystem            Size  Used Avail Use% Mounted on
		/dev/sda1             542G  230G  286G  45% /
		tmpfs                  24G     0   24G   0% /dev/shm
		//192.168.0.1/test    837G  116G  722G  14% /cifsmnt