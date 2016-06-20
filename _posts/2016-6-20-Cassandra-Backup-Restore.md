---	
layout: post
title:  Cassandra 3.x Backup & Restore data
description: Cassandra 3.x의 백업 및 복구
tags: cassandra-backup cassandra-restore
---

- Snapshot?

	Cassandra에서 snapshot을 수행하여 SSTable File을 백업한다. Snapshot은 하나 또는 모든 keyspace 그리고 온라인중에 single table을 백업 할 수 있다.  
	paralle ssh tool(pssh 등)을 이용하여 클러스터 내 모든 노드에 대해 Snapshot을 수행할 수 있다.   
	노드가 Snapshot 수행시 복제 노드의 일관성을 보장하지 않지만, 복원 된 스냅 샷은 카산드라의 내장된 일관성 메커니즘을 사용하여 일관성을 맞출 수 있다.  
	Snapshot이 수행된 이후에 incremental backup을 enable하여 마지막 snapshot의 변경분만 백업 받을수 있다. memtable이 flushing되거나 SSTable이 만들어질때 hardlink로 /backup 서브디렉토리로 복사된다.(JNA enable)
	
	Note : JNA[^1]가 Enable되어있으면 Snapshot은 hardlink로 수행되지만, disable되어있으면 file copy가 일어날때 I/O가 증가된다.
	

- Snapshot 수행  
 
	Snapshot은 노드에서 nodetool snapshot command로 수행한다. global snapshot을 수행하려면 parallel ssh utility(passh 등)을 이용한다.  
	Snapshot을 수행하면 메모리의 모든 내용을 disk로 flushing하고 SSTable file에 hard link를 만든다.
	Snapshot을 수행할 때 반드시 적정량의 여유공간이 필요하다. 하나의 snapshot은 용량이 크지않지만, snapshot은 disk 사용률의 증가속도를 빠르게한다. snapshot이 완료된 이후에는 backup file을 다른 곳으로 옮기는 것이 좋다.
	 
	Note : Cassandra는 table schema가 존재할때에는 data복구를 snapshot으로만 해야하기 때문에 schema backup을 권장한다.

	* 수행방법  
		- nodetool -h HOSTNAME -p JMX_PORT snapshot KEYSPACE

				nodetool -h localhost -p 7199 snapshot mykeyspace


	* 결과
		- data_directory_location/keyspace_name/table_name-UUID/snapshots/snapshot_name에 .db파일이 생성된다.


- Snapshot 삭제

	Snapshot은 자동적으로 지워지지가 않으므로 필요하지 않으면 삭제를 해주는것이 좋다.  
	nodetool clearsnapshot으로 모든 snapshot을 삭제할 수 있다.

	* 수행방법
		- nodetool -h HOSTNAME -p JMX_PORT clearsnapshot
		
- Incremental Backup 활성

	Increnemtal backup을 활성화하면 Cassandra는 각 memtable과 Flushing된 SSTable을 backup directory에 hard-link하기때문에 모든 snapshot을 copy를 하지않고 backup할수 있다.  
	Compacted된 SSTable은 backup 폴더에 hardlink를 만들지 않는다. 이유는 snapshot이 다른 compacted SSTable의 링크를 재구성해서 link를 만들기 때문이다.  
	Cassandrad는 자동적을 incremental backup file을 지우지 않기때문에 DataStax에서는 새로운 snapshot이 만들때 incremental backup hard link를 지우는 프로세스를 권장한다.

	* 수행방법
		- cassandra.yaml에 incremental_backups = true로 수정한다.

- Snapshot으로부터 복구

	snapshot으로 부터 keyspace를 복구하려면 모든 snapshot file이 피룡하고 만약 incremental backup을 사용하면 snapshot 이후 모든 incremental backup file이 필요하다.  
	snapshot 복원을 진행하기 전에 table을 truncate하는것을 권장한다.  
	Cassandra는 table schema가 존재할때에 snapshot으로부터만 복원이 가능하다. 만약 schema backup이 없다면 다음과 같이 해야한다.

		- Method 1
			1. Snapshot으로 복원한다.
			2. schema를 재생성한다.
		- Method 2
			1. schema를 재생성한다.
			2. Snapshot으로 복원한다.
			3. nodetool refresh를 수행한다.


	* 수행방법
			
			- Method 1
				sstableloader tool 사용
			- Method 2
				1. snaptshot SSTable directory(data/keyspace/table_name-UUID directory) copy
				2. JConsole에서 loadNewSSTable() 호출 또는 nodetool refresh 수행
			- Method 3
			  아래와 같이 node 재시작

