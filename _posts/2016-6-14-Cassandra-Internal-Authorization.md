---	
layout: post
title:  Cassandra 3.x에서 Internal authorization
description: Cassandra 3.x의 내부 허가 체계 
tags: cassandra-authorization 
---

Cassandra에서 내부 허가(Internal Authorization)에 대해 알아보자.  

- Object 권한  
	카산드라는 RDB와 유사한 GRANT/REVOKE 를 사용하여 카산드라의 데이터의 접근제어를 수행한다.  
	superuser에게 초기권한을 부여하고 나중에 user들에게 GRANT/REVOKE 권한을 줄수있다. Object Permission관리는 Internal Authorization을 바탕으로 한다.  
	아래의 시스템 테이블은 암묵적으로 모든 User에게 읽기권한을 부여한다. 아래 시스템 테이블은 대부분 카산드라 툴에서 사용하기 때문이다.  
		- system_schema.keyspaces    
		- system_schema.columns  
		- system_schema.tables  
		- system.local  
		- system.peers  

- Internal Authorization 설정
	CassandraAuthorizer는 다양한 Internal Authorizer 중에 하나이다.  
	CassandraAuthorizer는는 system_auth.permission table에 퍼미션을 저장하고 허가와 관련된 CQL을 지원한다.   
	CassandraAuthorizer는를 활성화하려면 cassandar.yaml를 수정해야 한다.

	* 절차
		1. cassandra.yaml파일을 열어 수정한다.
					
				# authorizer: AllowAllAuthorizer 주석처리
				authorizer: CassandraAuthorizer

		2. system_auth keyspace를 위해 Replication Factor를 1보다 큰값으로 지정한다.
		3. 퍼미션 캐싱 주기를 알맞게 조정하자.(default : 2000 / 0이면 disable)
		
				permissions_validity_in_ms: 2000


	* 결과  
	CQL에서 GRANT / LIST PERMISSIONS / REVOKE를 사용할 수 있다.

- 방화벽 설정
	Cassandra Cluster node에서 방화벽이 수행중이면 반드시 아래 포트를 오픈해야한다.

	* Public Port  
	
| PortNumber | Description |
|:----------:|:-----------:|
| 22         | SSH Port    |

	* Cassandra Inter-node Ports

| PortNumber |           Description          |
|:----------:|:------------------------------:|
| 7000       | Cassandra 노드간 클러스터 통신 |
| 7001       | Cassandra SSL 클러스터 통신    |
| 7199       | Cassandra JMX 모니터링         |

	* Cassandra Client Ports

| PortNumber |           Description          |
|:----------:|:------------------------------:|
| 9042       | Cassandra Client Port |
| 9160       | Cassandra Client Port(Thrift)    |


