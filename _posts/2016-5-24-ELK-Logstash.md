---
layout: post
title:  ELK Stack을 이용하여 로그시스템 구축하기 2편 - Logstash and Beats
description: ELK Stack - Logstash and Beats
tags: ELK Logstash beats logstash-forwarder

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

2. Logstash 설치 및 설정  
 Elastic 제품군은 잦은 업데이트로 인해 최신 릴리즈를 계속 확인하는게 좋다.[https://www.elastic.co/products](https://www.elastic.co/products "Elastic.co")  
 오늘날짜의 최신 Release Version은 2.3.2 버전이다.

		# wget https://download.elastic.co/logstash/logstash/packages/debian/logstash_2.3.2-1_all.deb
		# dpkg -i logstash_2.3.2-1_all.deb
		# vi /etc/logstash/conf.d/01-beats-input.conf
		input {
		  beats {
		    codec => plain {
		       charset => "UTF-8"
		    }
		    port => 5043
			# ssl을 이용하여 보안을 강화한 로그를 송수신할수 있으나 여기선 사용하지 않는것으로 테스트한다.
		    # ssl => true
			# ssl_certificate => "xx.crt"
			# ssl_certificate_authorities => ["xx.crt"]
			# ssl_key => "xx.key"
		}
		}

		# vi /etc/logstash/conf.d/30-elastic-output.conf
		output {
        	elasticsearch { hosts => ["192.168.10.10","192.168.10.11"] }
		}
		 

3. Filter 설정  
Filter를 설정하지 않으면 Client에서 보낸 Log를 그대로 Message라는 key를 가지고 ElasticSearch에 저장하게 된다.  
Logstash에서는 다양한 필터를 지원하고 있다.[https://www.elastic.co/guide/en/logstash/current/filter-plugins.html](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html "Logstash Filter")   
구조화 되지 않는 data를 Filtering하려면 정규식 또는 Grok 문법을 통해 파싱을 할 수 있다.[https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html "GROK")

4. Logstash 시작  
		
		# /etc/init.d/logstash start
		tcp6       0      0 192.168.10.10:5043    :::*                    LISTEN     
			
 	동일한 과정을 2번 노드에도 반복한다.

5. Client에서 Beat설치 및 설정


		# wget https://download.elastic.co/beats/filebeat/filebeat_1.2.3_amd64.deb
		# dpkg -i filebeat_1.2.3_amd64.deb
		# vi /etc/filebeat/filebeat.yml
		filebeat:
		  prospectors:
		    -
		      paths:
				# 전달할 Log File
		        - /log/test.log
		      input_type: log
			  # logstash filter시 사용할 수 있다.	
		      document_type: testlog
		      fields:
				# Log에 Custom Field를 추가 할수 있다.
		        name: abc
				# Custom Field는 Default로 fields.xxx 의 key로 저장이 되는데 아래 옵션을 true로 하면 xxx key로 저장한다.
		      fields_under_root: true
		
		  registry_file: /var/lib/filebeat/registry
		
		output:
		
		  logstash:
		    # The Logstash hosts
		    hosts: ["192.168.10.10:5043","192.168.10.115043"]
		    worker: 1
		    #compression_level: 3
		    # Optional load balance the events between the Logstash hosts
		    loadbalance: true
		
		
		logging:
		  files:
		    path: /var/log/mybeat
		    rotateeverybytes: 10485760 # = 10MB
		    keepfiles: 3
		  level: error


6. Client에서 Beat Test  
설정을 완료했으면 정상적으로 로그를 전달하는지 테스트가 필요하다.

		# filebeat -c /etc/filebeat/filebeat.yml -e -v
		2016/05/24 06:36:57.409461 geolite.go:24: INFO GeoIP disabled: No paths were set under output.geoip.paths
		2016/05/24 06:36:57.409650 logstash.go:106: INFO Max Retries set to: 3
		2016/05/24 06:36:57.409847 outputs.go:126: INFO Activated logstash as output plugin.
		2016/05/24 06:36:57.409917 publish.go:288: INFO Publisher name: NFS-MGC-GL1
		2016/05/24 06:36:57.410272 async.go:78: INFO Flush Interval set to: 1s
		2016/05/24 06:36:57.410284 async.go:84: INFO Max Bulk Size set to: 2048
		2016/05/24 06:36:57.410330 beat.go:147: INFO Init Beat: filebeat; Version: 1.2.3
		2016/05/24 06:36:57.410645 beat.go:173: INFO filebeat sucessfully setup. Start running.
		2016/05/24 06:36:57.410721 registrar.go:68: INFO Registry file set to: /var/lib/filebeat/registry
		2016/05/24 06:36:57.410755 prospector.go:133: INFO Set ignore_older duration to 0
		2016/05/24 06:36:57.410788 prospector.go:133: INFO Set close_older duration to 1h0m0s
		2016/05/24 06:36:57.410805 prospector.go:133: INFO Set scan_frequency duration to 10s
		2016/05/24 06:36:57.410817 prospector.go:93: INFO Input type set to: log
		2016/05/24 06:36:57.410828 prospector.go:133: INFO Set backoff duration to 1s
		2016/05/24 06:36:57.410839 prospector.go:133: INFO Set max_backoff duration to 10s
		2016/05/24 06:36:57.410850 prospector.go:113: INFO force_close_file is disabled
		2016/05/24 06:36:57.410874 prospector.go:143: INFO Starting prospector of type: log
		2016/05/24 06:36:57.410935 crawler.go:78: INFO All prospectors initialised with 1 states to persist
		2016/05/24 06:36:57.410954 registrar.go:87: INFO Starting Registrar
		2016/05/24 06:36:57.410977 publish.go:88: INFO Start sending events to output
		2016/05/24 06:36:57.410978 spooler.go:77: INFO Starting spooler: spool_size: 2048; idle_timeout: 5s
		2016/05/24 06:36:57.411060 log.go:113: INFO Harvester started for file: /log/test.log
		2016/05/24 06:37:32.304961 registrar.go:133: INFO Stopping Registrar
		2016/05/24 06:37:32.304984 registrar.go:97: INFO Ending Registrar
		2016/05/24 06:37:32.305157 registrar.go:162: INFO Registry file updated. 1 states written.
		...

		특별한 에러가 발생하지 않는다면 ctrl+c로 종료한다.

7. Beat Start
Test시 별다른 이상이 없으면 데몬을 시작한다.  
  
		# /etc/init.d/filebeat start