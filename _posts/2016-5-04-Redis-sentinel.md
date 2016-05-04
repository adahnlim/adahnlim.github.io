---	
layout: post
title:  Redis Sentinel 구성하기
description: Redis 3.0.7 Sentinel 구성하기 
tags: redis nosql In-MemoryDB sentinel
---

Redis Sentinel은 Master/Slave환경에서 Master가 다운되면 자동으로 감지하여 Slave를 Master로 올려준다.


0. 테스트 환경
	
		Master : localhost:6000
		Slave  : localhost:6100  
		Sentinel : localhost:7000


1. OS설정  
	Redis의 bgsave등 memory fork로 인해 메모리 할당 방식을 변경해야합니다.  
	※ 0 : default, malloc 요청 크기의 물리 메모리를 할당합니다.  
	※ 1 : malloc 요청 크기의 물리 메모리가 부족해도 할당 성공을 남깁니다.(단, Swap이 발생할 수 있어 성능 이슈가 생김)  
	※ 2 : malloc 요청 크기의 swap + (물리메모리 * vm.overcommitratio) 에 메모리 할당 합니다.  
				
		# vi /etc/sysctl.conf
		vm.overcommit_memory=1

2. 다운로드 및 압축해제  

		# wget http://download.redis.io/releases/redis-3.0.7.tar.gz
		# tar xvfz redis-3.0.7.tar.gz
		# cd redis-3.0.7/utils

3. 설치  
	쉘 하나만 돌려주면 설치는 끝이다.
  
		# ./install_server.sh 
		Welcome to the redis service installer
		This script will help you easily set up a running redis server
		
		Please select the redis port for this instance: [6379] 7000             
		Please select the redis config file name [/etc/redis/7000.conf]			
		Selected default - /etc/redis/7000.conf			
		Please select the redis log file name [/var/log/redis_7000.log] 		
		Selected default - /var/log/redis_7000.log
		Please select the data directory for this instance [/var/lib/redis/7000] 	
		Selected default - /var/lib/redis/7000
		Please select the redis executable path [/usr/local/bin/redis-server] /usr/local/bin/redis-sentinel
		Selected config:
		Port           : 7000
		Config file    : /etc/redis/7000.conf
		Log file       : /var/log/redis_7000.log
		Data dir       : /var/lib/redis/7000
		Executable     : /usr/loca/bin/redis-sentinel
		Cli Executable : /usr/local/bin/redis-cli
		Is this ok? Then press ENTER to go on or Ctrl-C to abort.
		Copied /tmp/7000.conf => /etc/init.d/redis_7000
		Installing service...
		 Adding system startup for /etc/init.d/redis_7000 ...
		   /etc/rc0.d/K20redis_7000 -> ../init.d/redis_7000
		   /etc/rc1.d/K20redis_7000 -> ../init.d/redis_7000
		   /etc/rc6.d/K20redis_7000 -> ../init.d/redis_7000
		   /etc/rc2.d/S20redis_7000 -> ../init.d/redis_7000
		   /etc/rc3.d/S20redis_7000 -> ../init.d/redis_7000
		   /etc/rc4.d/S20redis_7000 -> ../init.d/redis_7000
		   /etc/rc5.d/S20redis_7000 -> ../init.d/redis_7000
		Success!
		Starting Redis server...
		Installation successful!
		
4. Sentinel 설정  
	설치하면 Redis Server Config를 가지고 있으므로 Sentinel Config로 수정해준다.  
	- sentinel monitor mymaster  
		마지막 숫자(default 2)는 quorum(의사진행에 필요한 정족수)이라고 합니다. 예를 들어 센티널이 3대 일때 quorum이 2이면 2대 이상의 센티널에서 해당 레디스 마스터가 다운되었다고 인식하면 장애조치(failover) 작업을 진행합니다.     
		테스트 환경에서는 센티널을 1대로 했기 때문에 quorum을 1로 설정합니다.
	
	- sentinel down-after-milliseconds  
		마스터가 다운되었다고 인지하고 몇 초 후에 슬레이브를 마스터로 승격시키는 failover 작업을 시작 시점을 정합니다.  

			# /etc/init.d/redis_7000 stop
			# vi /etc/redis/7000.conf
			daemonize yes
			port 7000
			dir "/tmp"
			
	
			sentinel monitor mymaster 127.0.0.1 6300 1 
			sentinel down-after-milliseconds mymaster 3000
			sentinel failover-timeout mymaster 9000000
			sentinel config-epoch mymaster 0
			sentinel leader-epoch mymaster 0

 			# /etc/init.d/redis_7000 start


5. Sentinel 확인  
	이제 설정은 모두 끝났다. 정상적으로 구성이 되었나 확인하자.
	
		# redis-cli -p 7000 

		127.0.0.1:7000> sentinel get-master-addr-by-name mymaster
		1) "127.0.0.1"  	# 현재 Master 정보가 출력된다.
		2) "6300"
		127.0.0.1:7000>