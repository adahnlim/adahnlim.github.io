---
layout: post
title:  Cassandra 3.x에서 Cluster 구성시 고려해야할 사항
description: Cassandra 3.x에서 Cluster 구성(Enterprise/EC2)시 고려사항과 피해야할 사항 
tags: cassandra cluster architecutre 
---

카산드라 클러스터를 구축시 어플리케이션의 부하에 따라 초기볼륨을 어떻게 산정해야하는지에 대해 알아보자.

**Enterprise환경에서 H/W 계획**

Memory/CPU/Disk/Node의 갯수/네트워크 등을 밸런스에 맞게 선택하는게 중요하다.  
주의 : 부하테스트 및 운영환경을 개발환경에 맞추지 말아라! 오류가 발생할 수 있다.

- 메모리  
    노드에서 메모리가 많으면 Read Performance가 좋아진다.
	메모리양에 따라 최근 Write된 Data가 Memtable에 가지고 있는 데이터의 양이 달라진다.
	큰 Memorytable은 Disk Flushing 횟수를 줄이고 Read를 위해 적은 파일을 Scan할 수 있다.
	RAM의 양은 Hot data[^1]의 예상 사이즈에 따라 조절하면 좋다.
	Dedicate 환경과 Virtualization 환경에서 동일하다.   

		-  Production : 16GB~64GB (최소 8 GB)
		-  Development(TB 제외) : 4GB 미만

- CPU  
	다량 Insert는 Memory에서 처리하기전에 CPU에 부하가 생기는데 Cassandra에서는 Writing에 대해 CPU를 효율적으로 관리해준다.)

		- Production 
			* Dedicate : 8core CPU Processor
			* Virtual Env : 4~8 Core CPU Processor
		- Development(TB 제외)
			* Dedicate : 2core CPU Processor
			* Virtual Env : 2core CPU Processor


- Spinning Disk or SSD  
	Cassandra에서는 SSD를 추천한다. NAND Flash Chip기반의 SSD는 Random Read와 Sequential Write Performance에 Low-Latency Response time을 제공한다.    
	SSD를 구매시 부하에 따른 내구성으로 결정하지 말고 Fail 상태가 발생시 드라이브 교체가 어려운지에 초점을 맞추는게 좋다.


- Disk space  
	Disk Space는 사용량에 기초한다. Cassandra에서 Data를 Disk를 쓰는경우는 Commitlog 발생시와 memtables->SSTable로 Flushing되는 경우이다.
	SSTable은 주기적으로 Compacted된다.  
	Compaction은 Merging / Rewrite Data / Old data Remove 작업으로 성능을 향상시킨다.단, Compaction의 종류와 갯수에 따라 달라진다. 
	Compaction이 수행될때 Disk Util과 볼륨이 일시적으로 증가된다. 그러므로 노드에 충분한 Disk space가 있어야 한다. 큰 Compaction(WorstCase)일 경우 Compaction 종류에 따라 50% 이상의 충분한 Space가 있어야 한다.

		- Capacity per node 
			* 대부분의 경우 노드당 500GB~1TB가 Best!
			* Maximum Recommand : 노드당 3TB~5TB(Uncompressed data)
 
		- Capacity and I/O
			* Disk를 선택할때 Capacity와 I/O를 모두 고려해야한다. 어떠한 경우에는 싼 SATA Disk를 사용하여 node를 병렬로 늘리는것이 좋을 때도 있다.

		- Number of Disks - SATA
			* 최소 2장필요(Commit log / data)
		
		- Commit Log Disk - SATA
			* 용량이 클 필요는 없지만, 모든 Write에 대해 빠르게 처리가능해야한다.
		
		- Commit Log Disk - SSD
			* Commit Log와 SSTables를 같은 Mount Point에 위치하는게 좋다.

		- Data Disks
			* 한장 또는 여러장 디스크를 사용하고 큰 데이터볼륨과 캐싱되지 않은 데이터와 Compaction을 빠르게 처리할수 있어야 한다.
		
		- RAID on Data Disks
			* 다른 클러스터에 Replica가 있고, 1.2버전부터 JBOD feature가 포함되어 RAID10의 Overhead를 빼고 Disk Array를 구성할수 있기때문에 RAID는 권장되지 않는다.
			
		- RAID on the Commit log disk
			* Replica에서 Data loss를 방지하기때문에 필요하지 않지만, 더 강력한 Redundancy를 유지하고 싶으면 RAID 1를 사용해라.

		- Extended File Systems
			* XFS 또는 EXT4를 사용해라. Cassandra에서 SizeTieredCompactionStrategy를 사용하면 하나의 파일이 Disk Space의 반을 사용할 수 있기때문에 size limit이 큰 XFS또는 EXT4를 사용하는것이 좋다.


