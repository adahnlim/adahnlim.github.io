---	
layout: post
title:  Redis Master/Slave 구성하기
description: Redis 3.0.7 Master/Slave 구성하기 
tags: redis nosql In-MemoryDB 
---

캐시 및 DB용도로 널리 쓰이는 Redis를 Master/Slave로 구성해보자.

0. 테스트 환경
	
		Master : localhost:6000
		Slave  : localhost:6100  


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
	쉘 하나만 돌려주면 설치는 끝이다. Master / Slave를 한 노드에 설치할 예정이니 쉘을 두번돌려주면 되겠다.
  
		# ./install_server.sh 
		Welcome to the redis service installer
		This script will help you easily set up a running redis server
		
		Please select the redis port for this instance: [6379] 6000    
		Please select the redis config file name [/etc/redis/6000.conf] 
		Selected default - /etc/redis/6000.conf			
		Please select the redis log file name [/var/log/redis_6000.log]
		Selected default - /var/log/redis_6000.log
		Please select the data directory for this instance [/var/lib/redis/6000]
		Selected default - /var/lib/redis/6000
		Please select the redis executable path [/usr/local/bin/redis-server] 		
		Selected config:
		Port           : 6000
		Config file    : /etc/redis/6000.conf
		Log file       : /var/log/redis_6000.log
		Data dir       : /var/lib/redis/6000
		Executable     : /usr/local/bin/redis-server
		Cli Executable : /usr/local/bin/redis-cli
		Is this ok? Then press ENTER to go on or Ctrl-C to abort.
		Copied /tmp/6000.conf => /etc/init.d/redis_6000
		Installing service...
		 Adding system startup for /etc/init.d/redis_6000 ...
		   /etc/rc0.d/K20redis_6000 -> ../init.d/redis_6000
		   /etc/rc1.d/K20redis_6000 -> ../init.d/redis_6000
		   /etc/rc6.d/K20redis_6000 -> ../init.d/redis_6000
		   /etc/rc2.d/S20redis_6000 -> ../init.d/redis_6000
		   /etc/rc3.d/S20redis_6000 -> ../init.d/redis_6000
		   /etc/rc4.d/S20redis_6000 -> ../init.d/redis_6000
		   /etc/rc5.d/S20redis_6000 -> ../init.d/redis_6000
		Success!
		Starting Redis server...
		Installation successful!
		 
4. Slave 설정  
	설치하면 독립적으로 master mode로 동작하기때문에 master instance에서는 변경할 사항이 없고  
	slave instance 설정만 변경해주면 된다.
	설치 쉘을 돌리면 자동으로 instance를 올려주므로 slave instance를 내려준다.
		
		# /etc/init.d/redis_6100 stop
		# vi /etc/redis/6100.conf
		slaveof localhost 6000
		# /etc/init.d/redis_6100 start

5. Master/Slave 확인  
	이제 설정은 모두 끝났다. 정상적으로 구성이 되었나 확인하자.
	- Master  

			# redis-cli -p 6000
			127.0.0.1:6000> info replication
			# Replication
			role:master	# Master 
			connected_slaves:1	# 연결된 Slave 숫자
			slave0:ip=::1,port=6100,state=online,offset=141,lag=1	# 연결된 Slave 정보
			master_repl_offset:141
			repl_backlog_active:1
			repl_backlog_size:1048576
			repl_backlog_first_byte_offset:2
			repl_backlog_histlen:140

	- slave 

			# redis-cli -p 6100
			127.0.0.1:6100> info replication
			# Replication
			role:slave	# Slave
			master_host:localhost	# Master 정보
			master_port:6000
			master_link_status:up
			master_last_io_seconds_ago:1
			master_sync_in_progress:0
			slave_repl_offset:211
			slave_priority:100
			slave_read_only:1
			connected_slaves:0
			master_repl_offset:0
			repl_backlog_active:0
			repl_backlog_size:1048576
			repl_backlog_first_byte_offset:0
			repl_backlog_histlen:0