- JMX 인증 Enable

	Cassandra의 기본설정은 JMX는 localhost에서만 접근가능하다. 원격 JMX Connection을 수행하려면 cassandra-env.sh에서 LCOAL_JMX 설정을 변경하고 인증과 SSL(선택) enable해야한다.  JMX 인증을 Enable하고 nodetool 등으로 확인한다.

	* 절차
		1. cassandra-env.sh 을 열어 수정한다.
 
				JVM_OPTS="$JVM_OPTS Dcom.sun.management.jmxremote.authenticate=true"
				JVM_OPTS="$JVM_OPTS -Dcom.sun.management.jmxremote.password.file=/etc/cassandra/jmxremote.password"

				LOCAL_JMX=no

		2. jmxremote.password.template을 복사한다.
		
				cp jdk_install_location/lib/management/jmxremote.password.template /etc/cassandra/jmxremote.password

		3. jmxremote.password 파일에 대한 퍼미션을 설정한다.

				chown cassandra:cassandra /etc/cassandra/jmxremote.password
				chmod 400 /etc/cassandra/jmxremote.password

		4. jmxremote.password 파일을 열어 JMX-Utilities를 위한 user와 password를 설정한다.
		
				monitorRole QED
				controlRole R&D
				cassandra cassandrapassword

		5. cassandra user한테 read permission을 준다.
		
				monitorRole readonly
				cassandra readwrite
				controlRole readwrite \
				create javax.management.monitor.,javax.management.timer. \
				unregister

		6. Cassdnra node 재시작
		7. nodetool로 확인
				
				nodetool status -u cassandra -pw cassandra

			만일 user와 password를 빼고 nodetool을 수행하면 아래와 같은 에러가 발생한다.

				Exception in thread "main" java.lang.SecurityException: Authentication failed!
				Credentials required
				at
				com.sun.jmx.remote.security.JMXPluggableAuthenticator.authenticationFailure(Unknown
				Source)
				at com.sun.jmx.remote.security.JMXPluggableAuthenticator.authenticate(Unknown
				Source)
				at sun.management.jmxremote.ConnectorBootstrap
				$AccessFileCheckerAuthenticator.authenticate(Unknown Source)
				at javax.management.remote.rmi.RMIServerImpl.doNewClient(Unknown Source)
				at javax.management.remote.rmi.RMIServerImpl.newClient(Unknown Source)
				at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
				at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
				at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
				at java.lang.reflect.Method.invoke(Unknown Source)
				at sun.rmi.server.UnicastServerRef.dispatch(Unknown Source)
				at sun.rmi.transport.Transport$1.run(Unknown Source)
				at sun.rmi.transport.Transport$1.run(Unknown Source)
				at java.security.AccessController.doPrivileged(Native Method)
				Configuration
				106
				at sun.rmi.transport.Transport.serviceCall(Unknown Source)
				at sun.rmi.transport.tcp.TCPTransport.handleMessages(Unknown Source)
				at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run0(Unknown Source)
				at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run(Unknown Source)
				at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
				at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
				at java.lang.Thread.run(Unknown Source)
				at sun.rmi.transport.StreamRemoteCall.exceptionReceivedFromServer(Unknown
				Source)
				at sun.rmi.transport.StreamRemoteCall.executeCall(Unknown Source)
				at sun.rmi.server.UnicastRef.invoke(Unknown Source)
				at javax.management.remote.rmi.RMIServerImpl_Stub.newClient(Unknown Source)
				at javax.management.remote.rmi.RMIConnector.getConnection(Unknown Source)
				at javax.management.remote.rmi.RMIConnector.connect(Unknown Source)
				at javax.management.remote.JMXConnectorFactory.connect(Unknown Source)
				at org.apache.cassandra.tools.NodeProbe.connect(NodeProbe.java:146)
				at org.apache.cassandra.tools.NodeProbe.<init>(NodeProbe.java:116)
				at org.apache.cassandra.tools.NodeCmd.main(NodeCmd.java:1099)



- Gossip 설정

	노드가 처음 시작될때 클러스터의 다른 노드들의 정보들을 주고받기 위해 설정된 클러스터 이름으로 자신이 속해있는 클러스터를 판별한다.
 
	* 절차
		1. cluster_name 설정
		2. listen_address 설정
		3. broadcast_address 설정
		4. seed_provider 설정
		5. storage_port 설정
		6. inital_token / num_tokens 설정
	
- heap dump directory 설정

	heap dump파일 분석은 메모리 문제에 대한 해결에 도움을 준다.  cassandra를 시작할때 JAVA Option에 -XX:+HeapDumpOnOutOfMemoryError.를 사용하면 OutOfMemory가 발생할때 heapdump를 수행한다.     
	기본적으로 Cassandra는 자신의 서브디렉토리에 파일을 넣기때문에 서비스를 root diretory에서 수행할때 root directory에 대한 write권한이 없으면 heapdump는 실패하게 되고 root directory에 여유용량이 없을때는 서버가 crash된다.

	heap dump를 성공하고 crash를 예방하기 위해서는 다음과 같은 요구사항이 필요하다.
  
		* Cassandra가 write할수 있는 퍼미션
		* heapdump를 위한 넉넉한 공간
	
	* 절차
		1. cassandra-env.sh 을 열어 수정한다.
		   
				set jvm HeapDumpPath with CASSANDRA_HEAPDUMP_DIR CASSANDRA_HEAPDUMP_DIR=<path>
	
		2. 노드를 재시작한다. 