- Number of Nodes  
	3의 배수로 노드를 가져가면 좋다.
	[http://thelastpickle.com/blog/2011/06/13/Down-For-Me.html](http://thelastpickle.com/blog/2011/06/13/Down-For-Me.html "Node갯수와 Consitency Level에 따른 Fail Range")
	

- Network
	Cassandra는 분산 데이터 스토어로서, Request와 Replication을 네트워크를 통해 수행한다. 그러므로 Bottelneck이 발생하지 않도록 NIC를 분리하는게 좋다.

		- 1G 이상의 대역폭을 추천
		- rpc_address는 thrift/navtie protocol를 사용하는 것을 추천
		- listen_address는 내부 스토리지 프로토콜을 사용하는것을 추천

- Firewall
	Firewall을 사용하면 클러스터내의 각 노드들이 서로 통신이 가능해아한다.(세부 포트는 문서 104p 참조)  
	

**AWS EC2 Cluster에서의 계획**

- AMI는 신뢰된 Source에서 선택
	AMI는 신뢰된 Source에서 선택해라. 보안이슈 및 성능저하가 올수 있다.
  
		- Ubuntu Amazon EC2 AMI Locator  
		- Debian AmazonEC2Image  
		- CentOS-6 images on Amazon's EC2 Cloud

- Multi Region 및 Multi AZ로 클러스터 구성

- Production 환경에서 EC2 Configuration

		- Development and Light Production : m3.large
		- Moderate Production : m3.xlarge
		- SSD Production with light data : c3.2xlarge
		- Largest heavy production: m3.2xlarge (PV) or i2.2xlarge (HVM)
		- Micro, small, and medium types are not supported.

- EBS Volume
	- GP2(General Purpose Volumn)
	  
				- 대부분 환경에서 권장  
				- 10,000 IOPS 지원  
				- x millisecond latency
				- 99.0% 비율로 시간 내에 제공
	- PIOPS(Provisioned IOPS Volumn)
	
				- x millisecond latency
				- 99.0% 비율로 시간 내에 제공 

- EBS Magnetic volumn은 비추천  
	데이터에 자주 액세스하지 않는 Cold Data에 사용하는 볼륨이고, 100 IOPS정도밖에 지원하지 않는다.


**Partition 크기 측정**

효율적인 Operation을 위해 파티션은 약간의 제한을 걸어야 한다. 


- 파티션의 갯수
	실용적인 갯수는 20억개
- 파티션의 사이즈
	row의 갯수 및 컬럼의 갯수에 따라 달라지는데 보통의 경우 최대 100,000 row와 100MB 아래로 유지할것을 권고한다.
 

**Uable Disk 용량 측정**

1. raw_capacity = disk_size * number_of_data_disks
2. formatted_disk_space = (raw_capacity * 0.9)
3. usable_disk_space = formatted_disk_space * (0.5 to 0.8)

Cassandra의 보통 Operation과 Compaction와 Repair Operation을 수행할때 Disk 용량을 요구한다.  
그러므로 Cassandra에서는 Disk를 가득 채워서 사용하지 말고 50~80%정도로 운영하는것을 권장한다.


**Cassandra 구성시 피해야할 구성**

- Storage area network  
	SAN storage는 권장되지 않는다.  
		- ROI가 맞지 않는다.  
		- 분산 환경에서 SAN은 Bottleneck과 SPOF가 발생한다.(cassandra의 IO는 array controller의 한계치를 빈번하게 넘길수 있다)  
		- I/O 대기작업이 오래걸릴 수 있어 Heap에 영향을 줄수 있다.

- Network attached storage  
	SSTables를 NAS에 저장하는것을 권장하지 않는다.
		- NAS는 때때로 Network Bottleneck을 발생시켜 Read/Write 요청에 대기가 발생할 수 있다.

- Shared Network File System  
	NFS는 파일이 이동되거나 삭제될수 있어 일관성을 보장받지 못하기때문에 권장되지 않는다.

- Excessive heap space size  
	대부분의 경우 기본 heap사이즈를 권장한다. 이 사이즈가 넘어가면 GC(Garbage Collection)을 수행하는 능력을 손실할 수 있다.  
| Heap | Cpu Util | QPS | Latency |
|:---:|:---:|:---:|:---:|
| 40GB | 50% | 750 | 1 sec |  
| 8GB | 5% | 8500(not max) | 10ms |

 
- Cassandra Rack Feature
	Single-token Architere(Not Virtualnode)에 해당되는 내용입니다.  
	모든 클러스터를 하나의 Rack에 위치하는게 좋습니다.  
		- 많은 유저들은 Rack은 교차순서로 구성되어야 함을 무시하거나 잊어버리는 경향이 있다. 이 순서는 데이터가 안전하고 적절하게 분산하게 해준다.  
		- 많은 유저들은 Rack 정보를 효율적으로 관리하 지 않는다.  
	
	랙을 올바르게 사용하는 경우는 다음과 같다.   
		- 각 랙에 같은 노드의 수로 유지해라.  
		- 하나의 랙을 사용하여 교대 패턴으로 다른 랙에 노드를 배치합니다. 카산드라의 랙 기능은 빠르게 클러스터 확장을 할 수 있습니다.  

- SELECT .. IN or index lookups  
	특별한 경우를 제외하고는 SELECT ..IN과 Indexl Lookup(Secondary Index) 사용을 피해라.

- Using the Byte Ordered Partitioner
	Byte Ordered Partitioner의 사용은 권장되지 않는다.
	Virtual node를 사용할때나 사용하지 않을때에도 Murmur3Partioner(Default) 또는 RandomPartitioner를 사용하는게 좋다.

- Reading before writing  
	캐시되지 않는 읽기를 위해 일반적으로 여러개의 디스크를 일는데 시간이 걸리기 때문에 write전에 read를 수행하는게 좋다.

- Load balancers  
	cassandra는 LB를 필요로 하지 않는다. cassandra와 client사이에 LB를 놓으면 Performance/Cost/Debugging/Testing/Scaling에 어렵다.

- 부족한 테스트  
	운영환경 부하에 따라 Scale을 하면서 테스트를 진행해라. 이 방법이 노드당 처리량을 알 수 있는 최선이 방법이다.

- Too many keyspaces or tables  
	Cassandra Keyspace는 JVM Memory의 Overhead를 일으킨다. 각 테이블은 약 1MB를 사용한다. 예를 들어 3500개의 Table이 있으면 JVM Memory 3.5GB정도를 사용한다. 그래서 많은 Table 또는 keyspace가 있으면 메모리 요구사항이 높아진다.  
	최적은 500개 이하로 유지하고 최대 1000개를 넘지 않는것이 좋다.
	
- 친숙하지 않은 리눅스 환경
	Linux에는 좋은 툴들이 많다. 이것들과 친숙해져라.  
		- Parallel SSH and Cluster SSH
		- Passwordless SSH
		- dstat / top / iostat / mpstat / iftop / sar /lsof / netstat /vmstat 
		
 
	
[^1]: 데이터의 접근빈도가 많은 Data, <-> Cold data