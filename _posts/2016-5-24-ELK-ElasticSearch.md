---
layout: post
title:  ELK Stack을 이용하여 로그시스템 구축하기 1편 - ElasticSearch
description: ELK Stack - ElasticSearch
tags: ELK ElasticSearch

---


가난한 자를 위한 Splunk. a.k.a "ELK(ElasticSearch + Logstash + Kibana)"를 이용하여 로그시스템을 구축해보자.  
테스트 환경은 아래와 같으며 Logstash은 LoadBalancing을 위해 두노드를 구성하고 Elastic은 2 Cluster로 구성해보자.  
Client에서는 Beats(기존 Logstash-Forwarder Project가 변경됨) - FileBeats를 사용하여 Logstash로 전달한다.

1. 테스트 환경

		Logstash01 : 192.168.10.10:5043  
		Logstash02 : 192.168.10.11:5043
		Elastic01  : 192.168.10.10:9200
		Elastic02  : 192.168.10.11:9200
		Log Client : client01~09    

2. ElasticSearch 설치 및 설정  
 Elastic 제품군은 잦은 업데이트로 인해 최신 릴리즈를 계속 확인하는게 좋다.[https://www.elastic.co/products](https://www.elastic.co/products "Elastic.co")  
 오늘날짜의 최신 Release Version은 2.3.3 버전이다.  

	- 1번 노드
	
			# wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/deb/elasticsearch/2.3.3/elasticsearch-2.3.3.deb
			# dpkg -i elasticsearch-2.3.3.deb
			# vi /etc/elasticsearch/elasticsearch.yml
			
			다양한 옵션이 있지만 기본적인 옵션만 알아본다.
			
			# Cluster의 이름
			cluster.name: log_cluster
			
			# Node의 이름 (지정하지 않으면 알아서 지정해준다.)
			node.name: log_node01
			
			# Swap사용을 최소화한다.
			bootstrap.mlockall: true
			
			# Node의 IP
			network.host: 192.168.10.10
			
			# Cluster를 구성시 node를 찾기 위한 Host List
			discovery.zen.ping.unicast.hosts: ["192.168.10.10","192.168.10.11"]
			
			# 노드당 ES Instance
			node.max_local_storage_nodes: 1
	
			# /etc/init.d/elasticsearch start
			# netstat -an
			# 정상으로 start가 되면 9200/9300 포트가 리슨한다.
			tcp6       0      0 192.168.10.10:9200    :::*                    LISTEN     
			tcp6       0      0 192.168.10.10:9300    :::*                    LISTEN   		

	- 2번노드는 1번노드와 동일한 과정을 거친다.


3. Cluster구성 확인  
정상적으로 1/2번 노드가 설치되었으면 Cluster구성 확인을 하자.
		
		# cat /var/log/elasticsearch/log_cluster.log 
		...
		[2016-05-24 20:14:50,849][INFO ][transport                ] [log_node01] publish_address {192.168.10.10:9300}, bound_addresses {192.168.220.232:9300}
		[2016-05-24 20:14:50,853][INFO ][discovery                ] [log_node01] log_cluster/ZZ09lZxCRIyZJgTdVvJpKQ
		# Master node
		[2016-05-24 20:14:53,875][INFO ][cluster.service          ] [log_node01] new_master {log_node01}{ZZ09lZxCRIyZJgTdVvJpKQ}{192.168.10.10}{192.168.10.10:9300}{max_local_storage_nodes=1}, reason: zen-disco-join(elected_as_master, [0] joins received)  
		[2016-05-24 20:14:53,926][INFO ][http                     ] [log_node01] publish_address {192.168.220.232:9200}, bound_addresses {192.168.220.232:9200}
		[2016-05-24 20:14:53,926][INFO ][node                     ] [log_node01] started
		[2016-05-24 20:14:53,979][INFO ][gateway                  ] [log_node01] recovered [0] indices into cluster_state
		# 2번 노드 Add
		[2016-05-24 20:15:00,965][INFO ][cluster.service          ] [log_node01] added {log_node02}{10P6VMDzTLWK113_1yNUaA}{192.168.10.11}{192.168.10.11:9300}{max_local_storage_nodes=1},}, reason: zen-disco-join(join from node[{log_node02}{10P6VMDzTLWK113_1yNUaA}{192.168.10.11}{192.168.10.11:9300}{max_local_storage_nodes=1}]) 

 