- Node 재시작 방법

	만약 single node 복구라면 먼저 shutdown수행하고 모든 클러스터의 복구라면 모든 노드를 shutdown수행 후 복원하고 start를 진행한다.

	Note : snapshot과 incremental backup file을 이용하여 복원을 진행할 때 일시적으로 CPU와 I/O가 증가한다.


	* 절차
		1. 노드 shutdown
		2. data 유실방지확인을 위해 nodetool drain을 수행한다.
		3. commitlog directory의 모든 파일을 삭제한다.
		4. data_directory/keyspace_name/keyspace_name-keyspace_name/의 모든 *.db파일을 삭제한다. 그러나 snapshot과 backup subdirectory는 삭제하지 말라.
		5. 최신 snapshot을 data_directory/keyspace_name/table_name-UUID/snapshots/snapshot_name에 복사한다.
		6. 만약 backup파일이 있으면 data_directory/keyspace_name/table_name-UUID/backups에 복사한다.
		7. node를 start한다.
		8. nodetool repair를 수행한다.
			
- Snapshot으로부터 새로운 cluster에 복구

	만약 3node로 이루어진 클러스터(vnode + 256 tokens)로 부터 snapshot을 수행해서 새로운 3node cluster(vnode + 256 tokens)에 복구를 한다고 가정하자. 이때에는 token range가 old cluster와 new cluster가 완전히 같지는 않을것이기 때문에 old cluster에서 사용하던 token range를 new cluster에 적용할 필요가 있다.

	* 절차
		1. old cluster의 token list를 받아온다. 
			
				nodetool ring | grep ip_address_of_node | awk '{print $NF ","}' | xargs

		2. new cluster 각 노드의 cassandra.yaml 에 initial_token 항목에 1번의 결과값을 적는다.
		3. old cluster과 new cluster의 cassandra.yaml 기타 config를 맞춘다.
		4. new cluster의 system table data를 삭제한다.
				
				sudo rm -rf /var/lib/cassandra/data/system/*

		5. node를 시작한다.
		6. new cluster에 schema를 생성한다.
		7. node를 stop하고 nodetool refresh를 수행한다.
		8. new cluster에 동일한 directory에 snapshot을 복구한다.
		9. node를 재시작한다.


- JBOD에서 한개의 disk fail났을때 복구

	* 노드를 재시작할 수 있을 때 
		1. cassandra stop & node shutdown
		2. failed disk 교체
		3. start node & start cassandra
		4. nodetool repair 수행

	* 노드를 재시작할 수 없을 때(vnode사용)
		1. cassandra stop & node shutdown
		2. failed disk 교체
		3. 살아있는 node에서 명령어 수행
				
				nodetool ring | grep ip_address_of_node | awk ' {print $NF ","}' | xargs

		4. 위의 결과를 cassandra.yaml의 initial_token에 수정
		5. system directory 삭제
		
			-/mnt1/cassandra/data  (disk1)
			-/mnt2/cassandra/data  (disk2)
			-/mnt3/cassandra/data  (disk3)
		
			만약 disk1이 fail났으면

				rm -fr /mnt2/cassandra/data/system
				rm -fr /mnt3/cassandra/data/system

		6. node start & cassandra start
		7. nodetool repair 수행
		8. node가 정상적으로 올라오면 vnode 설정을 되돌린다.
		
				- num_tokens 복구
				- initial_token 주석처리

	* 노드를 재시작할 수 없을 때(single node)
		1. cassandra stop & node shutdown
		2. failed disk 교체
		3. system directory 삭제
		
			-/mnt1/cassandra/data  (disk1)
			-/mnt2/cassandra/data  (disk2)
			-/mnt3/cassandra/data  (disk3)
		
			만약 disk1이 fail났으면

				rm -fr /mnt2/cassandra/data/system
				rm -fr /mnt3/cassandra/data/system

		4. node start & cassandra start
		5. nodetool repair 수행
		


------------------------------------------


[^1]: JNA는 저작권 이슈로 카산드라 기본 패키지에는 들어가지 못하고 있지만, 메모리 관리 등의 성능 향상을 위해서는 이 라이브러리를 포함하여 cassandra를 구동시키는 것이 필요함.
특히 cassandra 백업을 위한 snapshot 기능을 이용하기 위해서는 반드시 포함 시키는 것이 좋다.