---	
layout: post
title:  Cassandra 3.x Configuration
description: Cassandra 3.x Configuration Properties 
tags: cassandra-properties
---


**Cassandra.yaml**


- Quick Start
  클러스터를 구서앟기 위한 가장 기본적인 항목이다.


| 항목           | 디폴트값     | 설명                                                                                                                                                                                                            |
|----------------|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| cluster_name   | Test Cluster | Cluster이름. 클러스터의 모든 노드들은 동일한 값을 가져야한다.                                                                                                                                                   |
| listen_address | localhost    | 노드들과 통신하기 위한 IP 또는 Hostname<br>- 값이 비어있으면 내부적으로 InetAddress.getLocalhost()를 호출한다.<br>- Single Node Cluster이면 Default(localhost)를 사용한다.<br>- 0.0.0.0은 절대로 사용하지 말라. |
|                |              |                                                                                                                                                                                                                 |
|                |              |                                                                                                                                                                                                                 |     |              |                                                               |