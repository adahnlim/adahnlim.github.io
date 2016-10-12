---	
layout: post
title:  Zabbix UserParameter
description: Zabbix UserParameter를 이용한 커스텀 매트릭 추가
tags: zabbix zabbix-UserParameter
---	

Custom Metric은 쉘명령어의 결과값을 이용하여 Zabbix에서 보여줄수 있다.

이 포스트에서는 RESTful을 이용하여 서버에서 값을 전달받아 Zabbix로 보내는 예제이다.

- 서버(node.js)

		// 0~2 사이의 랜덤 값을 리턴
		app.get('/rest/getResTimeTest', function(req, res){
	            res.send({'time':parseFloat((Math.random() * (2-0)+0).toFixed(2))})
		});

- Client(zabbix_agentd)

		# vi zabbix_agentd.conf
		UserParameter=web.restime,curl http:///rest/getResTimeTest -s | grep time| cut -d":" -f2 |awk '{print $1}'

		// 재시작
		# pkill zabbix_agentd
		# zabbix_agentd

		// 테스트
		# zabbix_agentd -t web.restime

		web.restime                                   [t|1.09]


- Zabbix WEB

	Configuration -> Template or Hosts -> Items -> Create Item

	![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/zabbix-1.png?raw=true)

	Item 특성에 맞게 그래프 또는 Event Trigger 추가

	![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/zabbix-2.png?raw=true)